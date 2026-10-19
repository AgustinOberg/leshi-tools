# State and data

## Place state by scope

| Kind of state | Where it lives |
|---|---|
| Server data | The server-state cache (for example TanStack Query), with centralized query keys |
| Local UI state (open, selected, input text) | The screen's or component's hook |
| Multi-step flow state (wizard, checkout) | A reducer owned by the flow |
| Persisted preferences | One storage module with a registry of keys |
| Truly global app state | Only when the above do not fit, and only after asking |

No global store by default. Do not copy server data into local state.

## Data in three tiers

```
HTTP client  →  service per backend area  →  query / mutation hooks  →  screens
```

Screens never call the HTTP client directly.

**1. One HTTP client.** Knows the base URL, attaches auth and language, refreshes tokens, applies a timeout, honors the caller's cancellation signal and turns failures into coded errors.

```ts
// lib/http/client.ts
export class ApiError extends Error {
  constructor(readonly code: string, readonly status: number, readonly fields?: Record<string, string>) {
    super(code);
  }
}
```

**2. One service per backend area.** Calls the client and parses the response with a schema. It returns typed, trusted data and accepts a cancellation signal.

```ts
// features/orders/api/orders-service.ts
export const ordersService = {
  async list({ cursor, signal }: { cursor: string | null; signal?: AbortSignal }) {
    return OrderPageSchema.parse(await http.get('/orders', { query: { cursor }, signal }));
  },
  async cancel(id: string) {
    return OrderSchema.parse(await http.post(`/orders/${id}/cancel`));
  },
};
```

**3. Query and mutation hooks.** Keys come from one place per area.

```ts
// features/orders/api/order-keys.ts
import { createQueryKeys } from '@lukemorales/query-key-factory';

export const orderKeys = createQueryKeys('orders', {
  list: null,
  detail: (id: string) => [id],
});
```

```ts
// features/orders/hooks/use-get-orders.ts
export function useGetOrders() {
  return useInfiniteQuery({
    ...orderKeys.list,
    queryFn: ({ pageParam, signal }) => ordersService.list({ cursor: pageParam, signal }),
    initialPageParam: null as string | null, // widens the cursor type; not a cast of untrusted data
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  });
}

// features/orders/hooks/use-cancel-order.ts
export function useCancelOrder() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ordersService.cancel,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: orderKeys._def }),
  });
}
```

- Invalidate after mutations instead of editing the cache by hand.
- Optimistic updates only on purpose (the interaction needs instant feedback), with rollback on error.
- Passing the query's `signal` down means leaving a screen cancels its requests.

## The query client

Created once in `lib/query/client.ts`:

- Do not retry client errors: `retry: (count, error) => !(error instanceof ApiError && error.status >= 400 && error.status < 500) && count < 2`.
- React Native: wire `onlineManager` to NetInfo and `focusManager` to `AppState` once, or offline pausing and refetch-on-focus never happen.
- In tests, a fresh client per test with `retry: false` (see `testing.md`).

## Frameworks that fetch on the server

In Next.js server components or loader-based routers (React Router, Expo Router loaders), the loader replaces the query hook for the initial read. It still calls the service and parses with the schema. Interactive client data stays in the query cache.

## Lists, waterfalls and prefetching

- Server lists are paginated (`useInfiniteQuery` with a cursor) and rendered in a virtualized list (FlatList or LegendList on native, a virtualizer on web) with stable keys. Never `.map` an unbounded list into a ScrollView.
- A screen's independent queries start together in its screen hook. Children that only start fetching after the parent's data arrives create a waterfall.
- Dependent queries use `enabled`. Predictable next screens are prefetched on press with `queryClient.prefetchQuery`.

## When the backend is not ready

Write a mock that implements the **same service signature**:

- realistic latency (a named constant such as `MOCK_LATENCY_MS`);
- the same coded errors the real backend returns;
- switchable scenarios: ready, empty, error, slow.

Choose the real or mock implementation in one place (config), so switching never touches screens or hooks.

## The contract with the backend

- Parse every response; never trust a shape.
- The contract is agreed with the backend side. If it is insufficient or wrong, stop and say so. Never invent endpoints or change backend code from a frontend task.
- How types are kept in sync (duplicated per app with contract tests, or a shared package) is a project decision. Ask when it is not recorded.

## The four states of every data screen

| State | What to show |
|---|---|
| Loading | Skeletons shaped like the content (not a lone spinner on a blank screen) |
| Error | A message from the error code, and a retry action; keep cached data on screen when a background refetch fails |
| Empty | Why it is empty and the next action, when there is one |
| Ready | The content |

Also consider the edge cases the feature implies: offline, slow network, permission denied, limits reached, very long text, the second language, large text sizes.

## Forms

- One hook per form (`useCheckoutForm`) owns values, validation and submit.
- Validate with the same schema family used at the boundary; show field errors next to the field.
- Map the server's `fields` and error codes to field or form messages; never show raw server text.
- Disable submit only while submitting; keep the user's input after an error, and re-enable submit when the request fails.

## Persisted preferences

```ts
// lib/storage/keys.ts — the registry: every persisted key lives here
export const STORAGE_KEYS = {
  theme: 'prefs.theme',
  onboardingDone: 'prefs.onboardingDone',
} as const;
```

Read and write through the storage module only, parsing stored values with a schema (stored data may come from an older version of the app). Secrets and tokens never go here; they go to secure storage.
