# Task Manager protocol (shared)

The Task Manager (TM) is the **source of truth for product intent**: approved SPECs, decisions, features, subtasks, approvals and status. GitHub is the source of truth for the actual implementation. This file defines the provider-neutral model, how to discover what the configured TM can do, and how to store leshi artifacts in it.

## 1. Logical model

| Concept | Meaning |
|---|---|
| Project | The product being built (`taskManager.connection.projectRef`) |
| Spec | One of the four global SPECs: Product, Technical, Design System, MVP |
| SpecRevision | An approved (or proposed) version of a Spec: `v1`, `v2`, ... |
| Idea | An initial request (label/type *Idea*) |
| Feature | Root item with the approved functional WHAT; comments hold the HOW |
| Subtask | Child of a Feature/Bug with its own scope, acceptance criteria and dependencies |
| Bug | Defect report with expected vs actual behavior |
| Decision | A technical/product decision with rationale (current or superseded) |
| Approval | A developer approval of a gate for an exact artifact/revision |
| DesignRef | Link to a design artifact (low-fi, hi-fi, UI Kit) and its version |
| PullRequestRef | Link to the PR implementing an item, with the Spec revision it covers |
| StatusTransition | A status change (see `conventions.md` §4) |
| CapabilityManifest | What this TM connection can actually do (below) |

**Root vs children.** Feature root: *content* = WHAT (intent, requirements, acceptance criteria, current approved designs, attachments); *comments* = HOW (architecture and technical decisions with rationale; mark which are current, keep superseded ones as history). Subtask: *content* = scope + acceptance criteria + dependencies; *comments* = task-specific decisions; inherit global decisions from the parent instead of duplicating them.

## 2. Logical operations

| Operation | Needed for | If missing |
|---|---|---|
| `project.read` | locating the project | ask for the project reference |
| `item.list`, `item.get`, `item.search` | reading SPECs, ideas, features, bugs | **cannot use this TM** — stop and explain |
| `item.create`, `item.update` | writing SPECs, features, bugs | TM cannot be a written source of truth — stop and explain |
| `comment.list`, `comment.create` | decisions, approvals, evidence, revisions | store in the item body under a "Decisions/History" section only if the developer approves that strategy |
| `item.children.list`, `item.children.create` | subtasks | checklist in the parent body (developer approval required) |
| `label.apply` | Idea/Feature/Bug/Spec classification | use a type/kind field if the TM has one |
| `status.read`, `status.set` | workflow states | use labels such as `status:awaiting-approval` |
| `link.add` | linking designs, PRs | put the URL in a comment |
| `attachment.upload` | attaching design files | commit the file to the repo and link it |

Required for a TM to act as source of truth: read (`item.get`/`item.list`) **and** write (`item.create`, `item.update`, `comment.create`). Everything else may degrade with an explicit, developer-approved strategy.

## 3. Capability discovery — [CHECK:task-manager-capabilities]

Run this when the TM is chosen (bootstrap) and re-validate at the start of any workflow that writes to the TM if the snapshot is older than 7 days or an operation fails.

1. Identify the MCP server for the TM (config `taskManager.connection.serverRef`; `harness:<name>` means a server configured directly in the harness). If it is not connected, guide the developer to connect/authenticate it in the harness (OAuth or an environment variable). Never ask them to paste a token into the chat or the config.
2. List the server's tools, resources and prompts. Map each tool to the logical operations above. `leshi tm ops` prints the list; `leshi tm probe` performs the same discovery from the CLI when the server is reachable from it.
3. Exercise only **non-destructive** calls: get the project, list a few items, read one item and its comments. Do not create test items unless the developer agrees (and then delete or clearly label them).
4. Show the developer a capability table: verified (called successfully), advertised (tool exists, not exercised), unavailable.
5. Record the snapshot: `leshi tm record --via mcp --checked-by harness --verified <ops> --advertised <ops> --unavailable <ops> [--server <name>@<version>] [--note "..."]`. The snapshot is informative, never a permanent authorization: re-confirm critical operations at run time.
6. If a required operation is missing, explain what will not work and offer authorized options: grant permissions, use another server, use a verified CLI/API fallback (`taskManager.connection.fallback`, only if the developer authorizes it and it is verified the same way), or choose another TM. **Never** invent tickets, simulate persistence or pretend the TM was updated.

## 4. Markers inside TM content

Use these HTML comments so later runs can find artifacts without guessing. They are invisible in most renderers.

```text
<!-- leshi:spec kind=product|technical|design-system|mvp revision=v3 status=approved|proposed|superseded -->
<!-- leshi:decision id=D-007 status=current|superseded supersedes=D-003 -->
<!-- leshi:approval gate=<gate-id> artifact=<what> revision=<rev> by=developer date=YYYY-MM-DD -->
<!-- leshi:evidence run=<run-id> checks=<names> result=pass|fail|partial -->
<!-- leshi:design kind=low-fi|hi-fi|ui-kit tool=<provider> ref=<link-or-path> version=<rev> -->
<!-- leshi:pr url=<url> spec=<item>@<revision> -->
```

## 5. The project playbook — `.leshi/task-manager.md`

How THIS project's Task Manager is used lives in one short file, read **on demand** before any Task Manager operation (it is not loaded otherwise):

- **Connection block** (generated by `leshi sync` from the config): provider, MCP server/fallback, project ref, approved SPEC ids and revisions, and the latest capability table. Never edit inside its markers.
- **Below it** (owned by the project): where each concept lives (SPECs, features, subtasks, decisions, approvals, status, designs, PRs), the exact MCP tool and key arguments for each logical operation, conventions, limitations with the approved workarounds, and instance notes (ids, custom states, self-hosted URL — never credentials).

Rules:
1. Before reading or writing the Task Manager, read `.leshi/task-manager.md`. Use the tools and locations it names; do not guess another layout.
2. If the file is missing or still has `_(fill)_` placeholders for an operation you need, run `leshi tm playbook` (writes the template for the configured provider from leshi-tools below the connection block), then complete it from the discovered tools and show it to the developer for approval (`bootstrap-product` / `bootstrap-existing-project` do this inside their project-setup gate; other workflows ask before changing it).
3. When a tool name or behavior in the playbook turns out to be wrong, fix the playbook in the same run and tell the developer.
4. Keep it short: tables, ids and exact tool calls — not prose or SPEC content.
