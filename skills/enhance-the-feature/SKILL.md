---
name: enhance-the-feature
description: "leshi workflow 07 (manual). Use ONLY when the developer explicitly asks to polish an implemented Feature, e.g. 'enhance feature 123', '/enhance-the-feature 123'. Restrained, professional UX/UI polish against the approved hi-fi and UI Kit, or a documented N/A when there is no visual surface."
license: MIT
metadata:
  leshi-skill: "07"
  leshi-contract: "1"
  leshi-gates: "approved-decision-change"
  leshi-shared: "conventions, task-manager, sessions, delegation, design-tools, delivery"
---

# enhance-the-feature

Polish an implemented **Feature** so it feels professional and native: compare it with the approved hi-fi and UI Kit and refine hierarchy, spacing, typography, states, gestures, subtle micro-interactions and accessibility. It does **not** change the product, reopen approved decisions or add decoration. If the Feature has no relevant visual surface, report N/A and finish without changes.

Read `references/conventions.md` first. Visual quality bar and design tools: `references/design-tools.md`. Delegation: `references/delegation.md`. Branch and test rules: `references/delivery.md`.

## 1. Trigger

- Explicit developer request in the latest message: "enhance feature XXXX", "polish feature XXXX", "/enhance-the-feature XXXX", "$enhance-the-feature XXXX", or equivalent in any language, naming one Feature.
- Also used after `test-feature-locally` findings when the developer explicitly asks to fix polish issues here.
- Not triggered by: `implement-the-feature` finishing. [NO-AUTO-CHAIN]

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- The item is a **Feature** whose implementation is complete (or its subtasks are done) on a feature branch. If implementation is not finished, say so and ask whether to polish what exists or stop.
- Approved hi-fi and the approved UI Kit exist (design links + approval markers). Without them there is no reference to polish against: report that and stop as *Awaiting Input*.
- The feature branch builds/runs locally, or the developer accepts static review only (say so in the report).
- The TM snapshot shows read + write for items and comments.

## 3. Read

- `.leshi/task-manager.md` — this project's Task Manager playbook (where SPECs/features/subtasks live, which MCP tool does each operation). Read it before any Task Manager operation.
- `.leshi/design.md` — this project's design playbook (which tool serves low-fi/hi-fi/UI Kit, where designs live, exact tool per operation). Read it before any design work.
- The Feature root item (WHAT, acceptance criteria, design links, approval markers) and its subtasks.
- The approved hi-fi for each affected screen and device, and the Design System Spec / UI Kit (tokens, components, patterns).
- The implementation diff on the feature branch (`git diff <target-branch>...HEAD`) and the UI files it touches.
- Findings from `test-feature-locally` (TM comments) if the developer points to them.
- `testing` (commands for affected checks) and `delivery` from the config.

## 4. Questions

Usually none. Ask only when:
- the hi-fi is ambiguous for a state or device that exists in the implementation;
- a polish improvement would touch an approved decision or scope (that is a gate, §5);
- the environment cannot run the UI and the developer must choose between static review and setting up a device/simulator.

## 5. Approvals

- [GATE:approved-decision-change] — any polish idea that would reopen an approved decision (layout, flow, copy, component choice, UI Kit token) or alter scope. Present it as a proposal with rationale; implement only after approval; otherwise record it as a follow-up suggestion in the TM. Refinements that bring the implementation **closer** to the approved hi-fi/UI Kit need no gate.

## 6. Agents

- `leshi-design`: review the running UI or screenshots against the hi-fi and UI Kit; propose concrete, restrained refinements with rationale.
- `leshi-frontend`: implement the refinements and update affected frontend tests.
- `leshi-reviewer` (optional): independent check that polish did not cause regressions or accessibility issues on high-risk screens.
- Design review and implementation are sequential per screen; independent screens may be parallelized with disjoint file ownership. Briefs per `references/delegation.md`.

## 7. Scope

Allowed: UI code and styles of the Feature's screens/components; motion/interaction details; loading/empty/error state presentation; accessibility fixes; updating affected frontend tests; TM polish subtasks and decision comments; optional screenshots as evidence; session log.

Not allowed: product changes (new behavior, removed requirements, new screens); backend/API changes; changing approved designs or UI Kit tokens without [GATE:approved-decision-change]; showy animations, decorative "AI-style" gradients/glows/emoji; unrelated refactors; marking *Ready for Review*; starting `test-feature-locally` or any other workflow. [NO-AUTO-CHAIN]

## 8. Steps

1. **Preconditions and session.** Check §2. Start the session log (`leshi session start --workflow enhance-the-feature --ref <feature-id> --harness <id>`).
2. **Applicability.** Decide whether the Feature has a relevant UX/UI surface. If not (pure backend, data migration, internal job), report **N/A** with the reason, add a TM comment, close the session and finish without changes. [CHECK:applicability]
3. **Review the experience.** Run the UI (or use screenshots) for each affected screen, device and state; compare with the approved hi-fi and UI Kit. Delegate to `leshi-design` when useful. Produce a short list of gaps, each with: screen/state, observed vs expected, severity, proposed fix.
4. **Polish the details.** Fix hierarchy, spacing, alignment, typography, color/token usage, states (loading, empty, error, disabled), affordances, legibility and consistency with the UI Kit. Delegate implementation to `leshi-frontend` when useful.
5. **Add subtle micro-interactions** only where they give purposeful feedback: pressed/hover/focus states, gentle transitions, gesture feedback, haptics where native. Native feel, accessibility (contrast, touch targets, dynamic type, screen reader labels, focus order, reduced motion) and performance (no jank, no heavy animation libraries for trivial effects). No showy animations.
6. **Changes to approved decisions.** Anything beyond the approved hi-fi/UI Kit goes through [GATE:approved-decision-change] or becomes a follow-up suggestion.
7. **Track the polish.** Record actionable polish items as TM subtasks of the Feature (or comments on it per the provider strategy) and document design decisions as comments with decision markers. Mark them resolved as they are fixed.
8. **Verify the finish.** Re-review the final experience on the affected screens/devices; rerun the affected tests, lint and typecheck. [CHECK:affected-checks] Optionally attach screenshots (before/after) as evidence. Commit with the TM id on the feature branch.
9. **Close.** Update the session log to `completed`, produce the final report and suggest `test-feature-locally <id>` as the next manual step (do not start it).

## 9. Checkpoints

Write a session checkpoint after: applicability decided (N/A or not); gap list agreed; each screen polished and its affected checks passed; any [GATE:approved-decision-change] answered; polish subtasks recorded; final verification done (commit hash).

## 10. Testing

- [CHECK:affected-checks]: rerun the frontend tests covering touched components, plus `testing.lintCommand` and `testing.typecheckCommand` when relevant. Update snapshot/visual tests only when the change is an intended, approved refinement.
- Visual verification on the devices available in this environment; disclose devices that could not be checked (full device coverage belongs to `test-feature-locally`).
- Never delete or weaken tests to make polish pass.

## 11. Write

- **Code**: UI/style/interaction changes and affected tests on the feature branch, small commits referencing the TM id.
- **Task Manager**: polish subtasks (created and resolved), decision comments, N/A note when applicable, optional screenshot evidence links.
- **Session log**: `.leshi/sessions/<run-id>.md`.
- No backend changes, no design-source changes without approval, no *Ready for Review*.

## 12. Failures

- **No visual surface**: N/A per step 2; this is a successful completion, not a failure.
- **UI cannot run here** (no simulator/device/browser): do a static review against the hi-fi, state the limitation, and leave device verification to `test-feature-locally`.
- **Hi-fi missing or ambiguous for a state**: ask; if the developer is unavailable, checkpoint as *Awaiting Input*.
- **Polish breaks tests or behavior**: revert that refinement, report it, and keep the approved behavior.
- **TM write fails**: retry once, keep the list in the session log, report what was not recorded.
- **Blocked** only for genuine external impediments, with cause and resume condition.

## 13. Completion

Done when one of these holds:
- **N/A**: [CHECK:applicability] concluded there is no relevant visual surface; recorded in the TM; no code changed; or
- **Polished**: every agreed gap is fixed or recorded as a follow-up; any [GATE:approved-decision-change] was answered; [CHECK:affected-checks] passed; polish subtasks are resolved in the TM; changes are committed on the feature branch.

In both cases the final report lists changes (or the N/A reason), checks and results, evidence, follow-ups and the suggested next workflow (`test-feature-locally`), which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Polish is discreet: fewer, well-justified changes beat many cosmetic ones. Do not reload the UI with decoration.
- Stay within the Feature's screens and the approved UI Kit; shared component changes that affect other features need [GATE:approved-decision-change].
- Load only the screens and hi-fi frames in scope; reuse `test-feature-locally` findings instead of re-testing everything.
- One manual run may span several conversations; resume with `references/sessions.md` instead of starting over.
