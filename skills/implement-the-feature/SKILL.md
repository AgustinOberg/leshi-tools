---
name: implement-the-feature
description: "leshi workflow 06 (manual). Use ONLY when the developer explicitly asks to implement an approved Feature, e.g. 'let's do this feature: 123', '/implement-the-feature 123'. Implements its subtasks in order with targeted tests and TM updates. Never marks Ready for Review."
license: MIT
metadata:
  leshi-skill: "06"
  leshi-contract: "1"
  leshi-gates: "unresolved-decision, spec-reconciliation"
  leshi-shared: "conventions, task-manager, sessions, delegation, spec-revisions, delivery, craft, stack-profile"
---

# implement-the-feature

Implement an approved **Feature** by working through its child subtasks: load the approved WHAT and designs, resolve the remaining technical decisions with the developer, plan, implement (in parallel where safe), and integrate with targeted validation. The run ends with the implementation complete and pending enhancement/E2E — **never** with *Ready for Review*.

Read `references/conventions.md` first. How code is written and when to stop and question: `references/craft.md`. Delegation: `references/delegation.md`. Branches, tests and PR rules: `references/delivery.md`. Resume: `references/sessions.md`. Spec changes and drift: `references/spec-revisions.md`.

## 1. Trigger

- Explicit developer request in the latest message: "let's do this feature: XXXX", "implement feature XXXX", "/implement-the-feature XXXX", "$implement-the-feature XXXX", or equivalent in any language, naming one Feature.
- Also: "continue implementing XXXX" — resume the existing run (§8 step 1).
- Not triggered by: `inspect-the-feature` finishing or a Feature appearing in the TM. [NO-AUTO-CHAIN]

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- The item is a **Feature** with approved hi-fi (approval markers present) and child subtasks. If it is still an Idea, `inspect-the-feature` is the right step: say so and stop.
- The Technical Spec is approved (`specs.technical.approvedRevision`) and the test policy exists (`testing`). Without a test policy, ask the developer for the commands to use and record them via `leshi config set testing '<json>'` only after approval.
- The TM snapshot shows read + write for items, comments and status. Re-validate if stale (`references/task-manager.md` §3).
- A clean starting point: `git status` has no unrelated changes. If it does, ask before touching anything.

## 3. Read

- `.leshi/task-manager.md` — this project's Task Manager playbook (where SPECs/features/subtasks live, which MCP tool does each operation). Read it before any Task Manager operation.
- `.leshi/design.md` — this project's design playbook (where the approved hi-fi lives and how to read/export it).
- The Feature root item (WHAT, acceptance criteria, design links), **all** subtasks (scope, acceptance criteria, dependencies, status), and HOW decision comments (current vs superseded).
- The approved hi-fi for each subtask (only the frames/pages the subtask links).
- Technical Spec sections relevant to the feature (stack, conventions, security, testing).
- The codebase: architecture, conventions and existing implementations of similar features — scoped to the modules the subtasks touch.
- `testing` and `delivery` from the config.
- An unfinished session log for this Feature, if any (resume).

## 4. Questions

Ask only when a **meaningful** technical choice is unresolved by the Technical Spec, existing decisions and the codebase (e.g. data model shape, a new dependency, an API contract between frontend and backend, a migration strategy). Present options with a recommendation and trade-offs. Never re-ask approved decisions; never ask about product requirements here (a requirements gap goes back to the Feature via §5).

## 5. Approvals

- [GATE:unresolved-decision] — each meaningful unresolved technical choice, before code depends on it. Record the decision as a TM comment `<!-- leshi:decision id=D-<n> status=current [supersedes=D-<m>] -->` with rationale on the Feature (global) or the subtask (local).
- [GATE:spec-reconciliation] — when the approved Spec revision changed since the Feature was approved or since the run started, or when code and the approved SPEC disagree (drift). Stop, describe the difference, propose a revision or a code-side decision per `references/spec-revisions.md`, and wait.
- New dependencies, schema migrations and changes to public contracts are decisions under [GATE:unresolved-decision] unless already approved.

## 6. Agents

- `leshi-structure`: map organization, contracts and the files each subtask touches; architecture impact.
- `leshi-frontend`: UI, navigation, state, frontend integration + focused tests, following the approved hi-fi.
- `leshi-backend`: APIs, domain logic, persistence, validation + focused tests, within the approved runtime/deploy choice.
- `leshi-reviewer`: independent read-only review of the integrated feature diff, **always** at the end (Phase 5), against the SPECs and `references/craft.md`.
- Choose 0–N dynamically. Frontend and backend may run in parallel **only** after the interface between them is agreed and with disjoint file ownership; use one worktree per independent writing task (`delivery.worktreesDir`). The main session integrates, resolves conflicts and re-runs affected checks. Briefs and response contract: `references/delegation.md`.

## 7. Scope

Allowed: code, tests and config changes required by the Feature's subtasks, on the feature branch (and worktrees); targeted test/lint/typecheck runs; TM updates (subtask status, decision comments, progress notes); session log.

Not allowed: changing the approved WHAT or designs; unrelated refactors or "while I'm here" fixes (record them as TM follow-ups); weakening or skipping required checks; pushing to the target branch; merging; marking the Feature *Ready for Review* [CHECK:no-premature-ready-for-review]; starting `enhance-the-feature`, `test-feature-locally` or any other workflow. [NO-AUTO-CHAIN]

## 8. Steps

The canonical checklist (0–15) grouped in the five phases of the board.

**Phase 1 — Load feature context**
0. Read the Feature, all subtasks, the PRD/SPECs, decisions and hi-fi designs (§3). Start or resume the session log (`leshi session start --workflow implement-the-feature --ref <feature-id> --harness <id>`, or `leshi session resume`). On resume: reconcile the log with `git log`/`git status` and the TM; skip subtasks already validated and recorded; continue at the first unverified one (Case F). If the approved Spec revision changed, [GATE:spec-reconciliation].
1. Inspect the codebase, architecture, conventions and existing implementations (delegate to `leshi-structure` when the area is large or unfamiliar).

**Phase 2 — Resolve technical decisions**
2. Validate dependencies and task order and identify blockers (missing APIs, unapproved decisions, external services). Ask only meaningful unresolved choices. [GATE:unresolved-decision] Record each decision as a TM comment. Real external impediments → *Blocked* with cause and resume condition; missing answers → *Awaiting Input*.

**Phase 3 — Plan implementation**
Plan frontend, backend, database and integration changes per subtask: files, contracts between layers, test approach, order. Update subtask dependencies in the TM if the plan reveals new ones. Create the feature branch from the up-to-date target branch (`references/delivery.md`). Set the Feature to *In Progress*.

**Phase 4 — Execute (in parallel where independent)**
3. Pick the next unblocked subtask (independent subtasks may go to parallel agents with disjoint file ownership).
4. Understand its acceptance criteria and the relevant files.
5. Implement following existing patterns, project conventions and `references/craft.md`; stop at its stop-and-question moments (craft §3) through [GATE:unresolved-decision].
6. Write or update the tests the subtask needs; backend changes always come with backend tests (craft §9).
7. Run targeted tests, lint and typecheck when relevant — scoped to what changed, not full suites. [CHECK:targeted-tests]
8. Fix issues and verify the subtask's acceptance criteria.
9. Update the subtask status in the TM (done, with a short evidence note: commands + result) and document relevant local decisions as subtask comments. Commit with the TM id. Checkpoint the session.
10. Repeat 3–9 until all subtasks are complete.

**Phase 5 — Integrate & check**
11. Integrate parallel work (merge worktrees, resolve conflicts) and run feature-level integration tests; verify the UI against the hi-fi where the environment allows it (detailed device E2E belongs to `test-feature-locally`).
12. Review the integrated diff with `leshi-reviewer` for every Feature: requirements, regressions, security, craft and silent decisions. Without subagent support, do the same review inline with the craft checklist (§11) and say so.
13. Fix findings and rerun the affected validations. [CHECK:targeted-tests]
14. Generate the final report: changes per subtask, test results (commands + pass/fail), checks not run and why, remaining issues, decisions taken.
15. Mark the Feature according to actual progress: *implementation complete — pending enhancement / local E2E* (or the TM equivalent, e.g. stay *In Progress* with that note). **Never** *Ready for Review*: that requires the PR and required checks produced later (`test-feature-locally`, `references/delivery.md`). [CHECK:no-premature-ready-for-review] If the project's delivery policy opens the PR at this point, follow `references/delivery.md` and still leave *Ready for Review* to the moment the PR and required checks exist and pass. Close the session log and suggest `enhance-the-feature` or `test-feature-locally` as the next manual step.

## 9. Checkpoints

Write a session checkpoint after: context loaded (Spec revision recorded); each decision approved (decision id); plan agreed; **each subtask validated** (subtask id, commit, test commands + result, files); integration done; review findings resolved; final status set. Record branch, worktrees and base commit so a resume can verify them.

## 10. Testing

- Per subtask: focused unit/component tests for the change, plus lint/typecheck when relevant ([CHECK:targeted-tests]). Use `testing.unitCommand`, `testing.lintCommand`, `testing.typecheckCommand` scoped to the touched files/modules.
- Feature level: `testing.integrationCommand` when configured.
- Do not run every required suite after each subtask; `testing.requiredBeforePush` is the final gate before push (normally in `test-feature-locally`). Never delete, skip or weaken tests to pass. Disclose checks that cannot run here.

## 11. Write

- **Code and tests** on the feature branch (worktrees for parallel tasks), small commits referencing the TM id.
- **Task Manager**: subtask statuses and evidence notes, decision comments (Feature = global, subtask = local), follow-up items for out-of-scope findings, Feature status note *implementation complete — pending enhancement / E2E*.
- **Config**: only `testing` if the developer approved new commands (`leshi config set testing '<json>'`).
- **Session log**: `.leshi/sessions/<run-id>.md`.
- Nothing pushed to the target branch; no merges; no *Ready for Review*.

## 12. Failures

- **Interrupted run**: resume per step 0; never redo validated subtasks; never trust a stale log over Git and the TM.
- **Spec changed or drift found**: [GATE:spec-reconciliation]; pause affected subtasks; continue only independent ones if the developer agrees.
- **Tests fail and the fix is unclear**: report the failure with evidence, do not disable the test; *Awaiting Input* if a decision is needed.
- **Merge conflicts between parallel tasks**: resolve in the main session, rerun affected checks; never drop a side's changes silently.
- **TM write fails**: retry once, checkpoint locally, report what was not recorded; never claim it was.
- **Blocked** only for genuine external impediments (unavailable dependency, missing credentials the developer must provide), with cause, attempts and resume condition. Missing answers are *Awaiting Input*.

## 13. Completion

Done when all are true:
- every subtask is implemented, its acceptance criteria verified and its status updated in the TM with evidence (or explicitly deferred with the developer's approval);
- all [GATE:unresolved-decision] and [GATE:spec-reconciliation] items raised in this run were answered and recorded;
- [CHECK:targeted-tests] passed per subtask and after review fixes; integration checks ran or their absence is disclosed;
- [CHECK:no-premature-ready-for-review]: the Feature is marked *implementation complete — pending enhancement / E2E* (or equivalent), not *Ready for Review*;
- the final report lists commits, branch, test results, remaining issues and the suggested next workflow, which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Context: load one subtask's files at a time; reuse `leshi-structure` findings instead of re-reading the codebase.
- Tokens/time: spawn agents only for independent work; no full suites per subtask; no repeated research.
- Conflicts: parallel writes only with disjoint ownership and agreed contracts; one worktree per independent writing task; never delete a worktree with unmerged changes.
- Scope: anything outside the Feature becomes a TM follow-up, not a change in this run.
