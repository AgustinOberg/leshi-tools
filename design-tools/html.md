## HTML/CSS — how this project uses it

Designs are self-contained files in the repository, versioned with git (no credentials, no external service).

| Concept | In this project |
|---|---|
| UI Kit | `design/ui-kit/index.html` (tokens + components + states) and `design/ui-kit/tokens.css` (CSS custom properties) |
| Low-fi | `design/low-fi/<feature-slug>.html` — grayscale wireframes, every state labeled on the page |
| Hi-fi | `design/hi-fi/<feature-slug>/phone.html`, `…/tablet.html` — link `../../ui-kit/tokens.css`, only UI Kit components |
| Review | open the files in a browser; screenshots for the Task Manager/PR when a browser tool is available |
| Approval record | `<!-- leshi:design kind=hi-fi tool=html ref=design/hi-fi/<slug>/phone.html version=<commit> -->` |

## Operation → tool

| Operation | How |
|---|---|
| `design.read` / `design.create` / `design.update` | the harness's file tools on `design/…` |
| `design.components` | `design/ui-kit/tokens.css` + `design/ui-kit/index.html` |
| `design.link` | repo path (and the GitHub URL on the branch) |
| `design.version` | the git commit that contains the approved file |
| `design.export` | a browser/screenshot tool in the harness (_fill the exact tool_, or "unavailable": the developer opens the files) |

## Conventions
- No build step and no external network except an optional web font.
- Commit design files on the working branch; never overwrite an approved file — add `-v2` until the new one is approved.

## Instance notes
- _(Different root than `design/`? Browser tool used for screenshots? Target viewport sizes.)_
