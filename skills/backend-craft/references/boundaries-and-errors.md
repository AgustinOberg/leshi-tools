# Boundaries and errors

## Schema-first

Every input that crosses a boundary is parsed with a schema: request body, query, params, headers, third-party responses, webhook payloads, queue messages, stored JSON values. Its type is derived from the schema (the DTO). No hand-written duplicate types for the same shape, no `any`, no unchecked casts, no suppressed type or lint errors.

## The error shape

Every error response has the same shape. Clients map `code` to localized messages.

```json
{ "error": "Request failed", "code": "ORDER_ALREADY_OPEN" }
```

Validation failures add the failing fields (dotted path → schema issue code):

```json
{ "error": "Invalid request", "code": "VALIDATION_FAILED", "fields": { "items.0.quantity": "too_small" } }
```

```ts
// shared/validate.ts — the only way controllers validate input
export const validate = <T extends z.ZodType, Target extends keyof ValidationTargets>(target: Target, schema: T) =>
  zValidator(target, schema, (result, c) => {
    if (!result.success) {
      const fields = Object.fromEntries(result.error.issues.map((i) => [i.path.map(String).join('.'), i.code]));
      return c.json({ error: 'Invalid request', code: 'VALIDATION_FAILED', fields }, 400);
    }
  });
```

## Errors with stable codes

```ts
// shared/errors.ts
export class DomainError extends Error {
  constructor(readonly code: string) {
    super(code);
  }
}
```

Domain errors are typed classes in their module (`OrderAlreadyOpen`, `OrderNotFound`). Only the edge knows about HTTP:

```ts
// shared/error-handler.ts — the edge maps errors to transport
const STATUS_BY_CODE: Record<string, ContentfulStatusCode> = {
  UNAUTHENTICATED: 401,
  FORBIDDEN: 403,
  ORDER_NOT_FOUND: 404,
  ORDER_ALREADY_OPEN: 409,
  RATE_LIMITED: 429,
};
const CODE_BY_STATUS: Record<number, string> = { 400: 'BAD_REQUEST', 401: 'UNAUTHENTICATED', 413: 'PAYLOAD_TOO_LARGE' };

export const handleError: ErrorHandler<AppEnv> = (err, c) => {
  if (err instanceof DomainError) {
    return c.json({ error: 'Request failed', code: err.code }, STATUS_BY_CODE[err.code] ?? 400);
  }
  if (err instanceof HTTPException) {
    // Thrown by the framework and its middleware: malformed JSON, body limit, auth.
    return c.json({ error: 'Request failed', code: CODE_BY_STATUS[err.status] ?? 'BAD_REQUEST' }, err.status);
  }
  errorTracking.capture(err, { requestId: c.get('requestId') }); // role module; never request bodies
  return c.json({ error: 'Internal error', code: 'INTERNAL' }, 500);
};

export const handleNotFound: NotFoundHandler<AppEnv> = (c) =>
  c.json({ error: 'Not found', code: 'ROUTE_NOT_FOUND' }, 404);
```

- Anything unknown becomes a generic internal error. Never leak stack traces, SQL, vendor messages or internal ids.
- Return the same "not found" for a resource that does not exist and for one that belongs to another user, so ownership is not revealed.

## External calls

- Every external call has a timeout. Pass `AbortSignal.timeout(MS)` to `fetch` or to the SDK when it accepts a signal (the request is really cancelled); use a promise race only when it does not.
- Retry only what is idempotent; use idempotency keys for payments and other side effects that must not happen twice.
- Degrade instead of failing when a fallback exists: a secondary provider, a cached value, "no suggestion".

## Vendors behind role modules

```ts
// integrations/email/index.ts — the only file that imports the vendor SDK
export interface Email {
  send(message: EmailMessage): Promise<void>;
}

export class ResendEmail implements Email {
  constructor(private readonly client = new Resend(config.email.apiKey)) {}

  async send(message: EmailMessage) {
    try {
      // The SDK takes no AbortSignal, so the call is raced against a timeout.
      const { error } = await withTimeout(this.client.emails.send(toResendPayload(message)), EMAIL_TIMEOUT_MS);
      if (error) throw error;
    } catch {
      throw new EmailUnavailable(); // vendor, network and timeout failures become the app's error
    }
  }
}

let instance: Email | undefined;
export const defaultEmail = (): Email => (instance ??= new ResendEmail()); // once per runtime instance
```

- Named by role (`email`, `payments`, `ai`, `storage`, `search`), never by vendor.
- Translates every vendor, network and timeout failure into the app's own small vocabulary, and parses vendor payloads.
- Swapping the vendor touches one place.

## Data integrity

- Invariants live in the database: unique (and partial unique) indexes, foreign keys, NOT NULL, checks. A read-then-write check is only a fast path for a clear error; the constraint is the guarantee. The repository translates the violation into the domain error.
- Writes that must succeed together run in one transaction (on D1, which has no interactive transactions, use `db.batch([...])`).
- Email, push, payments and other side effects run after the commit, never inside the transaction.

## Pagination

Every list endpoint is paginated from day one; adding it later breaks clients.

- A cursor over a stable order (`createdAt, id`), never an offset over data that changes.
- A default `limit`, capped by `MAX_PAGE_SIZE`.
- The response is `{ items, nextCursor }`, with `nextCursor: null` on the last page.

## Inbound webhooks

- Verify the signature or shared secret on the raw body before parsing it.
- Store the event id and skip events already processed: providers deliver at least once and out of order.
- Acknowledge fast with a 2xx and move slow work to a queue.

## Config and secrets

- Declared in one typed place, parsed at startup, referenced by name.
- A missing optional secret turns its feature off; a missing required one fails loudly at startup.
- No secrets in code, config files, tests, fixtures, logs or responses. Use environment references.

## Authorization and abuse

- Authentication is checked at the edge; authorization is decided where data is accessed (a service rule or a repository query scoped by owner).
- Every query that reads or changes user data is scoped to the user or role allowed to see it.
- Sign-in, one-time codes and expensive endpoints (AI, email, uploads) are rate limited per user or IP and return 429 `RATE_LIMITED`.

## Evolving the API and the data

- **Additive API:** add fields and endpoints; do not change or remove them unless the project decided to break the contract.
- **Migrations add:** new tables, columns with defaults, new indexes. Destructive changes happen in separate, deliberate steps.
- **Never edit a migration that already ran** outside your machine; add a new one.
- **Data migrations are idempotent:** running them twice is safe. They ship through the normal deploy; development work never reads or writes production data.

## Logs, privacy and naming

- Every request carries a request id (for example Hono's `requestId` middleware). Logs are structured: request id, route, status, duration and error code.
- Logs, analytics and error tracking carry no user content, personal data, request bodies or secrets.
- Constants carry their unit in the name (`TIMEOUT_MS`, `MAX_UPLOAD_BYTES`); limits and quotas live in one place per module; guessed values are labelled as such.
