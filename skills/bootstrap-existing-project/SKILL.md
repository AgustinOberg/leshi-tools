---
name: bootstrap-existing-project
description: "leshi workflow 04 (manual). Use ONLY when the developer explicitly asks to bootstrap or import an EXISTING project, e.g. 'bootstrap existing project', '/bootstrap-existing-project'. Inspects the repo read-only, reconstructs baseline Product/Technical/Design System specs and a feature inventory, and imports them to the Task Manager after approval. Never refactors."
license: MIT
metadata:
  leshi-skill: "04"
  leshi-contract: "1"
  leshi-gates: "baseline-review, task-manager-import"
  leshi-shared: "conventions, task-manager, sessions, delegation, design-tools, spec-revisions, craft, stack-profile, addons"
---

# bootstrap-existing-project

Bring an existing app/repository into the leshi factory **without changing its code**: discover what exists, reconstruct a baseline Product Spec, Technical Spec and Design System/UI Kit state plus a feature inventory, review gaps and drift with the developer, then import the approved baseline into the Task Manager. This is the alternative entry point to the new-project path; afterwards new ideas use the same workflows as any project.

GitHub shows what exists today; the Task Manager records the reviewed, approved definition of what the product is and should become.

Read `references/conventions.md` first (gates, statuses, questions policy, final report). Task Manager details are in `references/task-manager.md`, design tools in `references/design-tools.md`. The yardstick for how far the code is from leshi's way of writing it: `references/craft.md`. Tools connected to the steps marked `[USES:<moment>]`: `references/addons.md`.

## 1. Trigger

- Explicit developer request in the latest message: "bootstrap existing project", "import this repo into leshi", "/bootstrap-existing-project", "$bootstrap-existing-project", or equivalent in any language.
- Not triggered by `leshi init` finishing, by opening an existing repo, or by another skill. [NO-AUTO-CHAIN] If you are unsure the developer wants to start this workflow, ask once.

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- The repository has an existing product (source code, docs or a running app). If it is essentially empty, suggest `bootstrap-product` instead (do not start it).
- The working tree state is known: run `git status` at the start and record it; pre-existing uncommitted changes belong to the developer and are left untouched.
- If `specs.product` / `specs.technical` / `specs.designSystem` are already set, this is a **re-baseline**: confirm the intent and follow `references/spec-revisions.md`.
- A Task Manager is not required to *start*; choosing and connecting it is part of this workflow when not configured yet.

## 3. Read

- `.leshi/task-manager.md` and `.leshi/design.md` — this project's Task Manager and design playbooks, if they already exist (this workflow creates them in the project-setup step).
- `leshi.config.json`: `project`, `harnesses`, `taskManager`, `design`, `specs`, `testing`.
- The repository, read-only: README and docs, package manifests and lock files, app entry points, routes/screens/navigation, API endpoints, data models and migrations, configuration and environment templates (never secret values), CI/CD files, tests and their commands, issue/PR templates, changelogs.
- Observable behavior when it can be run locally without side effects (build or start commands from the repo's own docs); never against production data.
- Existing design artifacts and links (Figma/Penpot/HTML, screenshots in docs).
- Existing tracker content (issues, TODO/FIXME notes) for known bugs and pending work.

## 4. Questions

Ask only what cannot be observed, in small batches with recommended defaults:
1. **Intent**: what the product is for, who uses it today, what is in production vs in progress, the desired direction (keep, extend, pivot).
2. **Unknowns found during discovery**: undocumented behavior, conflicting docs vs code, features that look abandoned, ownership of external services.
3. **Project setup** (one combined step, only if not configured): Task Manager (Linear, Plane, GitHub Projects/Issues, Notion, Jira, or **Other (free text)**), low-fi/hi-fi/UI Kit tools, repository organization, `project.language`.
4. **Test policy**: confirm the existing test commands found in the repo (and what must pass before push) so later workflows can use them; deeper changes to the stack or tests belong to a later `bootstrap-technology` revision.

## 5. Approvals

- [GATE:baseline-review] — the reconstructed Product Spec, Technical Spec, Design System/UI Kit baseline, feature inventory and the gaps/risks/drift report, before anything is imported. Observed facts and assumptions must be clearly separated.
- [GATE:task-manager-import] — the exact list of Task Manager items to create (specs, features by status, bugs, improvements, pending work) and their structure, before creating them.
- Any tool choice or TM capability gap follows the same rules as `bootstrap-product` (the developer decides; never substitute).

## 6. Agents

- `leshi-structure` (recommended): architecture, module boundaries, conventions, dependencies, build/test setup, technical risks.
- `leshi-frontend` / `leshi-backend`: screens, navigation and state; APIs, data models, integrations; how each feature is actually implemented and its observable behavior.
- `leshi-design` (when there is a UI): current visual language, implicit UI Kit (tokens/components in code), UX inconsistencies.
- `leshi-qa` (optional): run the app or tests locally to confirm observable behavior and known bugs, without changing code.
- Discovery areas are independent and may run in parallel; every agent is **read-only for source code** in this workflow. Delegate per `references/delegation.md`.

## 7. Scope

Allowed: read-only inspection; running the project's own build/test/start commands locally when safe; writing the baseline specs and inventory to the Task Manager after approval; recording `taskManager`, `design`, `testing` and `specs.*` with `leshi config set`; the session log.

Not allowed: refactoring, reformatting, upgrading dependencies, fixing bugs, redesigning UI, changing tests, creating branches or PRs, migrating data, running against production, starting any other workflow. Findings become TM items; fixing them is for later, manually triggered workflows. [NO-AUTO-CHAIN]

## 8. Steps

1. **Preconditions and session.** Check §2; record `git status`. Start the session log (`leshi session start --workflow bootstrap-existing-project --harness <id>`).
2. **Discover what already exists.** Delegate discovery areas in parallel when useful. Identify the current product, platforms, features, user journeys, visual language, technical foundation (stack, runtime, hosting/deploy, data, auth, CI/CD), tests and how to run them, documentation, known issues and production state.
3. **Reconstruct the project foundation.** [USES:ui-kit] Draft from evidence: Product Spec baseline (purpose, users, capabilities, platforms), Technical Spec baseline (architecture, stack, runtime/deploy, data, security, CI/CD, test commands, conventions, and the code-level decisions of craft §2 as observed — contract sync between apps, frontend test policy, whether a backend test setup with a local test database exists — or listed as open), Design System/UI Kit baseline (tokens and components as implemented, references), and the **feature inventory** (feature → status: implemented / partial / in progress / abandoned; evidence: files, routes, screens).
4. **Identify gaps and differences.** Unknown intent, missing decisions, undocumented behavior, incomplete work, docs vs implementation differences, risks (security, outdated dependencies, missing tests, no backend test setup), divergences from the craft (as facts, never fixed here), known bugs and improvements. Label every statement **Observed** (with file/line or command evidence) or **Assumed**.
5. **Review with the developer.** Ask §4.1–4.2; confirm the desired direction and clarify important unknowns. Present the baseline and the gaps report. [GATE:baseline-review] Iterate on feedback in place.
6. **Project setup and Task Manager connection** (skip what is already configured). Ask §4.3–4.4. Record choices with `leshi config set taskManager '<json>'`, `leshi config set design '<json>'`, `leshi config set testing '<json>'` (only the commands that exist in the repo). Connect the TM and discover capabilities per `references/task-manager.md` §3 [CHECK:task-manager-capabilities]: list tools, exercise only read calls, show the verified / advertised / unavailable table, record it with `leshi tm record ...`. Then create and complete the playbook (`leshi tm playbook` → `.leshi/task-manager.md`: where each concept lives and the exact tool per operation) and include it in the approval of the setup. Do the same for the design tools (`references/design-tools.md`): discover and `leshi design record`, then `leshi design playbook` → `.leshi/design.md` describing where the existing UI/design artifacts actually live. Propose add-ons the same way (`references/addons.md` §4): tools the harness already has, the moments they serve, connected with `leshi addon add` once approved. If the TM is read-only or missing required operations, explain, offer authorized options and stop as *Awaiting Input*; never simulate persistence.
7. **Plan the import.** List the items to create: Product, Technical and Design System Spec items (approved v1 baseline), one item per inventoried feature (implemented features as documented Features; partial/in-progress as Features or Ideas per the developer), Bug items for known bugs, Idea/improvement items for pending work, with labels/status per the provider strategy. [GATE:task-manager-import]
8. **Import into the Task Manager.** Create the items per `references/task-manager.md` §5 with spec markers (`<!-- leshi:spec kind=... revision=v1 status=approved -->`), snapshot and approval comments, and links to the repository and available design artifacts. Reuse existing items instead of duplicating them.
9. **Record references.** `leshi config set specs.product '{"ref":"<id>","approvedRevision":"v1"}'` and likewise `specs.technical`, `specs.designSystem`; `leshi config validate`. Re-read the TM items to confirm they were saved.
10. **Verify no code changes.** [CHECK:no-code-changes] Run `git status` and compare with step 1: the only changes made by this run may be `leshi.config.json`, `leshi.lock.json`, files regenerated by `leshi sync`, and `.leshi/` session files. If anything else changed, report it and revert only what this run changed (never the developer's pre-existing changes).
11. **Close.** Update the session log to `completed`, produce the final report and suggest the next manual step (usually `inspect-the-feature` for a new idea or `fix-the-bug` for a known bug), which you do not start.

## 9. Checkpoints

Write a session checkpoint after: initial `git status` recorded; each discovery area delivered; baseline and gaps drafted; [GATE:baseline-review] approved; tools and TM capabilities recorded; [GATE:task-manager-import] approved; each batch of TM items created (ids, so a resumed run does not duplicate them); [CHECK:no-code-changes] passed.

## 10. Testing

Run only the project's existing build/test commands, locally and without side effects, to observe the current state (record pass/fail as facts; failing tests become findings, not fixes). Verifications: [CHECK:task-manager-capabilities]; [CHECK:no-code-changes]; `leshi config validate` after config writes; a read-back of the imported items.

## 11. Write

- **Task Manager**: Product, Technical and Design System baseline spec items (approved v1, snapshots, approvals); feature inventory items; Bug and Idea/improvement items; links to the repository and design artifacts.
- **Config** (`leshi config set`): `taskManager` (with capability snapshot), `design`, `testing` (existing commands), `project.*` metadata, `specs.product|technical|designSystem`.
- **Session log**: `.leshi/sessions/<run-id>.md`.
- Nothing else: no source code, tests, dependencies, branches or PRs.

## 12. Failures

- **Cannot build or run the project locally**: record it as an observed fact (with the error), rely on static reading, mark behavior claims as assumptions. Do not "fix" the build.
- **TM connection fails or is read-only**: see step 6; status *Awaiting Input*; never claim items were created.
- **Import interrupted midway**: resume from the checkpointed list of created ids; never duplicate items.
- **Docs and code disagree**: record the drift; let the developer decide which reflects the intended product.
- **Secrets found in the repo**: do not copy them anywhere; tell the developer the file and line so they can rotate and remove them.
- **Developer unavailable mid-gate**: checkpoint and stop as *Awaiting Approval* (not *Blocked*).
- **Blocked** only for genuine external impediments (e.g. no access to a required private dependency), with cause and resume condition.

## 13. Completion

Done when all are true:
- [GATE:baseline-review] and [GATE:task-manager-import] were approved by the developer (verifiable in the TM);
- the baseline spec items and the inventory exist in the TM and were read back;
- `taskManager` (with capability snapshot) and `specs.*` are in `leshi.config.json` and `leshi config validate` passes;
- [CHECK:no-code-changes] passed: no source changes, no branch, no PR;
- the final report lists TM ids, the gaps/risks summary, config keys written and the suggested next workflow, which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Breadth over depth: map every feature at inventory level; do not deep-dive into code unrelated to understanding behavior.
- Do not assume the factory's default stacks; describe the project as it is.
- Keep spec text in the TM; only ids, links and short status go in the session log.
- Do not re-ask decisions already recorded in the config unless the developer wants to change them.
- One manual run may span several conversations; resume with `references/sessions.md` instead of starting over.
