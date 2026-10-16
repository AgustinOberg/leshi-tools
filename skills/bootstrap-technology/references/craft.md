# How leshi writes code (craft, shared)

This is how code is written in a leshi project: the default craft for every implementation workflow and every `leshi-*` agent that writes or reviews code. It is stack-agnostic on purpose. Examples name a common stack only to make a rule concrete; apply the same idea in whatever the Technical Spec chose.

## 1. Precedence

1. The approved SPECs, the Technical Spec and a project extension (`.leshi/extensions/<id>.md`) win over this file.
2. A documented project rule (AGENTS.md, CLAUDE.md) or a convention the codebase follows consistently wins inside the module you touch: stay consistent there and raise the divergence once as a finding. Never refactor outside the brief to "fix" it.
3. Otherwise, apply this file.

## 2. Decisions that belong to the project — ask, never assume

Some choices depend on the stack or the product. They are recorded in the Technical Spec (or the config); read them there. When the task needs one that is not decided, stop and ask, with options and a recommendation (the `references/stack-profile.md` choice when it meets the requirements), before introducing it. Never adopt one silently because it is what you know.

- Language and its strictness (e.g. TypeScript strict).
- Libraries for server state/caching, cache-key management, forms, styling, validation, data access/ORM, i18n, HTTP client.
- How the client/server contract is kept in sync: types duplicated per app with contract tests, or a shared package.
- Test policy per layer (frontend logic, components, E2E) and the tools. Backend tests are not optional (§9); the project only chooses their tools.
- Lint/format tooling and the project's `check` command.

## 3. When to stop and question

These are the moments to stop writing code and talk to the developer (in a workflow, through its decision gate; inside an agent, as `decisions needed`). Present the options, recommend one with the reason, and wait.

- **Library vs hand-built.** Before hand-building something a maintained library already solves, or before adding a new dependency: name the candidates (and hand-built), compare fit, size, maintenance and DX, recommend.
- **Code that smells.** Before writing any of these, justify it in one line or pick the alternative; if neither works, ask: an effect that syncs state, duplicated or derived state stored separately, a component or file that keeps growing, a new abstraction layer, manual memoization, a boolean prop that switches behavior, a cast or suppressed error.
- **A spec or design that does not hold.** When the approved behavior breaks for the user, misses an edge case (empty, error, offline, permissions, limits) or contradicts another approved piece: stop and propose the change through the spec-revision flow. Do not implement it anyway, and do not "fix" it silently.
- **The platform cannot do it.** Ordinary work (a new endpoint, a field, a query, a screen) fits the platform; do not audit limits for it. Check the documented limits (with source and date) only when the work involves what usually collides with them: file uploads or downloads and large payloads, concurrent or parallel work, long-running or background jobs, cron/scheduled tasks, long-lived connections (websockets, streaming), heavy processing (media, documents, AI), bulk data operations, third-party quotas and rate limits. If it does not fit (e.g. a multi-GB upload through an edge function), stop and present alternatives (direct-to-storage upload, background job, another service) instead of a workaround that breaks in production.

## 4. Developer experience first; abstract what can change

- Use the core framework directly and idiomatically (router, styling engine, ORM, test runner). Wrapping the core hurts DX and buys nothing.
- Everything that is not core and might be replaced — third-party services and SDKs: AI providers, payments, analytics, email, push, search, storage providers, the HTTP client — lives behind one app-shaped module named by its **role**, not its vendor. Only that module imports the SDK; the rest of the code calls the module. Swapping the vendor touches one place.
- Translate vendor errors and payloads into the app's own small vocabulary inside that module; parse what the vendor returns there.
- A cross-cutting library used everywhere (i18n, theme) gets a thin hook/helper when it makes call sites simpler.
- Add an interface only where a change is expected or a test needs a seam. No speculative layers.

## 5. Structure

- **Slice by feature, then by role.** Each feature owns its folder; inside it, files split by role (entry, logic, data, schemas). A central entry only mounts features.
- **One concern per file, small files.** When a file does two things or keeps growing, split it.
- **Logic in pure functions.** Rules, state machines, calculations and mappings are pure and unit-testable. Edges (routes, controllers, components) only connect.
- **Dependencies point one way:** entry → logic → data → external. Lower layers never import upper ones. A feature exposes a small public surface (an index) and nobody reaches into another feature's internals.
- **Read what is there first.** Before writing, read the similar feature that already exists and copy its shape.

## 6. Types, boundaries and errors

- **Schema-first.** Every input crossing a boundary (request body, query, params, headers, third-party responses, stored values, route params) is parsed with a schema; its type is derived from the schema (DTOs). No hand-written duplicate types for the same shape.
- No `any`, no unchecked casts, no suppressed type or lint errors. Model states as discriminated unions (`loading | error | ready`) with one discriminant name per project.
- Constants carry their unit in the name (`TIMEOUT_MS`, `MAX_UPLOAD_BYTES`); limits and quotas live in one place per feature; values that are guesses are labelled as such.
- Config and secrets are declared in one typed place and referenced by name. A missing optional secret turns its feature off; a missing required one fails loudly at startup.
- **Errors have stable machine codes.** Domain errors are typed; the edge maps them to transport (HTTP status + `{ error, code }`) and the client maps codes to localized messages. Anything unknown becomes a generic internal/unavailable error, never a leaked message.
- Degrade instead of failing when a fallback exists (secondary provider, cached value, "no suggestion"). Every external call has a timeout; retry only what is idempotent.
- Privacy by default: logs and analytics carry no user content, personal data or secrets.

## 7. Frontend

- **Components render; hooks think.** State, effects, data fetching and handlers live in custom hooks named by purpose (`useXScreen`, `useXForm`, `useGetX`), one hook per file. The component reads the hook and returns markup.
- **Avoid effects.** Derived values are computed during render; reactions to a user action go in the event handler; resetting state uses a key or a reducer. An effect is only for synchronizing with an external system (subscription, device API, imperative widget) and gets a one-line comment saying why.
- **Small components, composed.** Split a component as soon as it has two responsibilities. Prefer composition (children, named parts, compound components sharing a context of `state / actions / meta`) over boolean props that switch behavior; variants are separate parts.
- **Pages/routes only compose** providers and parts; no logic in them.
- **A layered UI kit:** primitives → kit components → shared → feature parts. Screens use the kit, not raw platform elements, when a kit piece exists.
- **Place state by scope:** server data in the server-state cache (centralized keys, invalidate after mutations instead of editing the cache by hand, optimistic updates only on purpose); local UI state in the hook; flow state in a reducer owned by the flow; persisted preferences in one storage module with a registry of keys. No global store by default.
- **Data in three tiers:** one HTTP client (auth, language, refresh) → one service per backend area returning parsed data → query/mutation hooks. Screens never call the HTTP client directly. When the backend is not ready, a mock implements the same service signature with realistic latency, coded errors and switchable scenarios (empty, error, slow).
- **Every data screen has four states:** loading (skeletons shaped like the content), error with retry, empty, ready.
- **Tokens only:** spacing scale, radii, semantic color roles, one type ramp; no literal colors or sizes in components. Light and dark are each designed.
- i18n from the first screen (typed keys, `feature.screen.element`), accessibility as part of done (labels, roles, hit areas, reduced motion). Motion is short and has a purpose.
- No manual memoization when the framework or compiler handles it; measure before optimizing.

## 8. Backend

- **A module per feature:** controller (validate input, call the service, shape the response) → service (business rules) → repository (data access), plus its `dto/` schemas and an index. A central router mounts modules.
- **Classes for services, repositories, adapters and errors;** plain functions for everything else.
- **Split repositories by read and write** (and by search or stats) when a module grows.
- **One seam to the platform:** bindings, env, current user and language come through small accessors; each layer receives only what it needs.
- **Inject dependencies with production defaults** (clock, providers, notifiers) so tests can pass fakes without module mocking.
- **Evolve without breaking clients:** add fields, do not change or remove them, unless the project decided otherwise. Migrations add; data migrations can run twice safely. Never touch production data.
- Heavy clients are built once per runtime instance, not at import time.

## 9. Tests

- Test behavior, named as a sentence ("returns 409 when the item already exists").
- Pure logic gets unit tests.
- Fake only at the edges of the system (third-party APIs, AI, push, payments, email, storage providers). Never mock your own modules.
- Tests that cost money or hit real providers are opt-in scripts with a dry run by default, outside CI.
- Every bug fix comes with a test that fails before the fix.

### Backend tests — always required

Every backend change ships with its tests, whatever else the project's test policy says. If the project has no backend test setup yet, the first backend task builds it (asking only for the tools, §2).

- **A local database only for tests.** Its own config (e.g. a test env file or the runtime's local emulator), created from the same migrations before the run and cleaned between suites. Never the development, staging or production database.
- **Through the real entry point.** Tests call the app the way a client does (an HTTP request to the running app or its in-process handler), so validation, auth, errors and data access are all exercised. One test file per endpoint or use case, grouped by module, mirroring the source tree.
- **Data from factories.** One factory per entity in a shared test folder builds valid data from the model's own constraints (lengths, enums, required fields) with random values; generators persist it. No hand-written rows copied between tests.
- **Shared fakes per external service,** each with success and failure variants and a reset, in the same shared test folder. Tests never touch the network.
- **Shared assertion helpers** for the error shape (field errors, error codes) and auth headers; file fixtures in a fixtures folder.
- **Cover per endpoint:** the happy path, validation errors per field, authentication and roles, ownership (another user's data is never returned or changed), not found, conflicts and limits.

## 10. Finish line

- Zero errors: the project's types, lint, format and tests pass before you call it done.
- Comments explain why, not what. No dead code, debug flags, commented-out blocks or stray TODOs left behind.
- Self-review against the checklist below before handing over.

## 11. Review checklist

Report each line as ✅ / ❌ / n/a, with the file for every ❌.

- Structure: feature slice respected; one concern per file; dependency direction; no reach into another feature's internals.
- Boundaries: every input parsed with a schema; types derived; no `any`, casts or suppressions.
- Errors: coded, mapped at the edge, nothing unknown leaked; timeouts on external calls.
- Abstraction: vendors only inside their role module; no speculative layers; core framework used idiomatically.
- Frontend: components render and hooks hold logic; no effect without an external system; four states; tokens only; i18n; accessibility.
- Backend: controller → service → repository; DTOs per endpoint; additive API; safe migrations.
- Tests: behavior-named; fakes only at the edges; regression test for fixes; checks green.
- Backend tests: every changed endpoint has tests against the local test database, with factories and shared fakes, covering validation, auth, ownership and errors.
- Questions: every stop-and-question moment (§3) was raised, not decided silently.
