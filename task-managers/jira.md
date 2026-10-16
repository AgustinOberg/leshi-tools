## Where things live — Jira (Atlassian MCP)

| Concept | In this project |
|---|---|
| Global SPECs | Issues of type *Epic* (or a dedicated type) labeled `leshi-spec` + `spec-<kind>`, or Confluence pages if the team keeps specs there. Approved revision marker at the top of the description. |
| SPEC history | Revision snapshots as comments; proposals as comments with `status=proposed`. |
| Ideas / Features / Bugs | Labels `leshi-idea` → `leshi-feature` on Story/Epic; type *Bug* for bugs. |
| Subtasks | Sub-tasks (or child issues of an Epic). |
| Decisions / approvals | Comments with the leshi markers. |
| Status | Workflow transitions (map the logical states to transition names in "Instance notes"). |
| Designs / PRs | Remote links or comments with URLs. |

## Operation → tool (verify against the live tool list)

| Operation | Atlassian MCP |
|---|---|
| `project.read` | `getVisibleJiraProjects` |
| `item.list` / `item.search` / `item.children.list` | `searchJiraIssuesUsingJql {jql}` |
| `item.get` / `comment.list` | `getJiraIssue {issueIdOrKey}` |
| `item.create` / `item.children.create` | `createJiraIssue {projectKey, issueTypeName, summary, description, parent?}` |
| `item.update` / `label.apply` | `editJiraIssue {issueIdOrKey, fields}` |
| `comment.create` | `addCommentToJiraIssue {issueIdOrKey, commentBody}` |
| `status.read` / `status.set` | `getTransitionsForJiraIssue` · `transitionJiraIssue {transition}` |
| `link.add` | `createIssueLink` or a comment |

## Conventions
- Labels cannot contain `:` in some Jira setups: use `leshi-spec`, `status-awaiting-approval`.
- Descriptions may be ADF; keep markers in plain-text paragraphs.

## Instance notes
- _(Project key, issue types, transition names for each logical status.)_
