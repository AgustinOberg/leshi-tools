# Components and hooks

## Components render; hooks think

The component reads its one purpose hook (plus presentational hooks such as translation or theme) and returns markup. Everything else — state, effects, data, handlers, navigation — lives in custom hooks named by purpose, one hook per file.

```tsx
// features/orders/components/orders-screen.tsx
export function OrdersScreen() {
  const { orders, isPending, error, retry, loadMore, openOrder } = useOrdersScreen();
  const { t } = useTranslation();

  if (isPending) return <OrdersSkeleton />;
  if (error) return <ErrorState error={error} onRetry={retry} />;
  if (orders.length === 0) return <EmptyState title={t('orders.list.emptyTitle')} />;
  return <OrderList orders={orders} onEndReached={loadMore} onOpen={openOrder} />;
}
```

```ts
// features/orders/hooks/use-orders-screen.ts
export function useOrdersScreen() {
  const router = useRouter();
  const { data, isPending, error, refetch, hasNextPage, isFetchingNextPage, fetchNextPage } = useGetOrders();

  const openOrder = (id: string) => router.push(`/orders/${id}`);
  const loadMore = () => {
    if (hasNextPage && !isFetchingNextPage) void fetchNextPage();
  };

  return {
    orders: data?.pages.flatMap((page) => page.items) ?? [],
    isPending,
    error: data ? null : error, // a failed background refetch keeps the cached list on screen
    retry: () => void refetch(),
    loadMore,
    openOrder,
  };
}
```

Naming: `useXScreen` for a screen's orchestration, `useXForm` for a form, `useGetX` / `useCreateX` for queries and mutations.

## Pages and routes only compose

A route file mounts providers and the feature's screen. No logic, no data fetching, no styling decisions.

```tsx
// app/orders/index.tsx
export default function OrdersRoute() {
  return <OrdersScreen />;
}
```

## Avoid effects

| You want to… | Do this instead |
|---|---|
| Compute a value from props or state | Compute it during render |
| React to a click, submit or gesture | Put it in the event handler |
| Reset state when an id changes | Pass `key={id}` or use a reducer |
| Fetch data | A query hook backed by the server-state cache |
| Notify a parent of a change | Call the parent's callback in the same handler |
| Subscribe to an external system (device API, socket, imperative widget, browser API) | An effect, with a one-line comment saying why |

```tsx
// ❌ derived state synced by an effect
const [total, setTotal] = useState(0);
useEffect(() => setTotal(calculateTotal(items)), [items]);

// ✅ derived during render
const total = calculateTotal(items);
```

```ts
// ✅ a legitimate effect: synchronizing with an external system
useEffect(() => {
  // Keeps the session alive while the app is in the foreground (AppState is external).
  const subscription = AppState.addEventListener('change', onAppStateChange);
  return () => subscription.remove();
}, [onAppStateChange]);
```

When the effect calls a handler that must see fresh props or state without re-subscribing, wrap the handler in `useEffectEvent` (React 19.2+) and leave it out of the dependencies.

## Small components, composed

Split a component as soon as it has two responsibilities. Prefer composition over boolean props that switch behavior.

```tsx
// ❌ behavior switched by booleans
<OrderCard compact showActions isSelectable />

// ✅ named parts; variants are separate parts
<OrderCard>
  <OrderCard.Header order={order} />
  <OrderCard.Summary order={order} />
  <OrderCard.Actions>
    <CancelOrderButton orderId={order.id} />
  </OrderCard.Actions>
</OrderCard>
```

When parts need to share state, use a compound component with a context shaped as `state / actions / meta`:

```tsx
type ComposerContextValue = {
  state: { text: string; attachments: Attachment[] };
  actions: { setText: (text: string) => void; send: () => void };
  meta: { isSending: boolean; maxLength: number };
};

const ComposerContext = createContext<ComposerContextValue | null>(null);

function useComposer() {
  const value = useContext(ComposerContext);
  if (!value) throw new Error('Composer parts must be rendered inside Composer.Provider');
  return value;
}
```

The provider receives `state`, `actions` and `meta` from a hook (`useComposerState`), so the same parts can be driven by different state sources (a new message, an edit).

## The layered UI kit

```
primitives  →  kit components  →  shared  →  feature parts
(Box, Text,    (Button, Input,    (EmptyState,  (OrderRow,
 Pressable)     Sheet, Badge)      ErrorState)   OrderCard)
```

- Screens use the kit, not raw platform elements, when a kit piece exists.
- Kit components consume tokens only and expose variants, not style overrides.
- A missing kit piece is added to the kit (and reused), not hand-styled inside a feature.

## Memoization

- With React Compiler on: no manual `useMemo` / `useCallback` / `memo`.
- Without it: memoize only where a measurement shows a cost or identity is required (a context value, an effect dependency, a prop of a memoized child).
- Measure before optimizing; when a measurement shows a real cost, fix that spot and leave a one-line comment with the reason.
