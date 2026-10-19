# Frontend testing

## The policy belongs to the project

How much frontend testing a project does is its decision: logic only, plus component tests, plus E2E. Read it from the project docs or the existing tests. If nothing is recorded, ask before choosing. Two things are never optional:

- Pure logic you extract (rules, calculations, mappings, reducers) gets unit tests whenever the project tests logic at all.
- Every bug fix comes with a test that fails before the fix, at the lowest level the project tests. If the project only tests pure logic and the bug is in the wiring, say that it cannot get a failing test and ask.

## How tests are written

- **Test behavior**, named as a sentence: `it('shows the empty state when the user has no orders')`.
- **Pure logic gets unit tests.** This is why logic lives in pure functions: `calculateTotal`, `statusToBadgeTone`, the flow reducer.
- **Fake only at the edges** of the system: the network layer, analytics, push, payments, storage providers, device APIs.
- **Never replace your own modules** with `vi.mock` / `jest.mock` (hooks, services, components). The service mock is not module mocking: it is a second implementation of the service signature, chosen through the same config switch the app uses.
- **One render helper** (`renderWithProviders`) mounts the app's providers (query client, router, i18n, theme) with a fresh query client per test and `retry: false`, so error states show at once and no cache leaks between tests.
- **Prefer the user's view** in component tests: query by role and label, interact like a user, assert on what is visible.
- **Drive the four states** with the service mock's scenarios (ready, empty, error, slow).
- **Tests that cost money or hit real providers** are opt-in scripts with a dry run by default, outside CI.

## Examples

```ts
// features/orders/logic/order-total.test.ts
describe('calculateTotal', () => {
  it('sums line totals in cents', () => {
    expect(calculateTotal([{ priceCents: 250, quantity: 2 }, { priceCents: 100, quantity: 1 }])).toBe(600);
  });

  it('returns 0 for an empty cart', () => {
    expect(calculateTotal([])).toBe(0);
  });
});
```

```tsx
// features/orders/components/orders-screen.test.tsx
it('shows a retry action when loading orders fails', async () => {
  setOrdersScenario('error'); // switches the orders service mock to its error scenario
  renderWithProviders(<OrdersScreen />);
  expect(await screen.findByRole('button', { name: 'Retry' })).toBeVisible();
});
```

## Finish line

The project's types, lint, format and tests for what you touched pass with zero errors. Never remove, skip or weaken a test or a check to make it pass.
