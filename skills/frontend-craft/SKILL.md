---
name: frontend-craft
description: "Opinionated frontend engineering craft for web and mobile apps (React, React Native, Expo, Vite, any stack): components that only render, logic in purpose-named hooks, no effects without an external system, small composed components, server state in a query cache, a three-tier data layer, four states per data screen, design tokens only, i18n and accessibility from the first screen. Use when building, changing, fixing, refactoring or reviewing UI code - screens, components, hooks, navigation, forms, styling, i18n, accessibility, client state or data fetching - e.g. 'build this screen', 'add a form', 'refactor this component', 'fix this UI bug', 'review my frontend'. Not for API endpoints, route handlers, server actions, databases or migrations (use backend-craft)."
license: MIT
metadata:
  author: AgustinOberg
  version: 1.0.0
---

# Frontend craft

How frontend code is written: clean, small, predictable and easy to change. The rules are stack-agnostic; examples use TypeScript and React only to make a rule concrete. Apply the same idea in whatever stack the project uses.

## Important: precedence

1. The project's own written rules win: AGENTS.md, CLAUDE.md, specs, design docs, and what the developer says in the conversation.
2. A convention the codebase already follows consistently wins inside the module you touch. Stay consistent there and mention the divergence once. Never refactor outside the task to "fix" it.
3. Otherwise, apply this skill.

## Critical

- Components render; hooks think.
- No effect unless it synchronizes with an external system.
- Every data screen ships loading, error with retry, empty and ready.
- Tokens, typed i18n keys and kit components only: no literal colors, sizes or user-facing strings.
- Every response, param and stored value is parsed with a schema.
- Ask before adding a dependency or choosing a library the project has not decided.

## Workflow

### Step 1: Understand the task
Restate the objective in one line and list the acceptance criteria. If there is a design (hi-fi, UI kit), it is the target. If the requirement or design does not hold for the user (it misses the empty, error, offline or permission case, or contradicts something already approved), stop and propose the change. Do not implement it anyway and do not "fix" it silently.

### Step 2: Read before writing
Find the closest existing screen or feature and copy its shape: folder layout, kit components, hooks, data layer, tokens, tests. Read only the files the task touches.

### Step 3: Check the stack decisions
Some choices belong to the project: language strictness, server-state library, forms, styling, validation, i18n, HTTP client, how the client/server contract stays in sync, frontend test policy, lint/format tooling and the `check` command. Read them from the project docs or the code. If the task needs one that is not decided, stop and ask with options and a recommendation. Offer the choice in `references/stack-profile.md` only when it meets the requirements. Never adopt a library silently because it is the one you know.

### Step 4: Plan the slice
Name the files before coding, in this order:
route/page (composition only) → feature parts (components) → hooks (state and orchestration) → pure logic (`logic/`) → query/mutation hooks and query keys → service (plus a mock when the backend is not ready) → schemas.

### Step 5: Implement
Apply the core rules below. Cover loading, error with retry, empty and ready, plus the edge cases the feature implies (offline, permissions, limits, long text, second language).

### Step 6: Integrate only through the contract
Call the backend only through the agreed contract and parse every response with a schema. If the contract is missing something or is wrong, stop and say so. Never change backend code from a frontend task and never invent endpoints.

### Step 7: Test
Write the tests the project's policy requires (`references/testing.md`). A bug fix always gets a regression test that fails before the fix. If the project has no frontend test policy, ask instead of choosing one.

### Step 8: Finish
Run the project's types, lint, format and the tests for what you touched: zero errors. Self-review with `references/checklist.md`. No dead code, debug flags, commented-out blocks or stray TODOs. Comments explain why, not what. Tell the developer what changed, what you ran and anything you could not run; a check that did not run is never reported as passed.

## Core rules

**Components render; hooks think.** State, effects, data fetching and handlers live in custom hooks named by purpose (`useOrdersScreen`, `useCheckoutForm`, `useGetOrders`), one hook per file. The component calls its one purpose hook (plus presentational hooks such as translation or theme) and returns markup. Pages and routes only compose providers and parts. Details: `references/components-and-hooks.md`.

**Avoid effects.** Derived values are computed during render. Reactions to a user action go in the event handler. Resetting state uses a key or a reducer. Data comes from query hooks. An effect is only for synchronizing with an external system (subscription, device API, imperative widget) and carries a one-line comment saying why.

**Small components, composed.** Split a component as soon as it has two responsibilities. Prefer composition (children, named parts, compound components sharing a context of `state / actions / meta`) over boolean props that switch behavior. Variants are separate parts.

**A layered UI kit.** Primitives → kit components → shared → feature parts. Screens use the kit, never raw platform elements, when a kit piece exists.

**Structure by feature.** Each feature owns its folder and exposes a small public surface (an index). Inside, files split by role. One concern per file. Logic (rules, calculations, mappings, state machines) lives in pure functions that import nothing from React, services or the HTTP client. Dependencies point one way: routes → components → hooks → (pure logic, or query hooks → services → HTTP client and role modules). Nobody reaches into another feature's internals. Details: `references/architecture.md`.

**State by scope.** Server data lives in the server-state cache with centralized query keys; invalidate after mutations instead of editing the cache by hand; optimistic updates only on purpose. Local UI state lives in the hook. Flow state lives in a reducer owned by the flow. Persisted preferences live in one storage module with a registry of keys. No global store by default.

**Data in three tiers.** One HTTP client (auth, language, token refresh) → one service per backend area that returns parsed data → query/mutation hooks. Screens never call the HTTP client directly. When the backend is not ready, a mock implements the same service signature with realistic latency, coded errors and switchable scenarios (empty, error, slow). Details: `references/state-and-data.md`.

**Every data screen has four states:** loading (skeletons shaped like the content), error with retry, empty, ready.

**Types and boundaries.** Every input that crosses a boundary (API responses, route params, stored values, third-party payloads) is parsed with a schema and its type is derived from the schema. No `any`, no unchecked casts, no suppressed type or lint errors. Model states as discriminated unions with one discriminant name per project.

**Errors have codes.** The client maps stable error codes to localized messages. Unknown errors show a generic message; never render raw server or vendor messages.

**Vendors behind role modules.** Analytics, payments, push, storage, AI providers and similar SDKs live behind one module named by its role (`analytics`, not the vendor name). Only that module imports the SDK. The core framework (router, styling engine, query library) is used directly and idiomatically, never wrapped.

**UI quality.** Tokens only: spacing scale, radii, semantic color roles, one type ramp; no literal colors or sizes in components. Each theme is designed, not inverted. i18n from the first screen with typed keys (`feature.screen.element`). Accessibility is part of done: labels, roles, hit areas, focus order, dynamic type, reduced motion. Motion is short and has a purpose. Details and the design process when there is no design: `references/ui-design.md`.

**Performance.** With React Compiler on, no manual `useMemo`, `useCallback` or `memo`. Without it, memoize only where a measurement shows a cost or identity is required (a context value, an effect dependency, a memoized child). Long lists are paginated and virtualized; independent requests start together, never as a waterfall. Measure before optimizing.

**Security.** Auth tokens live in secure storage (Keychain/Keystore, or an httpOnly cookie on web), never in plain key-value storage. Never render untrusted HTML without a sanitizer. Nothing secret ships in the client bundle.

## When to stop and ask

These are the moments to stop writing code and talk to the developer. Present the options, recommend one with the reason, and wait.

- **Library vs hand-built.** Before hand-building something a maintained library already solves, or before adding any new dependency: name the candidates (and hand-built), compare fit, size, maintenance and DX, recommend.
- **Code that smells.** Before writing any of these, justify it in one line or pick the alternative; if neither works, ask: an effect that syncs state, duplicated or derived state stored separately, a component or file that keeps growing, a new abstraction layer, manual memoization, a boolean prop that switches behavior, a cast or a suppressed error.
- **A spec or design that does not hold.** When the approved behavior breaks for the user, misses an edge case or contradicts another approved piece, propose the change instead of building around it.
- **The platform cannot do it.** When the work touches device or browser limits (background work, large files, permissions, offline storage, long lists, heavy media), check the documented limits and propose an alternative instead of a workaround that breaks in production.

## Fixing a bug

1. Establish expected vs actual behavior and the environment (device, OS or browser, version, account state).
2. Reproduce it first, with the smallest set of steps. If it does not reproduce, do not guess a fix: check whether it is already fixed, ask for the missing details, and stop.
3. Find the root cause, not the symptom. Separate a bug from a missing feature or a spec gap. If the honest fix changes intended behavior or a contract, ask first.
4. Write a regression test at the lowest level that reproduces it and confirm it fails.
5. Apply the smallest safe fix that follows existing patterns. Confirm the test passes and re-run the reproduction.
6. No opportunistic refactors, unrelated fixes or dependency upgrades in the same change.

## Examples

**Example 1: "Add a screen that lists the user's orders"**
1. Read the closest list screen and copy its folder, hooks and kit usage.
2. Plan: `orders` route (composition) → `OrdersScreen`, `OrderList`, `OrderRow`, `OrdersSkeleton` → `useOrdersScreen` → `logic/order-status.ts` → `useGetOrders` (paginated) with keys from `orderKeys` → `ordersService.list` parsing with `OrderPageSchema` (plus a mock if the endpoint does not exist yet).
3. Implement the four states, a virtualized list, i18n keys `orders.list.*`, accessible rows, tokens only.
4. Unit-test the pure mapping (for example, status → badge tone). Run `check` and tests.
Result: a thin route, a component that only renders, all logic in hooks, data parsed at the boundary.

**Example 2: "The total doesn't update when I change the quantity"**
Reproduce, then find the cause: the total was copied into state and synced with an effect. Write the regression test at the lowest level the project tests (a hook or component test that changes the quantity and expects the new total); it fails before the fix. Delete the duplicated state and the effect, compute the total during render with the pure `calculateTotal`, and unit-test it. If the project tests only pure logic, say the wiring bug cannot get a failing test and ask.

**Example 3: "Add a date picker to the form"**
No date library is decided in the project. Stop and ask: the platform's native picker vs one or two maintained libraries vs hand-built, with size, accessibility and maintenance compared, and a recommendation.

## Common mistakes

- **"I need an effect to keep X in sync with Y."** Compute X from Y during render. If X must reset when an id changes, use a key.
- **"This component needs a new boolean prop."** Make a separate part or a variant, or use composition.
- **"The screen fetches with the HTTP client directly."** Move the call into the area's service and expose it through a query hook.
- **"The backend isn't ready."** Build a mock with the same service signature and scenarios; switch implementations in one place.
- **"There is no design."** Follow the design process in `references/ui-design.md` and get a direction approved before building screens.
- **"The codebase does it differently."** Stay consistent inside that module and mention the divergence once.

## References

- `references/architecture.md` — feature slices, folder layout, dependency direction, role modules, config, errors and privacy.
- `references/components-and-hooks.md` — components vs hooks, effects decision table, composition patterns, the layered kit.
- `references/state-and-data.md` — state by scope, the HTTP client → service → query hook tiers, query keys, mutations, mocks, the four states.
- `references/ui-design.md` — tokens, hierarchy, navigation semantics, motion, layouts by window size, accessibility, i18n, restrained polish, and how to design when there is no design.
- `references/testing.md` — what to test and how, fakes at the edges, regression tests.
- `references/stack-profile.md` — preferred tools, offered only as a recommendation.
- `references/checklist.md` — self-review and code review checklist.
