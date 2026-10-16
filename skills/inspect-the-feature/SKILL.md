---
name: inspect-the-feature
description: "leshi workflow 05 (manual). Use ONLY when the developer explicitly asks to turn an Idea into a Feature, e.g. 'convert from idea to feature 123', '/inspect-the-feature 123'. Clarifies the Idea, reuses or creates low-fi, produces approved hi-fi and executable subtasks. Never writes application code."
license: MIT
metadata:
  leshi-skill: "05"
  leshi-contract: "1"
  leshi-gates: "open-questions, low-fi, hi-fi"
  leshi-shared: "conventions, task-manager, sessions, delegation, design-tools, spec-revisions, craft, stack-profile, addons"
---

# inspect-the-feature

Turn a Task Manager item labeled **Idea** into an approved **Feature**: complete functional WHAT (intent, requirements, acceptance criteria, states), approved hi-fi for the relevant devices, and executable child subtasks. Technical HOW decisions known so far go in comments. This workflow **does not implement** anything.

Read `references/conventions.md` first (gates, statuses, questions policy, final report). TM model and markers: `references/task-manager.md`. Design rules and tool layouts: `references/design-tools.md`. Revisions: `references/spec-revisions.md`. How the subtasks will be built (to slice them): `references/craft.md`. Tools connected to the steps marked `[USES:<moment>]`: `references/addons.md`.

## 1. Trigger

- Explicit developer request in the latest message: "convert from idea to feature XXXX", "inspect feature idea XXXX", "/inspect-the-feature XXXX", "$inspect-the-feature XXXX", or equivalent in any language. The request names (or clearly points to) one Idea item.
- Not triggered by: finishing `bootstrap-mvp`, a new Idea appearing in the TM, or any other skill finishing. [NO-AUTO-CHAIN] If the item id is missing or ambiguous, ask once.

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- `taskManager` is configured and its capability snapshot shows read + write (`item.get`, `item.update`, `comment.create`, ideally `item.children.create`). If the snapshot is older than 7 days or an operation fails, re-validate per `references/task-manager.md` §3.
- The item exists and is an **Idea** (label/type). If it is already a Feature, this is a **revision**: confirm intent and follow `references/spec-revisions.md`.
- Global SPECs exist and are approved: Product, Technical, Design System (UI Kit), MVP (`specs.*` in the config). If the UI Kit is not approved, hi-fi cannot follow it: tell the developer `bootstrap-mvp` is the right step and stop as *Awaiting Input*.
- `design.lowFi` and `design.highFi` tools are configured and reachable (e.g. the Figma MCP server is connected). Never substitute a tool.

## 3. Read

- `.leshi/task-manager.md` — this project's Task Manager playbook (where SPECs/features/subtasks live, which MCP tool does each operation). Read it before any Task Manager operation.
- `.leshi/design.md` — this project's design playbook (which tool serves low-fi/hi-fi/UI Kit, where designs live, exact tool per operation). Read it before any design work.
- The Idea item: body, all comments, attachments/links, labels, linked items.
- Relevant SPECs by reference: Product Spec (users, value), Technical Spec (constraints that change UX, e.g. offline, platform limits), Design System Spec (UI Kit, patterns), MVP Spec (journeys, dependencies, and the **approved low-fi** for this feature if it was part of the MVP).
- Existing hi-fi files related to the feature and neighbouring screens it must stay consistent with.
- Related features/subtasks (dependencies, overlaps) — titles and status, not their full history.
- Code only to understand existing behavior the feature touches (read-only, minimal).

## 4. Questions

Ask only what cannot be inferred, batched, with recommended defaults:
1. **Edge cases**: empty data, errors, offline, permissions, limits, concurrency, first-run vs returning user, accessibility needs.
2. **Undefined behavior**: anything the Idea leaves open that changes requirements or acceptance criteria.
3. **References** (if they apply): products or flows the developer wants this to resemble or avoid.
4. **Devices**: which form factors need hi-fi (phone + tablet by default when the product targets them; desktop/web breakpoints if relevant).
Questions that only affect implementation details belong to `implement-the-feature`, not here.

## 5. Approvals

- [GATE:open-questions] — answers to non-inferable requirements questions, and the proposed flow, before designing.
- [GATE:low-fi] — only when the low-fi is **new or materially modified** (no approved current MVP low-fi, or the feature changed since it was approved). Reusing the approved, current low-fi needs no new approval.
- [GATE:hi-fi] — the final hi-fi for every relevant device, before the item is marked Feature.
- Any change to an approved global SPEC uncovered here (e.g. a new pattern for the UI Kit) is a proposal per `references/spec-revisions.md`, approved separately.

## 6. Agents

- `leshi-design` (usual): flow proposal, low-fi changes, hi-fi phone + tablet using the UI Kit, design rationale.
- `leshi-research` (optional): how comparable products handle this flow, references for edge cases.
- `leshi-structure` (optional): existing code/architecture constraints that affect the feature or how to slice subtasks.
- Research and structure analysis are independent and may run in parallel; design waits for the answered questions. Delegate per `references/delegation.md`; do small work inline.

## 7. Scope

Allowed: conversation; reading the TM, SPECs, designs and code; creating/updating design artifacts with the configured tool; updating the Idea/Feature item, its comments and child subtasks in the TM; committing design files (HTML provider) on a design branch when the project uses repo-stored designs; writing the session log.

Not allowed: application code changes, tests, migrations or dependency changes [CHECK:no-implementation]; changing approved global SPECs without the revision protocol; reopening approved UI Kit decisions; starting `implement-the-feature` or any other workflow. [NO-AUTO-CHAIN]

## 8. Steps

1. **Preconditions and session.** Check §2. Start the session log (`leshi session start --workflow inspect-the-feature --ref <idea-id> --harness <id>`). Set the item to *In Progress*.
2. **Understand the request.** Read §3. Summarize the intent, the users affected, and how it fits the Product and MVP SPECs in 4–6 bullets. Inspect every hi-fi file related to the feature.
3. **Ask.** [USES:design-references] Edge cases, undefined things, references and devices (§4). Investigate references when given (delegate to `leshi-research` if broad). Set *Awaiting Input* while waiting. [GATE:open-questions]
4. **Suggest a flow.** Entry points, steps, decision points, exits, and every state (default, loading, empty, error, permission denied, offline where relevant). Confirm it with the developer as part of [GATE:open-questions].
5. **Low-fi.** [USES:screens] If the MVP has an approved, current low-fi for this feature and the flow did not change materially, **reuse it** and say so (link + version). Otherwise create or modify it with `design.lowFi` (`references/design-tools.md`) and get approval. [GATE:low-fi]
6. **Hi-fi.** [USES:screens] Produce hi-fi with `design.highFi`, using only UI Kit tokens/components, for each relevant device (phone + tablet when applicable), covering all states from step 4. Delegate to `leshi-design` when useful.
7. **Ping-pong.** Present the hi-fi, collect feedback, iterate in the same run (feedback is not failure). Set *Awaiting Approval* while waiting. [GATE:hi-fi]
8. **Write the Feature WHAT.** Update the root item **preserving the original request** (keep it quoted under "Original request"), then: intent, requirements, acceptance criteria (testable, numbered), states, non-goals, dependencies, approved design links with `<!-- leshi:design kind=hi-fi tool=<provider> ref=<link> version=<rev> -->` (and low-fi), approval markers for each gate.
9. **Record the HOW known so far.** Technical decisions that already constrain the feature (from the Technical Spec or answers) as comments with `<!-- leshi:decision id=D-<n> status=current -->` and rationale. Unresolved technical choices are listed as open, not decided (they belong to `implement-the-feature`).
10. **Divide into subtasks.** Create child items (sub-issues or the provider equivalent) labeled **Implementation**, small and independently verifiable: scope, acceptance criteria, dependencies/order, definition of done, and the link to the exact hi-fi file/frame for that task (for the HTML provider, the `.html` page). Children inherit global decisions from the parent; do not duplicate them. If the TM lacks child items, use the developer-approved fallback (`references/task-manager.md` §2).
11. **Promote to Feature.** Change the label/type **Idea → Feature** and set the status to *Feature* (ready to plan/implement). Read the item and its children back from the TM to verify. [CHECK:no-implementation] — confirm with `git status`/diff that no application code changed in this run.
12. **Close.** Update the session log to `completed`, produce the final report and suggest `implement-the-feature <id>` as the next manual step (do not start it).

## 9. Checkpoints

Write a session checkpoint after: questions answered ([GATE:open-questions]); low-fi reused or approved ([GATE:low-fi]) with link/version; hi-fi approved ([GATE:hi-fi]) with links/versions per device; Feature WHAT saved and read back; subtasks created (ids); label switched to Feature.

## 10. Testing

No code tests. Verifications: read-back of the Feature item, its approval comments and every child subtask; design links resolve (file exists in the repo at the recorded commit, or the frame/page link opens); [CHECK:no-implementation] via `git status` showing only design/doc changes, if any.

## 11. Write

- **Task Manager**: root item updated (original request preserved, WHAT, acceptance criteria, design links, approval markers), HOW decision comments, child **Implementation** subtasks with design links, label Idea → Feature, status *Feature*.
- **Design artifacts**: low-fi (only if new/changed) and hi-fi per device in the configured tool; for HTML, files under `design/low-fi/` and `design/hi-fi/<feature-slug>/` committed on a branch and linked.
- **Config**: nothing, unless a design `ref` is recorded for the first time (`leshi config set design.highFi '<json>'`).
- **Session log**: `.leshi/sessions/<run-id>.md`.
- No application code, tests or dependency changes.

## 12. Failures

- **Item is not an Idea / not found**: report what was found and ask; do not guess another item.
- **TM read-only or missing operations**: explain what cannot be saved, offer authorized options, status *Awaiting Input*; never claim the Feature was saved.
- **No child-item support**: propose the checklist fallback and wait for approval.
- **Design tool unreachable** (e.g. MCP not connected): guide the developer to connect it; *Awaiting Input*; never switch tools.
- **Feature contradicts an approved SPEC**: stop, describe the conflict, propose a revision per `references/spec-revisions.md` ([GATE:open-questions] covers the decision); do not resolve silently.
- **Developer unavailable at a gate**: checkpoint and stop as *Awaiting Input/Approval*. That is not *Blocked*.
- **Blocked** only for genuine external impediments (e.g. design tool account suspended), with cause and resume condition.

## 13. Completion

Done when all are true:
- [GATE:open-questions] and [GATE:hi-fi] were approved by the developer; [GATE:low-fi] was approved, or the approved current MVP low-fi was reused (and that reuse is recorded);
- the root item holds the complete WHAT with the original request preserved, design links and approval markers, and is labeled **Feature**;
- executable child subtasks exist (or the approved fallback) with acceptance criteria, dependencies and hi-fi links, and were read back;
- [CHECK:no-implementation] passed: no application code changed;
- the final report lists TM ids, design links/versions, open technical questions and the suggested next workflow (`implement-the-feature <id>`), which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Do not redesign approved low-fi by routine; reuse it unless the feature changed materially.
- Keep hi-fi within the UI Kit; new shared patterns are UI Kit revision proposals, not silent additions.
- Subtasks are sized for one focused implementation pass each; avoid both mega-tasks and dozens of trivial ones.
- Keep SPEC and design content in the TM and design tool, not in the session log (ids, links and status only).
- One manual run may span several conversations; resume with `references/sessions.md` instead of starting over.
