# Design tools (shared)

The developer chooses the tools per project during `bootstrap-product` (`design.lowFi`, `design.highFi`, `design.uiKit` in the config). Choosing a tool is not producing designs. Use exactly the configured tool; if it cannot do something the workflow needs, say so and ask — never switch tools silently.

## Fidelity rules

- **UI Kit first.** The shared visual foundation (tokens: color, type, spacing, radius, elevation; core components and states; patterns) is designed and approved before any feature design.
- **Low-fi for ALL MVP features in one `bootstrap-mvp` run.** Structure, hierarchy, flows and navigation — no visual polish.
- **Hi-fi per feature, later**, in `inspect-the-feature`: phone + tablet when the product targets them (and other relevant form factors), using the approved UI Kit. Reuse the approved, current low-fi instead of redoing it.
- Every approval records the exact artifact version (file commit/hash, Figma/Penpot version or page link) in the TM approval marker.

## Providers

### HTML/CSS (no credentials; versioned in the repo)
Layout (default `design/` at the repo root; override with `design.<kind>.ref`):

```text
design/
  ui-kit/index.html            tokens + components + states (single page)
  ui-kit/tokens.css            design tokens as CSS custom properties
  low-fi/<feature-slug>.html   grayscale wireframes, one page per feature (all states)
  hi-fi/<feature-slug>/phone.html
  hi-fi/<feature-slug>/tablet.html
```
- Self-contained pages: no build step, no external network except an optional font. Hi-fi pages link `../../ui-kit/tokens.css`.
- Show every relevant state (default, loading, empty, error, edge cases) on the page, labeled.
- Commit the files on the working branch; link them in the TM (`<!-- leshi:design kind=hi-fi tool=html ref=design/hi-fi/x/phone.html version=<commit> -->`). Offer to open them in a browser for review.

### Figma (MCP)
- Use the Figma MCP server configured in the harness. Discover its tools first; capabilities differ between servers/plans (read designs, create/edit frames, export images).
- Keep one file/page per project area: `UI Kit`, `Low-fi — MVP`, `Hi-fi — <feature>`. Use components and variables from the UI Kit for hi-fi.
- If the server is read-only, generate designs with an authorized alternative only after asking (for example HTML), or ask the developer to create the frames and review them.
- Record node/page links and the file version in approval markers.

### Penpot
- Use the Penpot MCP server/plugin if configured; otherwise its API with credentials the developer manages outside the repo. Same structure as Figma (boards per area, shared library for the UI Kit).
- If neither is available, report the limitation and ask how to proceed.

### tldraw / other
- tldraw: low-fi only, as `.tldr` files under `design/low-fi/` or a shared board link the developer provides. Hi-fi needs another tool.
- Other: discover what the tool's integration can do, propose a storage/linking strategy, get approval, and record it in `design.<kind>.ref`.

## Visual quality bar

Professional, restrained, platform-native. No decorative "AI-style" gradients, glows, emoji or gratuitous animation. Accessibility: contrast, touch targets, dynamic type, focus order, motion that respects reduced-motion.
