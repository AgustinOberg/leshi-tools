## Figma — how this project uses it

| Concept | In this project |
|---|---|
| Files | One file per area: `<Product> — UI Kit`, `<Product> — Low-fi (MVP)`, `<Product> — Hi-fi`. Keys/links in "Instance notes". |
| Pages | Hi-fi file: one page per feature (`<TM id> <feature name>`), frames per device and state. |
| Frame names | `<Feature> / <Device: Phone \| Tablet> / <State: Default \| Loading \| Empty \| Error \| …>` |
| UI Kit | Published library: variables (color, type, spacing, radius, elevation) + components with variants for every state. Hi-fi uses only library components and variables. |
| Review | Export each frame for review (screenshots) and link the node URL in the Task Manager. |
| Approval record | `<!-- leshi:design kind=hi-fi tool=figma ref=<node URL> version=<date or Figma version name> -->` in the TM approval comment. Name a Figma version ("Approved — <feature> vN") when the plan allows it. |

## Operation → tool (official Figma MCP; verify against the live tool list)

| Operation | Tool |
|---|---|
| `design.read` | `get_design_context` (node design + code hints), `get_metadata` (structure, node ids), `get_figjam` |
| `design.create` | `create_new_file`; frames and content through `use_figma` |
| `design.update` | `use_figma` |
| `design.export` | `get_screenshot` (frame images), `download_assets` |
| `design.components` | `search_design_system`, `get_variable_defs`, `get_libraries` |
| `design.link` | node URLs `https://www.figma.com/design/<file-key>/…?node-id=<id>` (from `get_metadata`) |
| `design.comment` / `design.version` | not exposed by the MCP server → comments live in the Task Manager; versions are named by hand in Figma |

## Conventions
- Never edit approved frames in place: duplicate to `vN+1`, keep the approved one until the new one is approved.
- Read the UI Kit (`search_design_system` / `get_variable_defs`) before drawing; do not invent tokens.
- Write permission depends on the seat and plan: if `use_figma` fails, report it (the tool is then review-only) — never switch tools silently.

## Instance notes
- _(File keys/links, team/project, library name, seat limitations.)_
