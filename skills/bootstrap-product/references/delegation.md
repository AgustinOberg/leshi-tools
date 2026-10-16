# Delegating to leshi-* agents (shared)

The main harness session is the orchestrator. There is no orchestrator agent. Specialists are optional helpers: spawn **0..N** of them only when independent work justifies the extra context, tokens and time.

## The seven roles

| Agent | Use for | Never |
|---|---|---|
| `leshi-structure` | codebase shape, boundaries, conventions, dependencies, architecture impact | implement product UI or business logic by default |
| `leshi-frontend` | screens, components, navigation, state, frontend integration + focused tests | redefine requirements or backend contracts |
| `leshi-backend` | APIs, domain logic, persistence, validation, contracts + focused tests; runtime/deploy comparisons | choose cloud/deploy without developer approval |
| `leshi-research` | users, reviews, competitors, market, references; evidence vs assumptions | write production code or make product decisions |
| `leshi-design` | UX flows, UI Kit, low-fi/hi-fi, interaction rationale, restrained polish | change scope or approved visual direction |
| `leshi-reviewer` | independent review of a diff vs SPECs: regressions, security, quality | edit files (read-only) or fix its own findings |
| `leshi-qa` | reproduce bugs, run local E2E journeys, compare UI with hi-fi, collect evidence | assume a pass without executing |

## When to delegate

Delegate when at least one is true:
- two or more pieces of work are independent and can run in parallel (research + design references; frontend + backend with an agreed contract);
- a focused investigation would flood the main context (codebase mapping, log analysis, E2E runs);
- an independent view adds real value (high-risk change → `leshi-reviewer`).

Do the work inline when it is small, sequential or tightly coupled to the conversation. Never spawn all seven "just in case". Specialists never spawn other agents.

## Brief (what the agent receives)

```text
Objective: <one bounded outcome>
Context refs: <TM item ids + Spec revisions, exact file paths, prior findings> (by reference, not pasted SPECs)
Allowed to modify: <paths or "read-only">
Interfaces/contracts agreed: <API shapes, component props, file ownership>
Constraints: <conventions, test policy, approved decisions it must not reopen>
Craft: <path to this skill's references/craft.md> (code-writing work only: how code is written, when to stop and question)
Done when: <objective criteria>
Return: the response contract below, concise.
```

## Response contract (what the agent returns)

`summary` · `scope` · `files touched` · `checks executed + result` · `evidence` · `findings` · `risks` · `decisions needed` · `next recommended step` · `confidence/limitations`. Keep it short; link to evidence instead of pasting logs.

## Parallel writes

- Run writing agents in parallel only with **disjoint file ownership** and contracts agreed before they start.
- Prefer one git worktree per independent writing task (`git worktree add <dir> -b <branch>`; `delivery.worktreesDir` in the config). The main session merges, resolves conflicts and re-runs affected checks.
- Never delete a worktree that has unmerged changes. Announce success only after integration and validation in the main checkout.

## Harness specifics

- **Claude Code**: Agent tool with `subagent_type: "leshi-<role>"` (definitions in `.claude/agents/`). Read-only roles have Edit/Write disabled.
- **OpenCode**: the `subagent` tool with the agent id `leshi-<role>` (`.opencode/agents/`). Read-only roles have `edit` denied.
- **Codex**: ask Codex to spawn the custom agent by name (`.codex/agents/leshi-<role>.toml`); subagents inherit the parent sandbox, so read-only roles are instructed, not enforced.
- **No subagent support / disabled**: perform the role inline, following the same brief and response contract, and say so in the final report.
