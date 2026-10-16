---
name: fix-the-bug
description: "leshi workflow 09 (manual). Use ONLY when the developer explicitly asks to fix a specific Task Manager bug, e.g. 'fix bug 48', '/fix-the-bug 48'. Reproduces locally, finds the root cause, applies the smallest safe fix with regression tests and opens a PR, or reports findings without an empty PR."
license: MIT
metadata:
  leshi-skill: "09"
  leshi-contract: "1"
  leshi-gates: "behavior-change"
  leshi-shared: "conventions, task-manager, sessions, delegation, delivery, spec-revisions, craft, stack-profile"
---

# fix-the-bug

Take a Bug item from the Task Manager, **reproduce it locally**, find the **root cause**, apply the **smallest safe fix** with tests that cover the regression, validate locally, and deliver a PR linked to the bug. If the bug is already fixed, not reproducible, or has no safe justified change, record the findings in the Task Manager and **do not open an empty PR**.

Read `references/conventions.md` first. How code is written and when to stop and question: `references/craft.md`. Delivery rules are in `references/delivery.md`; the Task Manager protocol is in `references/task-manager.md`.

## 1. Trigger

- Explicit developer request in the latest message naming the bug: "fix bug 48", "fix BUG-48", "/fix-the-bug 48", "$fix-the-bug 48", or equivalent in any language.
- Not triggered by a failing test, a CI alert, an error seen while doing other work, or another skill finishing. [NO-AUTO-CHAIN] If you notice a different bug, report it as a new TM item suggestion instead of fixing it here.

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- A Task Manager is configured with read + write capabilities (`item.get`, `comment.list`, `comment.create`, `item.update`); re-validate per `references/task-manager.md` §3 if the snapshot is older than 7 days or an operation fails.
- The referenced item exists and is a Bug (label/type *Bug*). If it is an Idea or Feature, say so and suggest the right workflow (do not start it).
- The repository is a working checkout with the target branch available (`delivery.targetBranch`, default `main`).
- Check for an unfinished session for this bug (`leshi session resume`) and resume it instead of starting over.

## 3. Read

- `.leshi/task-manager.md` — this project's Task Manager playbook (where SPECs/features/subtasks live, which MCP tool does each operation). Read it before any Task Manager operation.
- The complete bug item: description, reported steps, environment/version, severity, expected vs actual behavior, attachments, linked items and **all comments**.
- `leshi.config.json`: `taskManager`, `testing` (commands, `e2e`), `delivery`, `project.platforms`.
- The approved SPEC parts that define the intended behavior (Feature item / Spec revision by reference), to distinguish a bug from a missing feature or a spec change.
- Only the code paths, logs and tests related to the failing behavior. Ask `leshi-structure` for a map when the area is unfamiliar.
- Git history of the affected files when it helps (recent changes, previous fixes).

## 4. Questions

Ask only what cannot be inferred:
- Missing reproduction details (exact steps, data, device/OS/browser, version, account type) when the ticket and comments do not have them.
- Whether expected behavior is truly expected when the SPEC is silent or ambiguous.
- Whether to proceed when reproduction needs something unavailable here (a specific device, production-only data). Never ask for secrets or production data to be pasted into chat.

## 5. Approvals

- [GATE:behavior-change] — before applying a fix that changes intended behavior, an approved SPEC, a public contract (API, schema, event) or user-visible flows beyond restoring the expected behavior. Present the root cause, the options (minimal fix vs broader change), risks and your recommendation; wait.
- A fix that only restores the behavior the SPEC already defines does not need extra approval.
- If the bug reveals a SPEC that is wrong or outdated, follow `references/spec-revisions.md` (proposal + approval); never rewrite the SPEC silently.
- Merging is never part of this workflow.

## 6. Agents

- `leshi-qa` (preferred for reproduction): reproduce the reported flow locally, with E2E where applicable, and later re-run the reproduction to confirm the fix; returns steps, evidence and pass/fail.
- `leshi-structure`: map the affected area, dependencies and boundaries when the root cause is not local.
- `leshi-frontend` / `leshi-backend`: implement the minimal fix and its regression test in their area, with a bounded brief (root cause, files allowed, contract constraints).
- `leshi-reviewer` (optional): independent review when the fix touches security, data integrity, payments, migrations or shared contracts.
- Delegate per `references/delegation.md`; small fixes are faster inline.

## 7. Scope

Allowed: read the bug and related SPECs; run the app and tests locally with practical test data; investigate code and logs; apply the minimal fix and regression tests; commit and push a fix branch; open a PR; update the bug in the TM (findings, evidence, PR link, status); write the session log.

Not allowed: opportunistic refactors, unrelated fixes, dependency upgrades not required by the fix, disabling or weakening tests/checks, touching production data or deploying, merging, changing approved behavior without [GATE:behavior-change], starting any other workflow. [NO-AUTO-CHAIN]

## 8. Steps

1. **Preconditions and session.** Check §2. Start or resume the session log (`leshi session start --workflow fix-the-bug --ref <bug-id> --harness <id>`). Set the TM status to *In Progress*.
2. **Understand the bug.** Read the complete ticket and comments. Establish expected vs actual behavior, environment, severity and the intended behavior per the approved SPEC. Write a one-paragraph problem statement.
3. **Reproduce locally.** [CHECK:reproduction] Create a fix branch from the up-to-date target branch (`delivery.branchPrefix.bug`, default `fix/` + id + slug). Run the relevant user flow in the appropriate local environment, preferably via `leshi-qa` using the project's E2E tooling (`testing.e2e`) when the bug is user-facing; otherwise a focused test or script. Document exact steps, data, device and evidence (screenshots, logs), and your hypotheses.
4. **Not reproduced?** Check whether it is already fixed on the target branch (recent commits, related PRs, versions), whether it depends on an environment you cannot reproduce, or whether the report is ambiguous. Do not guess a fix. Record the findings in the TM (versions/commits checked, environment, steps tried, evidence) and either ask the developer for missing details (*Awaiting Input*) or close the investigation per their decision. [CHECK:no-empty-pr] No PR when there is no justified change.
5. **Find the root cause.** Trace the failure to its cause, not the symptom. Distinguish bug vs missing feature vs SPEC gap. If the honest fix changes intended behavior or a contract, [GATE:behavior-change]. If there is no safe, justified change, record that analysis in the TM and stop without a PR ([CHECK:no-empty-pr]).
6. **Write the regression test first.** [CHECK:regression-test] Add a test at the lowest level that reproduces the bug (unit/integration, or E2E when the defect only shows end to end) and confirm it fails before the fix. A backend bug always gets an endpoint test against the local test database (craft §9); a unit test of the pure rule may be added next to it.
7. **Apply the smallest safe fix.** Follow existing patterns and contracts; keep the change focused; avoid breaking changes. Confirm the regression test now passes.
8. **Validate locally.** Re-run the reproduction steps (via `leshi-qa` when it reproduced the bug) and check related behavior for side effects and regressions. Run targeted tests for the touched modules.
9. **Required checks.** [CHECK:required-checks] Run every command in `testing.requiredBeforePush` (at least the configured unit tests), plus typecheck/lint when configured. All must pass. Never skip or weaken a check; disclose any that cannot run here and why.
10. **Commit, push and open the PR** against `delivery.targetBranch`: bug summary, root cause, the fix, regression test, local verification (commands + results, before/after evidence) and manual test instructions; link the TM bug. Record `<!-- leshi:pr url=... spec=<item>@<revision> -->` in the TM when a Spec revision applies. Never merge.
11. **Check CI.** Required GitHub checks must pass (GitHub MCP or `gh pr checks`). Fix real failures and repeat steps 8–10; if still running, wait or keep *In Progress* with a note.
12. **Update the Task Manager.** Save the PR URL, root cause, test instructions and evidence (`<!-- leshi:evidence run=<run-id> checks=... result=pass -->`). Set *Ready for Review* only when the PR is open, required local validations passed and required CI checks pass (see `references/delivery.md`). Otherwise keep the accurate status.
13. **Close.** Update the session log, produce the final report and suggest the developer's next step (review the PR). Do not start anything else.

## 9. Checkpoints

Write a session checkpoint after: problem statement agreed; reproduction result (reproduced or not, with evidence); root cause identified; [GATE:behavior-change] answered when applicable; regression test failing then passing; reproduction re-run passing; required checks passed; PR opened (URL); CI result; TM updated. Include the bug id and branch.

## 10. Testing

- [CHECK:reproduction]: documented reproduction (or documented non-reproduction) before any fix.
- [CHECK:regression-test]: a test that fails without the fix and passes with it.
- Reproduction re-run and related-behavior checks after the fix.
- [CHECK:required-checks]: `testing.requiredBeforePush` before the final commit/push.
- Ready for Review only with an open PR and passing required CI checks. A check that did not run is reported as not run, never as passed.

## 11. Write

- **Task Manager**: findings and hypotheses, reproduction evidence, root cause, PR link, test instructions, status (*In Progress*, *Awaiting Input/Approval*, *Blocked* or *Ready for Review* per the rules above).
- **GitHub**: fix branch commits, pushed branch, PR against the target branch.
- **Code**: the minimal fix and its regression test only.
- **Session log**: `.leshi/sessions/<run-id>.md`.
- Never: an empty PR, production data changes, deployments, merges, silent SPEC edits.

## 12. Failures

- **Missing reproduction details**: ask; status *Awaiting Input* (not *Blocked*).
- **Not reproducible / already fixed**: document what was checked and why; [CHECK:no-empty-pr]; follow the developer's decision.
- **Reproduction needs unavailable hardware/environment**: say so; offer alternatives (developer reproduces, another machine, extra logging behind a flag only with approval); never claim it was reproduced.
- **Fix requires a behavior/contract change**: [GATE:behavior-change]; status *Awaiting Approval*.
- **Required check or CI fails for reasons outside this fix**: report with evidence and ask how to proceed; do not disable the check.
- **TM write fails**: retry once; keep the update in the session log (no secrets); report it; never claim the TM was updated.
- **Blocked** only for genuine external or unavoidable impediments (third-party outage, missing access the developer cannot grant now), recorded with cause, attempts and resume condition.

## 13. Completion

Done when one of these holds:
- **Fixed**: [CHECK:reproduction] and [CHECK:regression-test] satisfied, reproduction re-run passes, [CHECK:required-checks] passed, a PR is open against the target branch and linked in the TM, and the TM status is *Ready for Review* only if required CI checks pass (otherwise the accurate status with what is missing); or
- **No change**: findings are recorded in the TM (already fixed / not reproducible / no safe justified change) and no PR was opened ([CHECK:no-empty-pr]); or
- **Paused**: *Awaiting Input/Approval* or *Blocked* with the reason recorded.

The final report lists the bug id, root cause (or why none was found), files changed, checks run with results, checks not run and why, PR URL if any, and a suggestion for the developer's next step, which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Smallest safe change: if the fix grows beyond the bug, stop and propose splitting the extra work into new TM items.
- Investigate with targeted reads and logs; do not scan the whole repository.
- Run targeted tests while iterating; run the full required suites once before the final push.
- Keep evidence concise in the TM and the PR; link logs and CI runs instead of pasting them.
- One manual run may span several conversations; resume with `references/sessions.md` instead of starting over.
