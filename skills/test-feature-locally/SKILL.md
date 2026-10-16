---
name: test-feature-locally
description: "leshi workflow 08 (manual). Use ONLY when the developer explicitly asks to test a feature locally end to end, e.g. 'test feature 123 locally', '/test-feature-locally 123'. Runs the app locally, validates journeys on the target devices, fixes findings, runs required checks, opens the PR and marks Ready for Review only when checks pass."
license: MIT
metadata:
  leshi-skill: "08"
  leshi-contract: "1"
  leshi-gates: "spec-reconciliation"
  leshi-shared: "conventions, task-manager, sessions, delegation, delivery, spec-revisions, design-tools, craft"
---

# test-feature-locally

Validate an implemented feature **locally and end to end** against its approved acceptance criteria and hi-fi designs, fix what fails, run the project's required checks, and deliver it: branch pushed, PR open against the target branch, Task Manager updated, and *Ready for Review* only when the PR exists and required checks pass.

Read `references/conventions.md` first. How code is written and when to stop and question: `references/craft.md`. Delivery rules (branches, tests, PRs, Ready for Review) are in `references/delivery.md`; the Task Manager protocol is in `references/task-manager.md`.

## 1. Trigger

- Explicit developer request in the latest message: "test feature 123 locally", "run the E2E for feature 123", "/test-feature-locally 123", "$test-feature-locally 123", or equivalent in any language.
- Not triggered by `implement-the-feature` or `enhance-the-feature` finishing, by CI failures, or by a generic "does it work?". [NO-AUTO-CHAIN] If you are unsure the developer wants to start this workflow, ask once.

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- A Task Manager is configured and its capability snapshot includes read + write (`item.get`, `item.update`, `comment.create`); re-validate per `references/task-manager.md` §3 if older than 7 days or an operation fails.
- The TM item is a Feature (label/type *Feature*) with acceptance criteria, approved designs when it has UI, and implementation present on a feature branch (subtasks implemented, or the developer confirms which part to test). If nothing is implemented, stop and say so; suggest `implement-the-feature` (do not start it).
- `testing` exists in the config (defined in `bootstrap-technology`): at least `testing.requiredBeforePush` or `testing.unitCommand`, and `testing.e2e` when the policy requires E2E for features. If the policy is missing, ask the developer how to validate and whether to record it; do not invent a policy.
- Check for an unfinished session for this item (`leshi session resume`) and resume it instead of starting over.

## 3. Read

- `.leshi/task-manager.md` — this project's Task Manager playbook (where SPECs/features/subtasks live, which MCP tool does each operation). Read it before any Task Manager operation.
- `.leshi/design.md` — this project's design playbook (which tool serves low-fi/hi-fi/UI Kit, where designs live, exact tool per operation). Read it before any design work.
- `leshi.config.json`: `taskManager`, `testing` (commands, `e2e.tool`, `e2e.command`, `e2e.devices`, `e2e.requiredFor`), `delivery` (`targetBranch`, `branchPrefix`, `worktreesDir`), `project.platforms`, `design`.
- The Feature item, all subtasks and their statuses, current HOW decisions (comments), the latest **approved** Spec revision, and the approved hi-fi references (`<!-- leshi:design kind=hi-fi ... -->`).
- The Technical Spec sections on local environment, services and test data (by reference).
- Git: current branch, diff against the target branch, existing PR for the branch if any.
- How the app runs locally: the repository's own scripts/docs (README, package scripts, Makefile, docker-compose, env examples). Read only what is needed to start it.

## 4. Questions

Ask only what cannot be inferred:
- Missing local setup details (env variables to set, seed data, accounts for test logins). Never ask for secrets to be pasted into chat; the developer sets them in their environment.
- Device coverage when the project's policy is ambiguous for this feature (visual-heavy vs primarily functional).
- Whether to proceed when a required check cannot run in this environment (for example no iOS simulator on this machine): offer options (run elsewhere, developer runs it, defer Ready for Review).

## 5. Approvals

- [GATE:spec-reconciliation] — if the approved Spec revision changed since implementation, or observed behavior conflicts with the acceptance criteria in a way that needs a product decision (the spec may be wrong, not the code), stop and present the drift per `references/spec-revisions.md`. Do not rewrite the spec or change intended behavior on your own.
- Opening the PR is part of this workflow's contract; merging is not: never merge automatically.
- Fixes that would change scope, approved designs or contracts are proposals; ask before applying them.

## 6. Agents

- `leshi-qa` (primary): run the journeys and edge cases on the chosen devices, compare UI with the hi-fi, collect evidence (screenshots, logs, steps). Several `leshi-qa` instances may cover different devices/platforms in parallel when they are independent.
- `leshi-frontend` / `leshi-backend`: fix defects in their area with focused tests; give each a bounded brief with the failing journey, evidence and files they may modify.
- `leshi-reviewer` (optional): independent review of the final diff when the change is high-risk (security, data, payments, migrations).
- Delegate per `references/delegation.md`; do small fixes inline.

## 7. Scope

Allowed: start local services, simulators/emulators and dependencies; seed local test data; run E2E and checks; fix defects found in this feature; commit and push the feature branch; open/update the PR; update the TM item (findings, evidence, status, PR link); write the session log.

Not allowed: deploying to real infrastructure or touching production data; changing approved requirements or designs; unrelated refactors; disabling or weakening required checks; merging; starting `implement-the-feature`, `enhance-the-feature` or any other workflow. Feedback loops to implementation or polish happen here as targeted fixes, never by launching those skills. [NO-AUTO-CHAIN]

## 8. Steps

1. **Preconditions and session.** Check §2. Start or resume the session log (`leshi session start --workflow test-feature-locally --ref <feature-id> --harness <id>`). Set the TM status to *In Progress*.
2. **Load the test basis.** Acceptance criteria, edge cases, approved hi-fi per device, subtasks done, current HOW decisions. Build a short test matrix: journey/criterion × device × expected result.
3. **Choose device coverage.** [CHECK:device-coverage] Visual/UI-heavy feature → every target platform (Android **and** iOS when both are targets; tablet when relevant). Primarily functional feature → one representative device if the project's policy (`testing.e2e.requiredFor`, `testing.e2e.devices`) allows it. Record the choice and the reason. Use the project's own tooling (`testing.e2e.tool` / `testing.e2e.command`, the platform simulators/emulators, browser automation, or whatever the Technical Spec chose); do not introduce a new tool without asking.
4. **Run everything locally.** Start frontend, backend and required dependencies with practical test data (local emulators/containers/mocks as the Technical Spec defines). Never point at production. If something cannot start, diagnose; if it is a genuine external impediment, record it (see §12).
5. **Test end to end.** Exercise the complete journeys, important edge cases (empty/error/loading states, permissions, offline, invalid input) and the real frontend↔backend integration. Compare observed behavior with each acceptance criterion and the UI with the approved hi-fi on each device. Capture evidence: screenshots, relevant logs, exact repro steps. No video recording is required.
6. **Log findings.** Record each defect in the TM (comment or child item per provider strategy) with steps, expected vs observed, device and evidence. Classify: implementation defect, polish defect, or spec conflict.
7. **Fix and retest (feedback loop).** Implementation and polish defects: fix with the right agent (or inline), add/adjust focused tests, then repeat only the affected journeys on the affected devices. Spec conflicts: [GATE:spec-reconciliation]. Loop until every criterion in the matrix passes or the remaining items are explicitly accepted or deferred by the developer.
8. **Required checks.** [CHECK:required-checks] Run every command in `testing.requiredBeforePush` (at least the configured unit tests), plus typecheck/lint/integration commands when configured. All must pass. Never skip or weaken a check; disclose any that cannot run here and why.
9. **Commit and push.** Small, meaningful commits referencing the TM id, following the repository's conventions; no secrets. Push the feature branch (create it per `delivery.branchPrefix` if needed).
10. **Open or update the PR** against `delivery.targetBranch` (default `main`): summary, link to the TM feature and the approved Spec revision, what changed, test matrix with results (commands + pass/fail per device), screenshots for visual changes, and manual test instructions a human can follow. Record `<!-- leshi:pr url=... spec=<feature>@<revision> -->` in the TM. Never merge.
11. **Wait for CI.** [CHECK:pr-before-ready-for-review] Check required GitHub checks (GitHub MCP or `gh pr checks`). If they fail, fix and repeat steps 7–10. If they are still running, wait or leave the item *In Progress* with a note.
12. **Update the Task Manager.** When the PR is open and required local validations and CI checks pass: save the PR URL, test instructions, final summary and evidence (`<!-- leshi:evidence run=<run-id> checks=... result=pass -->`), then set *Ready for Review*. Otherwise keep the accurate status and say exactly what is missing.
13. **Close.** Update the session log, produce the final report and suggest the next step for the developer (human review of the PR). Do not start anything else.

## 9. Checkpoints

Write a session checkpoint after: test matrix and device coverage decided; environment running (commands used); each journey/device result; each defect fixed and retested (tests that passed); required checks passed; branch pushed; PR opened (URL); CI result; TM status set. Include the TM ids, Spec revision and branch.

## 10. Testing

- E2E journeys and edge cases per the matrix on the chosen devices ([CHECK:device-coverage]).
- Focused tests for every fix; repeat only affected journeys after a fix.
- [CHECK:required-checks]: `testing.requiredBeforePush` (and configured typecheck/lint/integration) before the final commit/push.
- [CHECK:pr-before-ready-for-review]: PR exists and required CI checks pass before *Ready for Review*.
- Evidence: screenshots/logs/steps linked in the TM and the PR. A check that did not run is reported as not run, never as passed.

## 11. Write

- **Task Manager**: findings (comments/child items), evidence comment, PR link marker, final summary, test instructions, status (*In Progress* → *Ready for Review* only when §8.12 holds; *Awaiting Input/Approval* or *Blocked* when applicable).
- **GitHub**: commits on the feature branch, pushed branch, PR against the target branch (body as in step 10).
- **Code**: fixes for defects found in this feature and their tests only.
- **Session log**: `.leshi/sessions/<run-id>.md`.
- Never: production data, deployments, merges, edits to approved SPEC text.

## 12. Failures

- **App or dependency cannot start locally**: diagnose (logs, env, versions). Missing developer-provided setup (env vars, accounts) → *Awaiting Input*. Genuine external impediment (service down, license, hardware unavailable) → *Blocked* with cause, attempts and resume condition.
- **Target device unavailable** (e.g. no iOS simulator on this machine): say so; offer to run on another machine, let the developer run it, or keep the item out of *Ready for Review*. Never claim coverage you did not run.
- **Required check fails and the fix is outside this feature**: report it with evidence; ask the developer how to proceed; do not disable it.
- **CI fails or is flaky**: inspect logs, fix real failures; for flakiness, rerun once and report it; never mark *Ready for Review* on red checks.
- **TM write fails**: retry once; keep the update in the session log (no secrets); report it; never claim the TM was updated.
- **Spec conflict**: [GATE:spec-reconciliation]; status *Awaiting Approval*. That is not *Blocked*.

## 13. Completion

Done when all are true:
- every acceptance criterion in the matrix passed on the chosen devices ([CHECK:device-coverage]), or the remaining items were explicitly accepted/deferred by the developer;
- [CHECK:required-checks] passed before the final push;
- a PR is open against the target branch with summary, results, evidence and manual test instructions;
- [CHECK:pr-before-ready-for-review] holds before the TM shows *Ready for Review*; otherwise the TM shows the accurate status and the report says what is missing;
- the final report lists TM ids, PR URL, checks run with results, checks not run and why, and a suggestion for the developer's next step, which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Repeat only the journeys affected by a fix; run the full required suites once before the final push, not after every edit.
- Keep evidence concise: link files and CI runs instead of pasting long logs into the TM or the conversation.
- Stay inside this feature: other defects you notice become new TM items (Bug/Idea) for the developer to triage, not fixes in this PR.
- Do not reopen approved decisions or designs; surface conflicts through [GATE:spec-reconciliation].
- One manual run may span several conversations; resume with `references/sessions.md` instead of starting over.
