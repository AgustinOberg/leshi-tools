# Architecture

## Slice by feature, then by role

Each feature owns its folder. Inside it, files split by role. A central entry (the app router or root layout) only mounts features.

```
src/
  app/                      # routes/pages: composition only
    orders/
      index.tsx             # renders <OrdersScreen />
  features/
    orders/
      components/           # feature parts: render only
        orders-screen.tsx   # calls useOrdersScreen, renders the four states
        order-list.tsx
        order-row.tsx
        orders-skeleton.tsx
      hooks/                # state, effects, handlers
        use-orders-screen.ts
        use-get-orders.ts
        use-cancel-order.ts
      api/
        orders-service.ts   # calls the HTTP client, parses with schemas
        orders-service.mock.ts
        order-keys.ts       # query keys for this area
      schemas/
        order.schema.ts     # schemas + derived types
      logic/
        order-status.ts     # pure rules and mappings
      index.ts              # the feature's public surface
  ui/                       # the layered kit
    primitives/
    kit/
    shared/
  lib/
    http/                   # the one HTTP client
    query/                  # the one query client
    storage/                # the one storage module + key registry
    i18n/
  integrations/             # role modules: analytics, payments, push...
  config/                   # typed config, read in one place
```

Adapt the names to the framework (Expo Router's `app/`, Next.js `app/`, a Vite client router), but keep the split.

## Rules

- **One concern per file, small files.** When a file does two things or keeps growing, split it.
- **Logic in pure functions.** Rules, state machines, calculations and mappings are pure and unit-testable. Components, routes and hooks only connect.
- **Dependencies point one way:**

  ```
  routes → components → hooks → pure logic
                              → query hooks → services → HTTP client / role modules
  ```

  Lower layers never import upper ones. Pure logic imports nothing from React, services or the HTTP client.
- **A small public surface.** Other features import from `features/orders` (its `index.ts`), never from `features/orders/hooks/...`.
- **Read what is there first.** Before writing, read the similar feature that already exists and copy its shape.

## Developer experience first; abstract only what can change

- Use the core framework directly and idiomatically: router, styling engine, query library, test runner. Wrapping the core hurts DX and buys nothing.
- Everything that is not core and might be replaced lives behind one app-shaped module named by its **role**, not its vendor:

```ts
// integrations/analytics/index.ts — the only file that imports the vendor SDK
import posthog from 'posthog-js';

type AnalyticsEvent =
  | { name: 'order_created'; props: { itemCount: number } }
  | { name: 'order_cancelled'; props: { reason: CancelReason } };

export const analytics = {
  track(event: AnalyticsEvent) {
    posthog.capture(event.name, event.props);
  },
};
```

  Swapping the vendor touches one place. Translate vendor errors and payloads into the app's own small vocabulary inside that module.
- A cross-cutting library used everywhere (i18n, theme) gets a thin hook or helper only when it makes call sites simpler.
- Add an interface only where a change is expected or a test needs a seam. No speculative layers.

## Types and boundaries

- **Schema-first.** Every input that crosses a boundary is parsed with a schema: API responses, route and query params, stored values, deep links, third-party payloads. Its type is derived from the schema; no hand-written duplicate types for the same shape.
- Naming: a schema is `XSchema` and its derived type is `X`, the same convention the backend uses.

```ts
// features/orders/schemas/order.schema.ts
import { z } from 'zod';

export const OrderSchema = z.object({
  id: z.string(),
  status: z.enum(['pending', 'paid', 'cancelled']),
  totalCents: z.number().int(),
  createdAt: z.iso.datetime().transform((value) => new Date(value)), // never z.coerce at a trust boundary
});
export const OrderPageSchema = z.object({ items: z.array(OrderSchema), nextCursor: z.string().nullable() });

export type Order = z.infer<typeof OrderSchema>;
```

- No `any`, no unchecked casts, no suppressed type or lint errors.
- Model states as discriminated unions (`loading | error | ready`) with one discriminant name per project (for example always `status`).
- Constants carry their unit in the name (`DEBOUNCE_MS`, `MAX_UPLOAD_BYTES`). Limits live in one place per feature. Values that are guesses are labelled as such.

## Errors

- The backend sends stable machine codes (`{ error, code }`, plus `fields` for validation failures). The client maps codes to typed i18n keys in one place:

```ts
// lib/http/error-messages.ts
const MESSAGE_KEY_BY_CODE = new Map<string, TranslationKey>([
  ['ORDER_ALREADY_OPEN', 'errors.orderAlreadyOpen'],
  ['NETWORK_UNAVAILABLE', 'errors.offline'],
]);

export const messageKeyFor = (code: string): TranslationKey => MESSAGE_KEY_BY_CODE.get(code) ?? 'errors.generic';
```

- Anything unknown becomes a generic message. Never render raw server or vendor messages.
- Degrade instead of failing when a fallback exists (cached value, "no suggestion").
- Every network call has a timeout (`AbortSignal.timeout(MS)` combined with the query's cancellation signal); retry only what is idempotent.

## Config, secrets, security and privacy

- Config is declared in one typed place, parsed with a schema and read by name. A missing optional value turns its feature off; a missing required one fails loudly at startup.
- Public build-time variables are inlined only when referenced literally (`process.env.EXPO_PUBLIC_API_URL`, `import.meta.env.VITE_API_URL`). The config module names each one literally, never `process.env[name]`.
- No secrets in client code: anything shipped to the browser or the app bundle is public.
- Auth tokens live in secure storage (Keychain/Keystore, for example through `expo-secure-store`; an httpOnly cookie on web), never in MMKV, AsyncStorage or `localStorage`.
- Never render untrusted HTML (`dangerouslySetInnerHTML`, a WebView's `html`) without a sanitizer. Open URLs that come from content or deep links only after checking the scheme and host against an allow-list.
- Logs and analytics carry no user content, personal data or secrets.
