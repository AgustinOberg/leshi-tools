# leshi-tools

Two [Agent Skills](https://skills.sh) that teach your coding agent to write frontend and backend code the way I do — in every project, without re-explaining the rules each time.

| Skill | What your agent learns |
|---|---|
| [`frontend-craft`](skills/frontend-craft/SKILL.md) | Components that only render and logic in purpose-named hooks, no effects without an external system, small composed components, server state in a query cache, a three-tier data layer, four states per data screen, tokens only, i18n and accessibility from the first screen. |
| [`backend-craft`](skills/backend-craft/SKILL.md) | A module per feature (controller → service → repository + DTOs), schema-validated boundaries, errors with stable codes, invariants in the database, paginated lists, vendors behind role modules, additive APIs and safe migrations — and every change shipped with tests against a local test-only database. |

Both skills also know when to **stop and ask** instead of guessing: before adding a dependency, before hand-building what a library solves, when a spec does not hold, and when the platform cannot do what is asked.

They are stack-agnostic: the examples use TypeScript, React, Hono, Zod and Drizzle, and each skill ships a preferred stack that is offered only as a recommendation.

## Install

With the [skills CLI](https://github.com/vercel-labs/skills), in your project:

```sh
npx skills add AgustinOberg/leshi-tools
```

Only one of them, or for every project on your machine:

```sh
npx skills add AgustinOberg/leshi-tools --skill frontend-craft
npx skills add AgustinOberg/leshi-tools --skill backend-craft -g
```

They load automatically when the task is frontend or backend work (Claude Code, Codex, Cursor, OpenCode and any agent that supports Agent Skills).

## Pair them with an AGENTS.md

The skills carry **how** code is written. Each project's [`AGENTS.md`](https://agents.md) (or `CLAUDE.md`) carries the **facts** of that project: commands, structure, decisions. The project's rules always win over the skills. A minimal one:

```md
# AGENTS.md

## Commands
- Install: `bun install`
- Check (types + lint + format): `bun run check`
- Tests: `bun run test`

## Decisions
- Mobile: Expo + Expo Router, Unistyles, TanStack Query, Zod.
- Backend: Hono on Cloudflare Workers, D1 + Drizzle, Vitest with a local test-only D1.
- Client/server contract: shared `packages/contracts` package.
- Frontend tests: pure logic only.

## Code
Follow the `frontend-craft` and `backend-craft` skills.
```

## Structure

```
skills/
  frontend-craft/
    SKILL.md        # core rules and workflow (always loaded when relevant)
    references/     # details and examples, read on demand
  backend-craft/
    SKILL.md
    references/
```

## License

MIT
