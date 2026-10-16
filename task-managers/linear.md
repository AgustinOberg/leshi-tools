## Where things live — Linear

| Concept | In this project |
|---|---|
| Global SPECs | Linear Documents in the project when the MCP server exposes documents; otherwise issues labeled `Spec` + `spec:<kind>`. Approved revision marker at the top: `<!-- leshi:spec kind=… revision=vN status=approved -->`. |
| SPEC history | Revision snapshots as comments (or document history if available); proposals as comments with `status=proposed`. |
| Ideas / Features / Bugs | Issues labeled `Idea` → `Feature`, `Bug` (or issue types if the team uses them). |
| Subtasks | Sub-issues (`parentId`). |
| Decisions / approvals | Comments with the leshi markers. |
| Status | Workflow states. Prefer dedicated states *Awaiting Input*, *Awaiting Approval*, *Ready for Review*; if the team cannot add them, use labels `status:*`. |
| Designs / PRs | Links/attachments on the issue (Linear attachments are URLs); GitHub PR integration if enabled. |

## Operation → tool (verify against the live tool list)

| Operation | Linear MCP |
|---|---|
| `project.read` | `list_projects` · `get_project` · `list_teams` |
| `item.list` / `item.search` | `list_issues {team?, project?, label?, query?}` |
| `item.get` | `get_issue {id}` |
| `item.create` / `item.children.create` | `create_issue {title, description, teamId, projectId?, parentId?, labelIds?}` (some versions: `save_issue`) |
| `item.update` / `label.apply` / `status.set` | `update_issue {id, …, stateId?, labelIds?}` |
| `comment.list` / `comment.create` | `list_comments {issueId}` · `create_comment {issueId, body}` |
| `status.read` | `list_issue_statuses {team}` |
| `link.add` | `create_attachment` / issue links, or a comment with the URL |

## Conventions
- Use the team's identifiers (`TRAIN-123`) as item ids everywhere.
- Keep state names exactly as listed in "Instance notes".

## Limitations & workarounds
- If documents are not exposed, SPECs live in issues.

## Instance notes
- _(Team key, project id, state ids/names, label ids.)_
