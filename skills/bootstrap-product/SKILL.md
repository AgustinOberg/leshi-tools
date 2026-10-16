---
name: bootstrap-product
description: "leshi workflow 01 (manual). Use ONLY when the developer explicitly asks to bootstrap or define a NEW product, e.g. 'bootstrap the product', '/bootstrap-product'. Researches the space and produces an approved Product Spec + Project Setup (Task Manager, design tools). No technology choices."
license: MIT
metadata:
  leshi-skill: "01"
  leshi-contract: "1"
  leshi-gates: "positioning-scope, project-setup, product-spec"
  leshi-shared: "conventions, task-manager, sessions, delegation, design-tools, spec-revisions"
---

# bootstrap-product

Turn the developer's product idea into an **approved Product Spec** and an **approved Project Setup** (Task Manager connected and verified, design tools chosen, platforms and repository recorded). This is the first step of the new-project path (product → technology → MVP). For an existing codebase use `bootstrap-existing-project` instead.

Read `references/conventions.md` first; it defines gates, statuses, questions policy and the final report. Task Manager details are in `references/task-manager.md`, design tools in `references/design-tools.md`.

## 1. Trigger

- Explicit developer request in the latest message: "bootstrap the product", "let's define the product", "/bootstrap-product", "$bootstrap-product", or equivalent in any language.
- Not triggered by: finishing `leshi init`, a vague product conversation, or another skill finishing. [NO-AUTO-CHAIN] If you are unsure the developer wants to start this workflow, ask once.

## 2. Preconditions

- `leshi.config.json` exists (else: tell the developer to run `leshi init` and stop).
- If `specs.product.approvedRevision` is already set, this is a **revision**, not a new bootstrap: confirm the intent and follow `references/spec-revisions.md` (keep the approved revision until a new one is approved).
- If the repository clearly contains an existing product (substantial source code, a running app), ask whether `bootstrap-existing-project` is the better entry point before continuing.
- A Task Manager is not required to *start*; choosing and connecting it is part of this workflow.

## 3. Read

- `.leshi/task-manager.md` — this project's Task Manager playbook, if it already exists (this workflow creates it in the project-setup step).
- `leshi.config.json`: `project`, `harnesses`, `taskManager`, `design`, `specs`.
- Any material the developer provides or points to: notes, docs, links, competitor names, references.
- Existing Task Manager content only after the TM is connected (an existing Product Spec item, ideas).
- Do not read the whole repository; at most the README and top-level layout to detect an existing product.

## 4. Questions

Ask only what cannot be inferred, in small batches with recommended defaults:
1. **Vision**: the idea in one paragraph, the problem, who has it, why now, what success looks like, known constraints (budget, timeline, regulation, privacy).
2. **Audience and platforms**: target users/personas and target platforms (web, iOS, Android, desktop). Platforms only; frameworks are decided in `bootstrap-technology`.
3. **References**: competitors and inspiring products from any category; what the developer likes or dislikes in them.
4. **Project setup** (one combined step, not a questionnaire):
   - Task Manager: Linear, Plane, GitHub Projects/Issues, Notion, Jira, or **Other (free text)**. The factory imposes none.
   - Low-fi tool, hi-fi tool and UI Kit tool: Figma, HTML/CSS, Penpot, tldraw (low-fi only) or Other.
   - Repository location/organization (single repo, monorepo, multiple repos) and where SPECs should live in the TM (project/board reference).
   - Language for SPECs and TM artifacts (`project.language`) if unset.
   - Skill/agent customization: `leshi init` already offered "use all defaults" vs per-component overrides. Only mention that the developer can revisit it now (`leshi config` + `leshi sync`); do not re-ask all 16 components.

## 5. Approvals

- [GATE:positioning-scope] — the proposed positioning (audience, value proposition, differentiation, core capabilities, explicit non-goals) before writing the Product Spec.
- [GATE:project-setup] — the tool choices, the Task Manager capability table and the Task Manager playbook (`.leshi/task-manager.md`), before relying on the TM as the source of truth.
- [GATE:product-spec] — the complete Product Spec text before it is saved as the approved revision.
- Material additions the developer did not ask for (new segments, monetization models, scope expansions) are proposals and need explicit approval.

## 6. Agents

- `leshi-research` (recommended when the market is unfamiliar or competitors are many): competitor landscape, user reviews and complaints, unmet needs, pricing patterns, with sources and evidence vs assumption separated.
- `leshi-design` (optional): curate design references and describe the desired visual language and UX patterns. It produces references, not designs.
- Research and design references are independent and may run in parallel. Delegate per `references/delegation.md`; do small work inline.

## 7. Scope

Allowed: conversation, research (web and provided sources), writing the Product Spec and project setup to the Task Manager, recording choices in `leshi.config.json` via `leshi config set`, writing the session log.

Not allowed: choosing frameworks, runtimes, hosting or databases (that is `bootstrap-technology`); producing UI Kit or designs (that is `bootstrap-mvp`); writing application code; creating repositories or paid resources without explicit approval; starting any other workflow. [NO-AUTO-CHAIN]

## 8. Steps

1. **Preconditions and session.** Check §2. Start the session log (`leshi session start --workflow bootstrap-product --harness <id>`).
2. **Capture the vision.** Ask the §4.1–4.3 questions not already answered. Summarize back in 5–8 bullets and confirm understanding.
3. **Research the space.** Delegate to `leshi-research` (and `leshi-design` for references) when useful, otherwise research inline. Cover: competitors (features, positioning, pricing, platform), user reviews (store reviews, forums) with recurring praise/complaints, target users and unmet needs, market signals, differentiation opportunities, design references (competitors + any category). Always cite sources and mark assumptions.
4. **Propose positioning and scope.** Audience/personas, problem statements, value proposition, differentiation, core capabilities (not features list yet), platforms, non-goals, risks and open questions. [GATE:positioning-scope] Iterate on feedback.
5. **Project setup — tools.** Ask §4.4 in one step. Record each choice right away: `leshi config set taskManager '<json>'`, `leshi config set design '<json>'`, `leshi config set project.platforms '<json>'`, `leshi config set project.language '"es-AR"'` (examples). For the GitHub TM, `projectRef` is `owner/repo` (issues) plus an optional Projects v2 number in `taskManager.mapping`.
6. **Connect the Task Manager and discover capabilities.** Follow `references/task-manager.md` §3 [CHECK:task-manager-capabilities]: connect over MCP (guide the developer through the harness's MCP login if needed; never ask for a token in chat), list tools, exercise only read calls, show the verified / advertised / unavailable table, then record it with `leshi tm record ...`.
   - **Playbook**: run `leshi tm playbook` to create `.leshi/task-manager.md` from the provider template, then complete it with what discovery showed: where SPECs, features, subtasks, decisions, approvals and status will live, and the exact MCP tool (and key arguments) for each operation. Keep it to tables and ids. It is part of what the developer approves at the project-setup gate.
   - **Read-only or insufficient TM** (cannot create/update items or comments): explain exactly what will not work, offer authorized options (permissions, another server, a verified CLI/API fallback the developer authorizes, another TM). Do not continue as if the TM were writable; do not simulate persistence. If the developer wants to proceed later, set the run to *Awaiting Input* and stop.
   - **Design tools**: check that the chosen tools are reachable (e.g. the Figma MCP server is connected) or note what the developer must set up before `bootstrap-mvp`. Never substitute a tool.
   [GATE:project-setup]
7. **Write the Product Spec draft.** Sections: Vision · Problem & opportunity · Target users/personas · User needs & jobs-to-be-done · Research summary (competitors, reviews, market) with sources · Differentiation · Core capabilities · Platforms · Design references & desired visual language · Non-goals · Risks, assumptions & open questions · Success metrics. Present it in the conversation (or as a TM draft/proposal comment if long). [GATE:product-spec] Iterate on feedback in place.
8. **Save the approved revision.** Create or update the Product Spec item in the TM following the provider strategy (`references/task-manager.md` §5): body = approved text with `<!-- leshi:spec kind=product revision=v1 status=approved -->`; snapshot comment; approval comment `<!-- leshi:approval gate=product-spec artifact=product-spec revision=v1 by=developer date=YYYY-MM-DD -->`. Also store a short "Project Setup" section (tools chosen, TM capabilities summary, repository, platforms) in the Product Spec item or as a linked item.
9. **Record references.** `leshi config set specs.product '{"ref":"<item-id>","approvedRevision":"v1"}'`. Verify by re-reading the item from the TM ([CHECK:task-manager-capabilities] — a write that cannot be read back is not saved).
10. **Close.** Update the session log to `completed`, produce the final report and suggest `bootstrap-technology` as the next manual step.

## 9. Checkpoints

Write a session checkpoint after: vision confirmed; research delivered (with source list link); [GATE:positioning-scope] approved; tools recorded; TM capabilities recorded; [GATE:product-spec] approved; Product Spec saved and read back. Include the TM item ids and revision.

## 10. Testing

No code tests. Verifications: the capability probe results ([CHECK:task-manager-capabilities]); a read-back of the saved Product Spec item and its approval comment; `leshi config validate` passes after config writes.

## 11. Write

- **Task Manager**: Product Spec item (approved revision v1, snapshot comment, approval comment), Project Setup section, optional research attachment/links. Labels per provider strategy.
- **Config** (`leshi config set`): `taskManager` (provider, connection, mapping, capabilities snapshot), `design.lowFi|highFi|uiKit`, `project.platforms`, `project.language`, `project.repository`, `specs.product`.
- **Session log**: `.leshi/sessions/<run-id>.md`.
- Nothing else: no code, no designs, no repository changes beyond the config and the files `leshi sync` regenerates.

## 12. Failures

- **TM connection fails / MCP not authenticated**: guide the developer through the harness's connect/login flow; status *Awaiting Input*; never fake the connection.
- **TM read-only or missing required operations**: see step 6; status *Awaiting Input* until the developer chooses an option.
- **TM temporarily unreachable after approval**: keep the approved text in the session log (no secrets), mark *Awaiting Input*, retry on resume; never claim it was saved.
- **Research sources unavailable** (no web access): say so, rely on developer-provided material, mark findings as assumptions.
- **Developer unavailable mid-gate**: checkpoint and stop as *Awaiting Approval*. That is not *Blocked*.
- **Blocked** only for genuine external impediments (e.g. the organization forbids the chosen TM), recorded with cause and resume condition.

## 13. Completion

Done when all are true:
- [GATE:positioning-scope], [GATE:project-setup] and [GATE:product-spec] were approved by the developer in this run (or earlier runs, verifiable in the TM);
- the Product Spec approved revision exists in the TM and was read back;
- `taskManager` (with capability snapshot), `design` and `specs.product` are in `leshi.config.json` and `leshi config validate` passes;
- the final report lists TM ids, config keys written, capability gaps and the suggested next workflow (`bootstrap-technology`), which you do **not** start. [NO-AUTO-CHAIN]

## 14. Limits

- Research is bounded: aim for the 5–10 most relevant competitors and the strongest signals; stop when new sources stop changing conclusions.
- Keep SPEC text in the TM, not in `CLAUDE.md`/`AGENTS.md` or the session log (only ids and short status there).
- Do not re-ask decisions already recorded in the config unless the developer wants to change them.
- One manual run may span several conversations; resume with `references/sessions.md` instead of starting over.
