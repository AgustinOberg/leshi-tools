# Checklist

Use it to self-review before calling a task done, and to review someone else's backend code.

## Self-review

- **Structure:** a module per feature; controller → service → repository; DTO schemas per endpoint; the central router only mounts; dependencies point one way; no reach into another module's internals.
- **Boundaries:** every input parsed with a schema; types derived; no `any`, casts or suppressions.
- **Errors:** typed domain errors with stable codes; mapped at the edge to `{ error, code }`; nothing unknown leaked.
- **Data integrity:** invariants enforced by database constraints; writes that belong together are atomic; side effects after the commit; queries on indexed columns; no N+1.
- **Lists:** paginated with a cursor, a capped limit and `{ items, nextCursor }`.
- **Resilience:** timeouts on every external call; retries only for idempotent work; fallbacks where they exist; webhooks verified and deduplicated.
- **Abstraction:** vendor SDKs only inside their role module; dependencies injected with production defaults; no speculative layers; the core framework used idiomatically.
- **Security:** authentication at the edge; authorization where data is accessed; queries scoped by owner; sensitive and expensive endpoints rate limited; no secrets in code, tests or logs; no user content or personal data in logs or analytics; request ids in logs.
- **Evolution:** the API change is additive; migrations add and were not edited after running; data migrations are idempotent; production data untouched.
- **Tests:** every changed endpoint has tests through the entry point against the local test database, with factories and shared fakes injected through the app factory, covering validation, auth, ownership, not found, conflicts, limits and pagination; a regression test for every fix.
- **Questions:** every stop-and-ask moment (new dependency, hand-built vs library, smell, a spec that does not hold, a platform limit, infrastructure) was raised, not decided silently.
- **Finish:** types, lint, format and tests pass with zero errors; no dead code, debug flags or stray TODOs.

## Reviewing someone else's code

Review in this order of priority:

1. **Correctness against the requirements:** every acceptance criterion implemented; behavior nobody asked for is flagged.
2. **Regressions:** shared code, contracts, migrations or config changed; callers affected; API changes that are not additive.
3. **Security:** input validation, authentication and authorization, ownership checks, injection, secrets in code or logs, personal data exposure, unsafe deserialization.
4. **Data integrity:** migrations, transactions, constraints vs read-then-write races, idempotency, concurrency, webhook replays.
5. **Tests:** a backend change without tests against the local test database is a **blocker**; tests that cannot fail; tests that mock the project's own modules or touch a non-test database.
6. **Silent decisions:** a new dependency or undecided library introduced; something hand-built that a maintained library solves; an approved spec worked around; risky work (uploads, concurrency, jobs, cron, long connections, heavy processing, bulk operations, quotas) built without checking platform limits.
7. **Craft:** walk the self-review list above.

Make each finding verifiable: file and line, what happens, a concrete input that triggers it, and why it matters. Rank it:

- **Blocker:** must be fixed before merging.
- **Major:** a deviation that hides or will cause a bug (unvalidated input, a vendor SDK leaking across the code, an unknown error leaked to the client, a query not scoped by owner, an invariant checked only in code, an unpaginated list).
- **Minor:** everything else.

Code that follows a different convention already consistent in that module is not a finding. Say plainly when you found nothing significant.
