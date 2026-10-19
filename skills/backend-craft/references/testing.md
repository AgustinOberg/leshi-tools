# Backend testing

Every backend change ships with its tests, whatever else the project's test policy says. The project only chooses the tools.

## The setup

```
test/
  setup.ts                 # applies migrations to the test database
  helpers/
    api.ts                 # calls the app through its entry point, with or without a session
    expect-error.ts        # asserts the shared error shape and codes
  factories/
    user.factory.ts        # one per entity
    order.factory.ts
  fakes/
    payments.fake.ts       # one per external service: success + failure variants + reset
    email.fake.ts
  fixtures/
    avatar.png
  modules/
    orders/
      create-order.test.ts # one file per endpoint or use case, mirroring src/
      cancel-order.test.ts
```

### A local database only for tests

- Its own config (a test env file, an in-memory database, or the runtime's local emulator, such as a local D1 in the Workers Vitest pool).
- Created from the **same migrations** before the run.
- Never the development, staging or production database. If the configuration could point elsewhere, the setup refuses to run.
- **Isolation:** test files run in parallel, so each worker gets its own database (or the runtime's per-test isolated storage), or files run serially. Never truncate tables another worker is using. Each test creates its own rows and asserts only on them.

### Through the real entry point

Tests call the app the way a client does, so validation, auth, error mapping and data access are all exercised. Never call the service or repository directly in an endpoint test.

```ts
// test/helpers/api.ts
export function createApi(overrides: Partial<Deps> = {}) {
  const app = createApp(overrides); // the same factory production uses
  const call = async (method: string, path: string, { as, body }: CallOptions = {}) =>
    app.request(path, { method, headers: await headersFor(as), body: body && JSON.stringify(body) }, env);
  return { get: (path: string, o?: CallOptions) => call('GET', path, o), post: (path: string, o?: CallOptions) => call('POST', path, o) };
}
```

- `app.request(path, init, env)` runs the real app in process (in the Workers pool, `env` comes from `cloudflare:test`). `SELF.fetch` is fine when no fake is needed.
- `as: user` creates a real session through the session factory and sends its cookie or token. Auth is never stubbed or bypassed by an env flag.

### Data from factories

One factory per entity in the shared test folder. It builds valid data from the model's own constraints (lengths, enums, required fields); a generator persists it.

```ts
// test/factories/user.factory.ts
export function buildUser(overrides: Partial<NewUser> = {}): NewUser {
  return {
    id: crypto.randomUUID(),
    email: `${randomString(10)}@example.test`,
    name: randomString(randomInt(1, USER_NAME_MAX_LENGTH)),
    role: 'customer', // fields that change behavior get a fixed default; tests override them
    ...overrides,
  };
}

export async function createUser(overrides: Partial<NewUser> = {}) {
  const [user] = await testDb.insert(users).values(buildUser(overrides)).returning();
  return user;
}
```

- Randomize only fields no rule depends on (names, emails, ids). Fields that drive authorization or state (role, status, plan) get a fixed default and are set by the test that cares.
- No hand-written rows copied between tests.

### Shared fakes per external service

Each external service gets one fake in the shared folder with success and failure variants and a reset. Tests never touch the network.

```ts
// test/fakes/payments.fake.ts
export function createPaymentsFake() {
  let mode: 'success' | 'declined' | 'unavailable' = 'success';
  const charges: ChargeInput[] = [];

  const payments: Payments = {
    async charge(input) {
      charges.push(input);
      if (mode === 'declined') throw new PaymentDeclined();
      if (mode === 'unavailable') throw new PaymentsUnavailable();
      return { id: `ch_${charges.length}` };
    },
  };

  return {
    payments,
    charges,
    fail(next: 'declined' | 'unavailable') {
      mode = next;
    },
    reset() {
      mode = 'success';
      charges.length = 0;
    },
  };
}
```

Fakes enter through the app factory (`createApi({ payments: fake.payments })`), never by mocking modules (`vi.mock`, `jest.mock`).

### Shared assertion helpers

Helpers for the error shape (`expectError(res, 409, 'ORDER_ALREADY_OPEN')`, `expectFieldError(res, 'items')`) and auth headers; file fixtures in the fixtures folder.

## What to cover per endpoint

| Case | Example test name |
|---|---|
| Happy path | `creates an order for the current user` |
| Validation, per field | `returns 400 with a field error when items is empty` |
| Authentication | `returns 401 without a session` |
| Roles | `returns 403 when a customer calls the admin endpoint` |
| Ownership | `does not return or change another user's order` |
| Not found | `returns 404 for an unknown order` |
| Conflicts | `returns 409 when the user already has an open order` |
| Limits | `returns 400 when the order has more than 50 items` |
| Pagination (list endpoints) | `returns the next page from nextCursor without duplicates` |
| External failures (when relevant) | `keeps the order pending when the payment provider is unavailable` |

```ts
// test/modules/orders/create-order.test.ts
describe('POST /orders', () => {
  const api = createApi();

  it('creates an order for the current user', async () => {
    const user = await createUser();
    const res = await api.post('/orders', { as: user, body: buildCreateOrderBody() });
    expect(res.status).toBe(201);
  });

  it('returns 409 when the user already has an open order', async () => {
    const user = await createUser();
    await createOrder({ userId: user.id, status: 'pending' });
    const res = await api.post('/orders', { as: user, body: buildCreateOrderBody() });
    await expectError(res, 409, 'ORDER_ALREADY_OPEN');
  });

  it('returns 401 without a session', async () => {
    const res = await api.post('/orders', { body: buildCreateOrderBody() });
    await expectError(res, 401, 'UNAUTHENTICATED');
  });
});
```

## General rules

- Test behavior, named as a sentence.
- Pure logic (rules, calculations, mappings) also gets unit tests.
- Fake only at the edges of the system (third-party APIs, AI, push, payments, email, storage providers). Never mock your own modules.
- Tests that cost money or hit real providers are opt-in scripts with a dry run by default, outside CI.
- Every bug fix comes with a test that fails before the fix.
- Never remove, skip or weaken a test or a check to make it pass.

## Building the setup from scratch

When the project has no backend test setup, build it as the first task:

1. Ask only which tools to use if the project has not decided (offer the stack profile when it fits).
2. Add the test database config, the setup that applies migrations, and per-worker isolation.
3. Make sure the app is built by a factory that accepts dependency overrides (`createApp`).
4. Add the `api` and `expectError` helpers.
5. Add factories for the entities the first tests need, and fakes for the external services they touch.
6. Write the first endpoint test and make the test command part of the project's checks.
