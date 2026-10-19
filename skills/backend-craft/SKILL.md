---
name: backend-craft
description: "Opinionated backend engineering craft for APIs and services (stack-agnostic, examples in TypeScript with Hono, Zod and Drizzle): a module per feature with controller, service, repository and DTO schemas, every boundary parsed with a schema, errors with stable codes mapped at the edge, vendors behind role modules, injected dependencies, additive APIs and safe migrations - and every change shipped with tests through the real entry point against a local test-only database, with factories and shared fakes. Use when building, changing, fixing, refactoring or reviewing server code - endpoints, route handlers and server actions, business rules, database schemas and migrations, auth, integrations and webhooks, background jobs, queues and cron - e.g. 'add an endpoint', 'add a migration', 'handle the payments webhook', 'fix this API bug', 'write backend tests', 'review my backend'. Not for UI code (use frontend-craft)."
license: MIT
metadata:
  author: AgustinOberg
  version: 1.0.0
---

# Backend craft

How backend code is written: modules that read the same way, boundaries that never trust input, errors clients can rely on, and tests that prove every change. The rules are stack-agnostic; examples use TypeScript, Hono, Zod and Drizzle only to make a rule concrete. Apply the same idea in whatever stack the project uses.

## Important: precedence

1. The project's own written rules win: AGENTS.md, CLAUDE.md, specs, the technical design, and what the developer says in the conversation.
2. A convention the codebase already follows consistently wins inside the module you touch. Stay consistent there and mention the divergence once. Never refactor outside the task to "fix" it.
3. Otherwise, apply this skill.

## Critical

- **Backend tests are always required.** Every backend change ships with its tests, whatever else the project's test policy says. Tests go through the real entry point against a **local database used only for tests**, never the development, staging or production database. If the project has no backend test setup yet, building it is the first task (ask only which tools to use). Details: `references/testing.md`.
- Controller → service → repository; the controller never holds business rules.
- Every input is parsed with a schema; errors carry stable codes; nothing unknown leaks to the client.
- Invariants live in database constraints; lists are paginated.
- Ask before adding a dependency, choosing a library the project has not decided, or touching infrastructure.

## Workflow

### Step 1: Understand the task
Restate the objective in one line and list the acceptance criteria. If the approved behavior does not hold (it misses an edge case, contradicts another approved piece, or breaks for the client), stop and propose the change. Do not implement it anyway and do not "fix" it silently.

### Step 2: Read before writing
Read the closest existing module and copy its shape: routing, controller, service, repositories, DTOs, errors, auth, migrations, tests, factories and fakes. Read only the files the task touches.

### Step 3: Check the stack decisions
Some choices belong to the project: language strictness, framework, data access/ORM, validation, auth, HTTP client, how the client/server contract stays in sync, lint/format tooling and the `check` command. Read them from the project docs or the code. If the task needs one that is not decided, stop and ask with options and a recommendation. Offer the choice in `references/stack-profile.md` only when it meets the requirements. Never adopt a library silently because it is the one you know.

### Step 4: Check platform limits only when it matters
Ordinary work (an endpoint, a field, a query) needs no audit. Check the documented limits only when the work involves uploads or large payloads, concurrency, long-running or background jobs, cron, long-lived connections, heavy processing, bulk data operations or third-party quotas. If it does not fit, stop and present alternatives. Details: `references/platform-limits.md`.

### Step 5: Plan the slice
Name the files before coding, in this order:
database schema change and migration (if any) → DTO schemas → errors and pure rules → repository → service → controller → router mount → tests with their factories and fakes.

### Step 6: Implement
Apply the core rules below. Keep contracts explicit, enforce authorization where data is accessed, handle errors and idempotency deliberately, and keep logs free of user content, personal data and secrets.

### Step 7: Test
Write the tests per `references/testing.md`: per endpoint, the happy path, validation per field, authentication and roles, ownership, not found, conflicts, limits and pagination. A bug fix always gets a test that fails before the fix.

### Step 8: Finish
Run the backend tests against the local test database plus types, lint and format: zero errors. Self-review with `references/checklist.md`. No dead code, debug flags, commented-out blocks or stray TODOs. Comments explain why, not what. Tell the developer what changed, what you ran and anything you could not run; a check that did not run is never reported as passed.

## Core rules

**A module per feature.** Controller (validate input, call the service, shape the response) → service (business rules) → repository (data access), plus its `dto/` schemas and an `index`. A central router only mounts modules. Details and a full example: `references/module-structure.md`.

**Classes where they earn it.** Classes for services, repositories, adapters and errors; plain functions for everything else. A module starts with one repository; split it by read and write (and by search or stats) when the module grows.

**Dependencies point one way:** entry → logic → data → external. Lower layers never import upper ones. A module exposes a small public surface and nobody reaches into another module's internals. Business rules, calculations and mappings are pure functions.

**One seam to the platform.** Bindings, env, the current user and language come through small accessors. Each layer receives only what it needs.

**Inject dependencies with production defaults** (clock, providers, notifiers) so tests can pass fakes without module mocking. Heavy clients are built once per runtime instance, not at import time.

**Schema-first boundaries.** Every input that crosses a boundary (body, query, params, headers, third-party responses, stored values) is parsed with a schema, and its type is derived from it (DTOs). No `any`, no unchecked casts, no suppressed type or lint errors.

**Errors have stable codes.** Domain errors are typed classes with a machine code. The edge maps them to transport (HTTP status plus `{ error, code }`). Anything unknown becomes a generic internal error; never leak internal messages. Details: `references/boundaries-and-errors.md`.

**Data integrity.** Invariants are database constraints (unique and partial indexes, foreign keys, checks), not read-then-write checks; the repository translates a violation into the domain error. Writes that must succeed together are atomic. Side effects (email, push, payments) run after the commit. Queries filter and sort on indexed columns, and related rows load with a join or one batched query, never one query per row.

**Lists are paginated from day one:** a cursor over a stable order, a default limit capped by a `MAX_PAGE_SIZE`, and `{ items, nextCursor }` in the response.

**Resilience.** Every external call has a timeout. Retry only what is idempotent. Degrade instead of failing when a fallback exists (secondary provider, cached value, "no suggestion").

**Vendors behind role modules.** AI providers, payments, email, push, search, storage and other SDKs live behind one module named by its role (`payments`, not the vendor). Only that module imports the SDK, translates vendor errors into the app's vocabulary and parses vendor payloads. The core framework (router, ORM, test runner) is used directly and idiomatically, never wrapped. Add an interface only where a change is expected or a test needs a seam.

**Config and secrets.** Declared in one typed place and referenced by name. A missing optional secret turns its feature off; a missing required one fails loudly at startup. No secrets in code, config, tests, fixtures or logs.

**Evolve without breaking clients.** Add fields; do not change or remove them unless the project decided otherwise. Migrations add. Data migrations can run twice safely. Never edit a migration that has already run outside your machine; add a new one. Never read or write production data from development work; data changes ship as idempotent data migrations through the normal deploy.

**Abuse and observability.** Sign-in, one-time codes and expensive endpoints (AI, email, uploads) are rate limited. Inbound webhooks verify their signature on the raw body and skip events they already processed. Every request carries a request id that logs and error tracking include.

**Privacy by default.** Logs and analytics carry no user content, personal data or secrets. Constants carry their unit in the name (`TIMEOUT_MS`, `MAX_UPLOAD_BYTES`); limits live in one place per module.

## When to stop and ask

These are the moments to stop writing code and talk to the developer. Present the options, recommend one with the reason, and wait.

- **Library vs hand-built.** Before hand-building something a maintained library already solves, or before adding any new dependency: name the candidates (and hand-built), compare fit, size, maintenance and DX, recommend.
- **Code that smells.** Before writing any of these, justify it in one line or pick the alternative; if neither works, ask: duplicated logic, duplicated or derived data stored separately, a file that keeps growing, a new abstraction layer, a cast or a suppressed error.
- **A spec that does not hold.** When the approved behavior breaks for the client, misses an edge case or contradicts another approved piece, propose the change.
- **The platform cannot do it.** When the work collides with documented limits, present alternatives instead of a workaround that breaks in production.
- **Infrastructure.** Never choose the cloud, hosting or deploy platform, create paid resources or deploy without the developer's approval. Recommend; the developer decides.

## Fixing a bug

1. Establish expected vs actual behavior, the request that fails, and the environment.
2. Reproduce it first, ideally as a failing test through the real entry point. If it does not reproduce, do not guess a fix: check whether it is already fixed, ask for the missing details, and stop.
3. Find the root cause, not the symptom. Separate a bug from a missing feature or a spec gap. If the honest fix changes intended behavior or a public contract (API, schema, event), ask first.
4. Write the regression test: an endpoint test against the local test database that fails before the fix (a unit test of the pure rule can go next to it).
5. Apply the smallest safe fix that follows existing patterns and keeps contracts additive. Confirm the test passes and related tests still pass.
6. No opportunistic refactors, unrelated fixes or dependency upgrades in the same change.

## Examples

**Example 1: "Add an endpoint to cancel an order"**
1. Read the closest module (for example `orders`) and copy its shape.
2. Plan: no migration → `dto/cancel-order.dto.ts` → `OrderNotCancellable` error → `OrdersRepository.cancel` → `OrdersService.cancel` (rules: owner only; `pending` becomes `cancelled`; already `cancelled` returns it unchanged; any other status throws `OrderNotCancellable`) → controller route `POST /orders/:id/cancel` → `test/modules/orders/cancel-order.test.ts`.
3. Errors: `OrderNotFound` (404, also for another user's order), `OrderNotCancellable` (409).
4. Tests: cancels a pending order; 401 without a session; 404 for another user's order; 409 when already paid; second cancel is a no-op.
Result: a module that reads like every other module, with its behavior proven against the local test database.

**Example 2: "Send a welcome email when a user signs up"**
The email provider lives behind an `email` role module with a timeout. The service receives it by injection with the production default. Tests use the shared email fake (success and failure variants) and assert that signup still succeeds when email fails, if that is the agreed behavior; if it is not specified, ask.

**Example 3: "Let users upload videos up to 2 GB"**
This collides with request-size and CPU limits on most edge and serverless runtimes. Stop and present alternatives (direct-to-storage upload with a presigned URL, processing in a background job) before writing code.

## Common mistakes

- **"The controller has business rules."** Move them to the service; the controller validates, calls and shapes.
- **"The service imports the vendor SDK."** Move the SDK into its role module and inject it.
- **"Tests mock the repository."** Run them through the entry point against the local test database; fake only external services.
- **"The test needs a user row."** Use the user factory; never hand-write rows or copy them between tests.
- **"The client needs a field renamed."** Add the new field and keep the old one, unless the project decided to break the contract.
- **"The codebase does it differently."** Stay consistent inside that module and mention the divergence once.

## References

- `references/module-structure.md` — folder layout and a complete module: database schema, DTOs, repository, service, controller, the app factory, dependency injection, queries.
- `references/boundaries-and-errors.md` — the error and validation shape, error classes and edge mapping, timeouts, role modules, data integrity, pagination, webhooks, auth and rate limits, API evolution and migrations, logs.
- `references/testing.md` — the mandatory backend test setup: local test database, entry-point tests, factories, shared fakes, coverage per endpoint.
- `references/platform-limits.md` — when to check limits, common collisions and alternatives, and how to compare runtimes and deploy targets.
- `references/stack-profile.md` — preferred tools, offered only as a recommendation.
- `references/checklist.md` — self-review and code review checklist.
