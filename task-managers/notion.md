## Where things live — Notion

| Concept | In this project |
|---|---|
| Global SPECs | Pages in a "Specs" database (property `Kind` = product / technical / design-system / mvp, `Revision`, `Status`). Approved revision marker at the top of the page. |
| SPEC history | Previous revisions as child pages or comments; proposals as comments with `status=proposed`. |
| Ideas / Features / Bugs | A "Work" database with property `Type` (Idea / Feature / Bug / Subtask) and `Status`. |
| Subtasks | Relation property `Parent` (or sub-items). |
| Decisions / approvals | Page comments with the leshi markers. |
| Status | The `Status` property with the logical states as options. |
| Designs / PRs | URL properties or links in the page body. |

## Operation → tool (verify against the live tool list)

| Operation | Notion MCP |
|---|---|
| `item.list` / `item.search` | `notion-search` (and database queries if exposed) |
| `item.get` / `item.children.list` | `notion-fetch {id}` |
| `item.create` / `item.children.create` | `notion-create-pages {parent, properties, content}` |
| `item.update` / `label.apply` / `status.set` | `notion-update-page {page_id, properties/content}` |
| `comment.list` / `comment.create` | `notion-get-comments` · `notion-create-comment` |

## Instance notes
- _(Database ids, property names, status option names.)_
