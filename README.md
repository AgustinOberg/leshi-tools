# leshi-tools

The public workflow **skills** and specialist **agents** of [leshi-ai](https://github.com/AgustinOberg), a configurable product & software factory for agentic harnesses (Claude Code, Codex, OpenCode).

> This repository is **generated** from the leshi-ai source on every change to `main`. Do not open PRs here.

## Skills (9 manual workflows)

| Skill | Use it when you ask… | Ends with |
|---|---|---|
| `bootstrap-product` | "bootstrap the product" | approved Product Spec + Project Setup |
| `bootstrap-technology` | "bootstrap the technology" | approved Technical Spec + test policy |
| `bootstrap-mvp` | "bootstrap the MVP" | approved UI Kit + low-fi of ALL MVP features |
| `bootstrap-existing-project` | "bootstrap existing project" | approved baseline SPECs + feature inventory (no code changes) |
| `inspect-the-feature` | "convert idea 12 to a feature" | approved Feature (WHAT, hi-fi, subtasks), no code |
| `implement-the-feature` | "let's do feature 12" | implemented subtasks, written per the craft, with tests and an independent review (never Ready for Review) |
| `enhance-the-feature` | "enhance feature 12" | restrained UI polish, or N/A |
| `test-feature-locally` | "test feature 12 locally" | local E2E + PR + Ready for Review when checks pass |
| `fix-the-bug` | "fix bug 48" | validated fix PR, or a no-change report |

Every skill is started manually and never starts another one. Human approvals pause the run.

## Methodology

The skills define the process; how the work is done lives in shared references every relevant skill ships in its `references/` folder:

- `craft.md` — how code is written, stack-agnostic: structure, boundaries and errors, frontend and backend rules, backend tests always against a local test database, the four moments to stop and question (library vs hand-built, code that smells, a spec or design that does not hold, platform limits) and a review checklist.
- `stack-profile.md` — the maintainer's preferred tools, offered as a recommendation only when they meet the project's requirements, never imposed.
- `addons.md` — how a project's own tools (MCP servers, skills, CLIs it connected with `leshi addon add`) are used at the steps marked `[USES:<moment>]`, and how a project changes a workflow or agent with `leshi extend`.
- `design-tools.md` — design tools, fidelity rules and the design craft (real products first, directions before a choice, every screen specified, tokens only, purposeful motion).

## Agents (7 specialists)

`agents/` holds the canonical definitions of `leshi-structure`, `leshi-frontend`, `leshi-backend`, `leshi-research`, `leshi-design`, `leshi-reviewer` and `leshi-qa`. skills.sh installs skills only; the `leshi` CLI converts these agents into each harness's native format.

## Playbook templates

`task-managers/` (GitHub, Linear, Jira, Notion, Plane, generic) and `design-tools/` (Figma, Penpot, HTML, tldraw, generic) are the starting points of each project's `.leshi/task-manager.md` and `.leshi/design.md`: how that project's tools are used (where things live, which MCP tool does each operation). `leshi tm playbook` and `leshi design playbook` copy only the templates of the tools the project chose.

## Install

The skills expect a project connected with the `leshi` CLI (`leshi init`), which installs them from here through skills.sh and keeps them on the latest version:

```sh
leshi init      # inside your project
```

Manual install (skills only):

```sh
npx skills add AgustinOberg/leshi-tools -a claude-code -a codex -a opencode
```

## Customizing

Each skill slot can be the default, your own skill (from skills.sh, GitHub or a local path) or the default plus a project extension. Custom skills must keep the slot's contract (approval gates, checks and the manual-trigger rule); `leshi contract <skill>` prints it.

## License

MIT
