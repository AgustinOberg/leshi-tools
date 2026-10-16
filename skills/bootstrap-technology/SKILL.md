---
name: bootstrap-technology
description: "leshi workflow 02 (manual). Use ONLY when the developer explicitly asks to bootstrap or define the technology of a product, e.g. 'bootstrap the technology', '/bootstrap-technology'. Reviews architecture, stack, runtime, deployment, data, security and test policy with the developer and produces an approved Technical Spec. The developer decides."
license: MIT
metadata:
  leshi-skill: "02"
  leshi-contract: "1"
  leshi-gates: "deployment-choice, test-policy, technical-spec"
  leshi-shared: "conventions, task-manager, sessions, delegation, spec-revisions, delivery, craft, stack-profile"
---

# bootstrap-technology

Turn the approved Product Spec into an **approved Technical Spec** and a **test policy** recorded in the project configuration. Agents research, compare and recommend; **the developer decides** every material technical choice. This is the second step of the new-project path (product → technology → MVP).

Read `references/conventions.md` first (gates, statuses, questions policy, final report). Task Manager details are in `references/task-manager.md`, revisions in `references/spec-revisions.md`, test and delivery rules in `references/delivery.md`. Code-level decisions come from `references/craft.md` §2; the recommended defaults are in `references/stack-profile.md`.

## 1. Trigger

- Explicit developer request in the latest message: "bootstrap the technology", "let's define the stack", "/bootstrap-technology", "$bootstrap-technology", or equivalent in any language.
- Not triggered by `bootstrap-product` finishing, by a passing mention of a framework, or by another skill. [NO-AUTO-CHAIN] If you are unsure the developer wants to start this workflow, ask once.

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- An approved Product Spec exists: `specs.product.approvedRevision` is set and the item is readable in the Task Manager. If not, stop and suggest `bootstrap-product` (do not start it).
- The Task Manager is writable per the capability snapshot; re-validate per `references/task-manager.md` §3 if the snapshot is older than 7 days or a write fails.
- If `specs.technical.approvedRevision` is already set, this is a **revision**: confirm the intent and follow `references/spec-revisions.md` (keep the approved revision until a new one is approved).

## 3. Read

- `.leshi/task-manager.md` — this project's Task Manager playbook (where SPECs/features/subtasks live, which MCP tool does each operation). Read it before any Task Manager operation.
- `leshi.config.json`: `project` (platforms, repository, language), `taskManager`, `specs`, `testing`, `delivery`.
- The approved Product Spec (by id and revision): platforms, core capabilities, constraints (privacy, regulation, budget, timeline), success metrics.
- Developer-provided constraints: existing accounts and vendors, team skills, company standards.
- The repository only to detect what already exists (package manifests, CI files); do not read source in depth.
- `references/craft.md` §2 (the code-level decisions this workflow must settle) and `references/stack-profile.md` (the recommended defaults).

## 4. Questions

The skill — not the agents' spontaneity — guarantees these questions are covered. Ask in small batches, each with a recommended default and its tradeoffs. **Keep categories honest:** a framework (Vue, Angular, Expo, React Native) ≠ a runtime (Node, Bun, Workers isolates) ≠ packaging (Docker containers) ≠ a hosting/deploy platform (Cloudflare, AWS, a VPS, Vercel). Ask them separately; never offer them as equivalent options in one question.

For **each component** (frontend app(s), backend/API, workers/jobs, data stores):
1. Target platform / runtime / hosting?
2. How do we build and deploy (manual vs automatic, pipeline)?
3. Required capabilities and provider limits (filesystem, sockets/long connections, CPU/memory, execution timeout, cold starts, cron, native dependencies, connection limits)?
4. Environments (dev/staging/prod), secrets management, costs and CI/CD?
5. Local development and testing parity (emulators, local services, seed data)?
6. Existing constraints and preferred vendors?

Cross-cutting topics, as applicable: frontend platform and framework (web SPA/SSR, mobile native vs React Native/Expo, desktop), backend runtime, API style/gateway, persistence (Postgres, SQLite/D1, Turso, ...), file storage, cache/queues, authentication/authorization, observability/monitoring, domain/DNS, secrets location (a secret manager or platform env — never the repo), security approach (data classification, privacy, OWASP-relevant risks), repository/monorepo layout, linters/formatters/conventions, and the **testing policy**: unit, integration and E2E tools; exact commands; target devices/browsers; coverage expectations; what must pass before the final commit/push.

**Code-level decisions** (craft §2), asked here so implementation never has to guess: language and strictness; the library for each concern the product needs (server state and cache keys, forms, styling, validation, data access/ORM, i18n, HTTP client); how the client/server contract is kept in sync (types duplicated per app with contract tests, or a shared package); the frontend test policy (logic tests plus device QA, plus component tests, plus E2E); lint/format tooling and the `check` command. Backend tests are not a question — they are always required, against a local database only for tests (craft §9); only their tools are chosen. For every decision, offer the `references/stack-profile.md` choice as the recommendation **only when it meets the requirements**, next to real alternatives; the contract sync and the frontend test policy are always asked.

## 5. Approvals

- [GATE:deployment-choice] — runtime, hosting/deploy platform and build/deploy approach for **each** component, chosen by the developer after seeing the comparison. Never decided by an agent.
- [GATE:test-policy] — test tools, commands, devices/browsers, coverage expectations and `requiredBeforePush`, before writing them to `testing`.
- [GATE:technical-spec] — the complete Technical Spec text before it is saved as the approved revision.
- Any paid resource, account creation or vendor commitment is a separate explicit approval; this workflow never provisions infrastructure.

## 6. Agents

- `leshi-backend` (recommended): backend runtime vs containers vs serverless comparisons, storage, networking, runtime limits, monitoring, costs. Only options that **meet the Product Spec requirements**.
- `leshi-frontend`: web/mobile frameworks, distribution (stores, web hosting), target devices and their test tooling.
- `leshi-structure`: cross-layer tradeoffs, repository layout, conventions; for an existing repo, what is already in place.
- `leshi-research` (optional): how real projects solve a technical problem (documentation, public repositories, provider limits and pricing), with dated sources.
- Frontend and backend research are independent and may run in parallel; `leshi-structure` consolidates tradeoffs. Delegate per `references/delegation.md`; do small comparisons inline. Agents advise; the developer approves.

## 7. Scope

Allowed: conversation, documentation and pricing research, comparisons, writing the Technical Spec to the Task Manager, recording `testing`, `delivery` and `specs.technical` with `leshi config set`, the session log.

Not allowed: scaffolding projects, installing dependencies, creating cloud accounts or resources, writing application code, choosing for the developer, designing UI (that is `bootstrap-mvp`), starting any other workflow. [NO-AUTO-CHAIN]

## 8. Steps

1. **Preconditions and session.** Check §2. Start the session log (`leshi session start --workflow bootstrap-technology --ref <product-spec-id> --harness <id>`).
2. **Load product context.** Read the approved Product Spec; list the technical drivers (platforms, offline needs, realtime, media, AI/STT, privacy, scale, budget) in 5–10 bullets and confirm them.
3. **Identify components.** Propose the component list (e.g. mobile app, web app, API, background jobs, database, file storage) and confirm it.
4. **Research and compare.** Delegate to `leshi-backend` / `leshi-frontend` / `leshi-structure` when useful. For each decision present 2–4 options that satisfy the requirements, with capabilities and limits, operational burden, cost model and local-dev story, plus a recommendation.
   - *Example (Case B):* "I want a simple backend." Ask about scale, connection state, native dependencies, execution limits, cron, storage, networking and cost. Cloudflare Workers is an edge **runtime/platform** with its own constraints (execution time, no arbitrary native binaries, its own storage products). Docker is a **packaging** strategy: it still needs a host (VPS, container service, Kubernetes) and an operations policy. They are not equivalent answers to one question. The developer chooses; the Technical Spec records the contract.
5. **Cover the six questions per component.** Walk §4.1–4.6 for every component and record the answers. [CHECK:deployment-questions-covered] — before the Technical Spec draft, verify every component has an answer (or an explicit "not applicable / decide later" approved by the developer) for all six questions; list any gaps.
6. **Decide runtime and deployment.** Present the per-component summary (runtime, hosting/deploy platform, build/deploy approach, environments, secrets location, estimated cost). [GATE:deployment-choice] Iterate on feedback in place.
7. **Define the test policy.** Backend tests are fixed: through the real entry point against a local database only for tests, with factories and shared fakes (craft §9); propose their tools and commands. Ask the frontend test policy (logic tests plus device QA, plus component tests, plus E2E). Propose E2E tooling, exact commands, devices/browsers (Android + iOS when both are targets; tablet when relevant), coverage expectations and the commands required before push. [GATE:test-policy] Then record it, e.g. `leshi config set testing '{"definedBy":"bootstrap-technology","unitCommand":"npm run test:unit","typecheckCommand":"npm run typecheck","lintCommand":"npm run lint","integrationCommand":"npm run test:backend","requiredBeforePush":["npm run test:unit","npm run test:backend","npm run typecheck"],"e2e":{"tool":"maestro","command":"maestro test e2e/","devices":["ios-simulator","android-emulator"],"requiredFor":["bug-reproduction","feature-local-validation"]}}'`. Record delivery defaults if discussed: `leshi config set delivery '{"targetBranch":"main"}'`.
8. **Write the Technical Spec draft.** Sections: Architecture overview (components and their interactions) · Platforms & frontend stack · Backend runtime & API · Hosting, deployment & environments (per component) · Data: persistence, storage, cache · Authentication & authorization · Security & privacy · Observability · CI/CD & release · Secrets management (where they live; never in the repo) · Testing policy · Code-level decisions (craft §2: language, library per concern, client/server contract sync) · Repository layout & conventions (linters, formatting, `check`, commits) · Local development parity · Costs & vendor constraints · Risks, open questions & decisions deferred. Present it. [GATE:technical-spec] Iterate on feedback in place.
9. **Save the approved revision.** Create or update the Technical Spec item per `references/task-manager.md` §5: body = approved text with `<!-- leshi:spec kind=technical revision=v1 status=approved -->`; snapshot comment; approval comments for [GATE:deployment-choice], [GATE:test-policy] and [GATE:technical-spec]; one `<!-- leshi:decision ... status=current -->` comment per major decision with its rationale.
10. **Record references.** `leshi config set specs.technical '{"ref":"<item-id>","approvedRevision":"v1"}'`; `leshi config validate`. Re-read the TM item to confirm it was saved.
11. **Close.** Update the session log to `completed`, produce the final report and suggest `bootstrap-mvp` as the next manual step.

## 9. Checkpoints

Write a session checkpoint after: drivers confirmed; component list confirmed; comparisons delivered; [CHECK:deployment-questions-covered] passed; [GATE:deployment-choice] approved; [GATE:test-policy] approved and `testing` recorded; [GATE:technical-spec] approved; Technical Spec saved and read back. Include TM item ids and revision.

## 10. Testing

No application tests run here. Verifications: [CHECK:deployment-questions-covered]; `leshi config validate` after each config write; a read-back of the saved Technical Spec item and its approval/decision comments. The commands recorded in `testing` are not executed now (there may be no code yet); say so in the report.

## 11. Write

- **Task Manager**: Technical Spec item (approved revision v1, snapshot comment, approval comments, decision comments).
- **Config** (`leshi config set`): `testing` (definedBy, commands, `requiredBeforePush`, `e2e`), `delivery` (if discussed), `specs.technical`.
- **Session log**: `.leshi/sessions/<run-id>.md`.
- Nothing else: no scaffolding, no dependencies, no infrastructure, no code.

## 12. Failures

- **No approved Product Spec**: stop; suggest `bootstrap-product` (do not start it).
- **TM write fails or capability missing**: re-run discovery; status *Awaiting Input*; never claim the spec was saved.
- **Documentation/pricing unavailable** (no web access): say so; mark the affected comparisons as assumptions and lower the confidence.
- **Developer defers a decision**: record it as an open question in the spec (approved as deferred); never pick it silently.
- **Requirements contradict each other** (e.g. budget vs scale): present the conflict and options; do not resolve it unilaterally.
- **Developer unavailable mid-gate**: checkpoint and stop as *Awaiting Approval* (not *Blocked*).
- **Blocked** only for genuine external impediments (e.g. a mandatory vendor approval outside the team), with cause and resume condition.

## 13. Completion

Done when all are true:
- [CHECK:deployment-questions-covered] passed for every component;
- [GATE:deployment-choice], [GATE:test-policy] and [GATE:technical-spec] were approved by the developer (verifiable in the TM);
- the Technical Spec approved revision exists in the TM and was read back;
- `testing` and `specs.technical` are in `leshi.config.json` and `leshi config validate` passes;
- the final report lists TM ids, config keys written, deferred decisions and the suggested next workflow (`bootstrap-mvp`), which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Compare only options that meet the requirements; 2–4 per decision is enough. No exhaustive surveys.
- Keep SPEC text in the TM; only ids and short status go in the session log.
- Later workflows (`implement-the-feature`) use the approved decisions without reconsidering them per subtask; changes go through `references/spec-revisions.md`.
- Do not re-ask decisions already recorded in the config or the approved spec unless the developer wants to change them.
- One manual run may span several conversations; resume with `references/sessions.md`.
