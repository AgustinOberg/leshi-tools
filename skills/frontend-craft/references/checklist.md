# Checklist

Use it to self-review before calling a task done, and to review someone else's frontend code.

## Self-review

- **Structure:** the feature slice is respected; one concern per file; dependencies point one way; no reach into another feature's internals.
- **Components and hooks:** components only render; state, data and handlers live in purpose-named hooks; routes only compose.
- **Effects:** no effect without an external system; each remaining effect has a one-line why.
- **Composition:** no boolean prop that switches behavior; no component with two responsibilities.
- **Boundaries:** every response, param and stored value is parsed with a schema; types are derived; no `any`, casts or suppressions.
- **Data:** screens never call the HTTP client; query keys are centralized; mutations invalidate; requests are cancellable; no waterfalls; long lists paginated and virtualized.
- **Security:** tokens in secure storage; no untrusted HTML without a sanitizer; no secrets in the bundle; external URLs checked against an allow-list.
- **States:** loading, error with retry, empty and ready on every data screen; the feature's edge cases are covered.
- **Errors:** codes are mapped to localized messages; nothing raw or unknown reaches the user.
- **Abstraction:** vendor SDKs only inside their role module; no speculative layers; the core framework is used idiomatically.
- **UI:** tokens only; each theme designed; i18n keys; accessibility (labels, roles, hit areas, focus, dynamic type, reduced motion).
- **Tests:** behavior-named; fakes only at the edges; a regression test for every fix.
- **Questions:** every stop-and-ask moment (new dependency, hand-built vs library, smell, a spec that does not hold, a platform limit) was raised, not decided silently.
- **Finish:** types, lint, format and tests pass with zero errors; no dead code, debug flags or stray TODOs.

## Reviewing someone else's code

Review in this order of priority:

1. **Correctness against the requirements:** every acceptance criterion implemented; behavior nobody asked for is flagged.
2. **Regressions:** shared components, hooks, contracts or config changed; callers affected.
3. **Security and privacy:** secrets in client code, user content or personal data in logs or analytics, unsafe HTML rendering, tokens stored insecurely.
4. **Tests:** missing or weak tests for the changed behavior; tests that cannot fail; tests that mock the project's own modules.
5. **Silent decisions:** a new dependency or undecided library introduced; something hand-built that a maintained library solves; an approved design worked around; a platform limit ignored.
6. **Craft:** walk the self-review list above.

Make each finding verifiable: file and line, what happens, a concrete scenario that triggers it, and why it matters. Rank it:

- **Blocker:** must be fixed before merging (a bug, a security issue, a broken requirement).
- **Major:** a deviation that hides or will cause a bug (unvalidated response, an effect syncing derived state, a vendor SDK leaking across the code, a raw error shown to the user).
- **Minor:** everything else.

Code that follows a different convention already consistent in that module is not a finding. Say plainly when you found nothing significant.
