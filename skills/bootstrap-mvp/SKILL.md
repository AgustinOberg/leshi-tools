---
name: bootstrap-mvp
description: "leshi workflow 03 (manual). Use ONLY when the developer explicitly asks to bootstrap or define the MVP, e.g. 'bootstrap the MVP', '/bootstrap-mvp'. Defines MVP scope, designs and approves the UI Kit FIRST, then low-fi for ALL MVP features in the same run, and saves MVP + Design System Specs. No hi-fi, no code."
license: MIT
metadata:
  leshi-skill: "03"
  leshi-contract: "1"
  leshi-gates: "mvp-scope, ui-kit, low-fi-all-features, mvp-spec"
  leshi-shared: "conventions, task-manager, sessions, delegation, design-tools, spec-revisions, addons"
---

# bootstrap-mvp

Turn the approved Product and Technical Specs into an **approved MVP**: scope, journeys, dependencies and the full feature backlog; an **approved UI Kit** (designed first); **approved low-fi designs for ALL MVP features** produced in this same run; and the MVP Spec + Design System Spec saved in the Task Manager. This is the third step of the new-project path (product → technology → MVP). Hi-fi happens later, per feature, in `inspect-the-feature`.

Read `references/conventions.md` first (gates, statuses, questions policy, final report). Design rules and per-tool storage are in `references/design-tools.md`; Task Manager details in `references/task-manager.md`. Tools connected to the steps marked `[USES:<moment>]`: `references/addons.md`.

## 1. Trigger

- Explicit developer request in the latest message: "bootstrap the MVP", "let's define the MVP", "/bootstrap-mvp", "$bootstrap-mvp", or equivalent in any language.
- Not triggered by `bootstrap-technology` finishing or by another skill. [NO-AUTO-CHAIN] If you are unsure the developer wants to start this workflow, ask once.

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- Approved Product Spec and Technical Spec: `specs.product.approvedRevision` and `specs.technical.approvedRevision` are set and readable in the TM. If either is missing, stop and suggest the missing bootstrap (do not start it).
- Design tools are configured: `design.uiKit` and `design.lowFi` (and the tool is reachable, e.g. the Figma MCP server is connected). If not, ask the developer to choose or connect them now and record them with `leshi config set design '<json>'`; never substitute a tool.
- The Task Manager is writable per the capability snapshot (re-validate per `references/task-manager.md` §3 if stale).
- If `specs.mvp.approvedRevision` is already set, this is a **revision**: confirm the intent and follow `references/spec-revisions.md`.

## 3. Read

- `.leshi/task-manager.md` — this project's Task Manager playbook (where SPECs/features/subtasks live, which MCP tool does each operation). Read it before any Task Manager operation.
- `.leshi/design.md` — this project's design playbook (which tool serves low-fi/hi-fi/UI Kit, where designs live, exact tool per operation). Read it before any design work.
- `leshi.config.json`: `project` (platforms, language), `taskManager`, `design`, `specs`.
- The approved Product Spec (personas, core capabilities, design references, desired visual language) and Technical Spec (platforms, frameworks — they constrain native patterns and component availability).
- Existing Idea items in the TM that may belong to the MVP.
- Existing design artifacts at `design.*.ref` (to iterate instead of recreating).

## 4. Questions

Ask only what cannot be inferred, in small batches with recommended defaults:
1. **MVP goal**: the smallest product that proves the value proposition; launch audience; what is explicitly out of the MVP.
2. **Journeys**: the primary user journeys end to end (onboarding → core loop → retention moments), and which platforms/form factors each must work on.
3. **Prioritization**: must-have vs nice-to-have per feature; dependencies and ordering constraints.
4. **Visual direction for the UI Kit**: confirm the references and visual language from the Product Spec; brand assets that exist (logo, colors, fonts); accessibility requirements; light/dark mode.

## 5. Approvals

- [GATE:mvp-scope] — the MVP goal, journeys, dependencies and the **complete** feature list with priorities, before any design.
- [GATE:ui-kit] — the UI Kit (tokens, core components and states, shared patterns), before any feature low-fi.
- [GATE:low-fi-all-features] — the low-fi designs of **every** MVP feature, reviewed together so navigation and flows are coherent across the whole MVP.
- [GATE:mvp-spec] — the MVP Spec and Design System Spec texts before they are saved as approved revisions.
- Feedback at any gate is iteration within this run; the developer never has to restart the workflow.

## 6. Agents

- `leshi-design` (primary): UI Kit, overall UX map and navigation, low-fi per feature, interaction rationale, using the configured tool.
- `leshi-research` (optional): how comparable products solve a specific journey or pattern; references for edge cases.
- Low-fi for independent features may be split across parallel `leshi-design` instances **only after** [GATE:ui-kit] and with the navigation map agreed, each owning distinct files/pages; the main session checks consistency. Delegate per `references/delegation.md`.

## 7. Scope

Allowed: scoping conversation, design work in the configured tools (UI Kit and low-fi), committing HTML design files under `design/` when the tool is HTML, writing the MVP Spec, Design System Spec and one Idea item per MVP feature to the TM, recording `specs.mvp` and `specs.designSystem` with `leshi config set`, the session log.

Not allowed: hi-fi designs (they come per feature in `inspect-the-feature`), application code, changing approved Product/Technical decisions (propose a revision instead), switching design tools, starting any other workflow. [NO-AUTO-CHAIN]

## 8. Steps

1. **Preconditions and session.** Check §2. Start the session log (`leshi session start --workflow bootstrap-mvp --ref <product-spec-id> --harness <id>`).
2. **Define the MVP scope.** Ask §4.1–4.3. Produce: MVP goal, non-goals, primary journeys, the complete feature list (name, one-line intent, priority, platforms, dependencies), ordering and risks. [GATE:mvp-scope] Iterate on feedback in place.
3. **Design the UI Kit — first task.** [USES:design-references] [USES:ui-kit] With the configured `design.uiKit` tool (`references/design-tools.md`, including its **Design craft**). If the visual direction is not fixed yet, first show 3–4 genuinely different directions side by side with a recommendation and let the developer choose (record what was rejected and why); then build the kit on the chosen one: tokens (color, type scale, spacing, radius, elevation, motion), core components with all states (buttons, inputs, lists, cards, navigation, sheets/dialogs, feedback, empty/loading/error), shared UX patterns and accessibility rules, and each theme chosen in §4 (e.g. light and dark), each designed rather than inverted. Native to the target platforms; restrained, no decorative "AI style". Present it for review (links, or offer to open the HTML pages). [GATE:ui-kit] Iterate on feedback in place.
4. **Map the overall UX.** Navigation structure, screen inventory and flows across the entire MVP (a single map), so feature low-fi stays coherent. Confirm it with the developer as part of the low-fi review.
5. **Low-fi for ALL MVP features — in this same run.** [USES:screens] For every feature from step 2, create low-fi with the configured `design.lowFi` tool: structure, hierarchy, flows, all relevant states and edge cases, using UI Kit components (no visual polish). Do not stop after a subset; if the run must pause, checkpoint which features remain and continue on resume. [CHECK:low-fi-coverage] — before the review, verify that every feature in the approved scope has a low-fi artifact (list feature → artifact link); list gaps and finish them.
6. **Review the low-fi together.** Present all low-fi with the UX map. [GATE:low-fi-all-features] Iterate on feedback in place; re-run [CHECK:low-fi-coverage] after changes.
7. **Write the specs.**
   - **Design System Spec**: approved UI Kit (link + version), tokens, components and states, interaction principles, accessibility rules, references.
   - **MVP Spec**: goal, non-goals, journeys, prioritized feature backlog with dependencies, UX map, and the approved low-fi link + version for **every** feature; a note that hi-fi is completed per feature later.
   Present both. [GATE:mvp-spec] Iterate on feedback in place.
8. **Save to the Task Manager.** Per `references/task-manager.md` §5: Design System Spec and MVP Spec items with `<!-- leshi:spec kind=design-system|mvp revision=v1 status=approved -->`, snapshot comments, approval comments for every gate (with `<!-- leshi:design ... -->` markers for the UI Kit and each low-fi). Create one **Idea** item per MVP feature (label/type Idea, links to MVP Spec and its approved low-fi, priority and dependencies); reuse existing Idea items instead of duplicating them.
9. **Record references.** `leshi config set specs.designSystem '{"ref":"<item-id>","approvedRevision":"v1"}'`, `leshi config set specs.mvp '{"ref":"<item-id>","approvedRevision":"v1"}'`, and `design.*.ref` if the artifacts' location changed; `leshi config validate`. Re-read the TM items to confirm they were saved.
10. **Close.** Update the session log to `completed`, produce the final report and suggest `inspect-the-feature` for the first chosen feature as the next manual step.

## 9. Checkpoints

Write a session checkpoint after: [GATE:mvp-scope] approved (feature list); [GATE:ui-kit] approved (artifact version); each feature's low-fi created (so a resumed run knows what remains); [CHECK:low-fi-coverage] passed; [GATE:low-fi-all-features] approved; [GATE:mvp-spec] approved; specs and Idea items saved and read back. Include TM item ids, revisions and artifact links.

## 10. Testing

No code tests. Verifications: [CHECK:low-fi-coverage]; design artifacts open/render (HTML pages load; Figma/Penpot links resolve); `leshi config validate` after config writes; a read-back of the saved spec items, approval comments and Idea items.

## 11. Write

- **Design tool**: UI Kit and low-fi artifacts (HTML files committed under `design/` on the working branch when the tool is HTML).
- **Task Manager**: Design System Spec and MVP Spec items (approved v1, snapshots, approvals, design markers); one Idea item per MVP feature.
- **Config** (`leshi config set`): `specs.designSystem`, `specs.mvp`, `design.*.ref` if needed.
- **Session log**: `.leshi/sessions/<run-id>.md`.
- Nothing else: no hi-fi, no application code.

## 12. Failures

- **Missing approved Product/Technical Spec**: stop; suggest the missing bootstrap (do not start it).
- **Design tool unreachable or read-only** (e.g. Figma MCP not connected): explain the limitation; ask the developer to connect it or choose an authorized alternative; record the decision; never switch silently. Status *Awaiting Input* meanwhile.
- **Too many features for one sitting**: keep going across resumes; checkpoint per feature; never approve a partial low-fi set as complete.
- **Scope change discovered while designing** (a feature is missing or must split): return to [GATE:mvp-scope] with the proposed change; do not add features silently.
- **TM write fails**: re-run discovery; status *Awaiting Input*; never claim items were saved.
- **Developer unavailable mid-gate**: checkpoint and stop as *Awaiting Approval* (not *Blocked*).
- **Blocked** only for genuine external impediments (e.g. brand assets owned by a third party not yet delivered), with cause and resume condition.

## 13. Completion

Done when all are true:
- [GATE:mvp-scope], [GATE:ui-kit], [GATE:low-fi-all-features] and [GATE:mvp-spec] were approved by the developer, in that order (verifiable in the TM);
- [CHECK:low-fi-coverage] passed: every MVP feature has an approved low-fi linked from the MVP Spec and its Idea item;
- the Design System Spec and MVP Spec approved revisions and the Idea items exist in the TM and were read back;
- `specs.designSystem` and `specs.mvp` are in `leshi.config.json` and `leshi config validate` passes;
- no hi-fi or code was produced;
- the final report lists TM ids, artifact links, config keys written and the suggested next workflow (`inspect-the-feature` for a chosen feature), which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Low-fi is about structure and flow; do not spend effort on visual polish there.
- One UI Kit for the whole project; features reuse it. New shared components discovered later go through a Design System Spec revision.
- Keep spec text in the TM and designs in the design tool; only ids, links and short status go in the session log.
- Do not re-ask decisions already recorded in the config or approved specs unless the developer wants to change them.
- One manual run may span several conversations; resume with `references/sessions.md` instead of starting over.
