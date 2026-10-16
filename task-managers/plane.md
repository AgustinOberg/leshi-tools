## Where things live — Plane

| Concept | In this project |
|---|---|
| Global SPECs | Pages (if exposed) or work items with label `spec` + `spec:<kind>`; approved revision marker at the top. |
| Ideas / Features / Bugs | Work items with labels `idea` → `feature`, `bug` (or work item types). |
| Subtasks | Sub-work-items (parent). |
| Decisions / approvals | Comments with the leshi markers. |
| Status | Project states; add *Awaiting Input*, *Awaiting Approval*, *Ready for Review* or use labels. |

## Operation → tool (verify against the live tool list)

Map from the server's tools (names vary by version, e.g. `list_work_items` / `list_issues`, `create_work_item`, `update_work_item`, `list_comments`, `add_comment`). Record the mapping in this table after discovery.

| Operation | Plane MCP |
|---|---|
| `item.list` | _(fill)_ |
| `item.get` | _(fill)_ |
| `item.create` / `item.children.create` | _(fill)_ |
| `item.update` / `status.set` | _(fill)_ |
| `comment.create` | _(fill)_ |

## Instance notes
- _(Workspace slug, project id, state ids — self-hosted URL if applicable.)_
