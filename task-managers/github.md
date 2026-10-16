## Where things live — GitHub (Issues + sub-issues + Projects v2)

| Concept | In this project |
|---|---|
| Global SPECs | One issue each, labels `leshi:spec` + `spec:product` / `spec:technical` / `spec:design-system` / `spec:mvp`. Body = current approved revision with `<!-- leshi:spec kind=… revision=vN status=approved -->` at the top. Ids are in the connection block above. |
| SPEC history | Each approved revision is also a comment snapshot; proposals are comments with `status=proposed`. The body only changes after approval. |
| Ideas / Features / Bugs | Issues labeled `leshi:idea` → `leshi:feature` (relabel when the Feature is approved), `leshi:bug`. |
| Subtasks | **Sub-issues** of the Feature/Bug, labeled `leshi:subtask`. |
| Decisions (HOW) | Comments on the Feature (global) or subtask (local) with `<!-- leshi:decision id=D-n status=current -->`. |
| Approvals | Comments with `<!-- leshi:approval gate=… revision=… by=developer date=… -->`. |
| Status | Projects v2 single-select field (see `mapping.statusField`, project `mapping.projectNumber`). Without a project: labels `status:in-progress`, `status:awaiting-input`, `status:awaiting-approval`, `status:blocked`, `status:ready-for-review`. |
| Designs | Committed to the repo (`design/…`) and linked in a comment — issues cannot hold attachments through the API. |
| PRs | Linked from the item with `<!-- leshi:pr url=… spec=<item>@<rev> -->`; the PR body links back. |

## Operation → tool

Names observed on the official GitHub MCP server (`github-mcp-server`, 2026-10). Re-check them against the live tool list; fix this table if they differ.

| Operation | GitHub MCP | `gh` fallback |
|---|---|---|
| `item.list` | `list_issues {owner, repo, labels?, state?}` | `gh issue list -R o/r --label …` |
| `item.search` | `search_issues {query: "repo:o/r label:leshi:spec"}` | `gh search issues --repo o/r …` |
| `item.get` / `comment.list` / `item.children.list` | `issue_read {method: "get" \| "get_comments" \| "get_sub_issues", owner, repo, issue_number}` | `gh issue view N -R o/r --json body,labels,comments` · `gh api repos/o/r/issues/N/sub_issues` |
| `item.create` / `item.update` / `label.apply` | `issue_write {method: "create" \| "update", …, labels}` | `gh issue create` · `gh issue edit --add-label` |
| `item.children.create` | `sub_issue_write {method: "add", issue_number: <parent>, sub_issue_id: <child id, not number>}` | `gh api -X POST repos/o/r/issues/<parent>/sub_issues -F sub_issue_id=<id>` |
| `comment.create` / `link.add` | `add_issue_comment {owner, repo, issue_number, body}` | `gh issue comment N -R o/r --body …` |
| `status.read` / `status.set` | Projects tools (toolset `projects`, needs the `project` scope) | `gh project item-edit …` (needs `gh auth refresh -s project`) |

## Conventions
- Never delete or rewrite approved SPEC text; add a new revision.
- Keep issue bodies human-readable: the markers are HTML comments at the top.
- Reference items as `#<number>`; sub-issue APIs need the numeric issue **id** (`gh api repos/o/r/issues/N --jq .id`).

## Limitations & workarounds
- No attachments → commit + link.
- Projects v2 status unavailable without the `project` scope / projects toolset → status labels.
- The remote MCP server's default toolsets may not include projects; enable them per GitHub's MCP docs.

## Instance notes
- _(Filled in during bootstrap: project number, Status field options, label colors, anything specific to this repository.)_
