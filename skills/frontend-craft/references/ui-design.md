# UI design

The approved visual direction of the product always wins over any default here.

## Quality bar

Professional, restrained, platform-native. No decorative "AI-style" gradients, glows, emoji or gratuitous animation.

## Tokens only

- A spacing scale on a fixed base unit.
- A radius rule (for example one radius for actions, one for surfaces).
- Semantic color roles in background/foreground pairs (`surface` / `onSurface`, `danger` / `onDanger`).
- One named type ramp.
- No literal colors or sizes in components. In Tailwind, use theme tokens (`bg-surface`, `text-muted`, `p-4`), never arbitrary values (`bg-[#1a1a1a]`, `mt-[13px]`).
- Each theme (for example light and dark) is designed, not inverted.

## Hierarchy through restraint

- One accent color. A color with meaning is never decoration.
- At most one primary action per screen.
- Disabled reads quiet, not faded.
- Elevation only where it separates layers.

## Navigation has semantics

- Pick push, modal, sheet or overlay by the job, not by taste.
- Flows that must not go back (a completed payment, finished onboarding) remove back and draw their own exit.
- Unsaved edits get a discard guard.
- Every exit lands on a known screen.

## Motion has a purpose

- Purpose: feedback, orientation or continuity. Short. Entering eases out.
- Presses give immediate feedback.
- Every animation has a reduced-motion alternative.
- Decide what is deliberately not animated.
- No heavy animation libraries for trivial effects; no jank.

## Layouts respond to the window size class

Respond to the window size class, not to a device model. Larger screens get a composition (master-detail, columns), not a stretched phone layout.

## Images

- A cached image component (for example `expo-image` on native; `<img>` with `width`, `height` and `loading="lazy"` on web).
- Sources sized to the rendered box: thumbnails in lists, never the original upload.
- A placeholder that keeps the layout from shifting while the image loads.
- Decorative images are hidden from screen readers; meaningful ones have alt text.

## Accessibility is part of done

- Contrast meets WCAG AA for text and essential icons.
- Every interactive element has a label and a role; hit areas are at least 44×44 pt (iOS) / 48×48 dp (Android).
- Logical focus order; visible focus on web.
- Text scales with dynamic type without clipping.
- Reduced motion respected.

## i18n from the first screen

- Typed keys shaped `feature.screen.element` (`orders.list.emptyTitle`); shared strings live under `common.*` and error messages under `errors.*`, keyed by error code.
- No user-facing literal strings in components.
- Layouts survive longer translations and right-to-left when the product targets it.
- Dates, numbers and currency go through the i18n/formatting layer.

## When there is no design

Do not improvise screens straight into code. Follow this process and get the developer's approval at each choice:

1. **Real products first.** Study how strong products solve the same flow (in and outside the category): what to adopt, what is table stakes, what they all do badly. Cite each reference.
2. **Directions, then a choice.** For the visual direction, present 3–4 genuinely different directions side by side and recommend one. Record what was rejected and why.
3. **Specify every screen:** the flow it belongs to; how it is presented (push, modal, sheet, overlay); what back does (and where the flow is one-way); its states (default, loading, empty, error and the edge cases that matter); the copy for states that are only text.
4. **Make defaults visible.** List the small decisions you took as "defaults the developer can overrule".

## Restrained polish

When asked to polish a screen that already works:

- Compare it with the design and UI kit: hierarchy, spacing, alignment, typography, token usage, states (loading, empty, error, disabled), affordances, each theme.
- Add micro-interactions only where they give feedback: pressed, hover and focus states, gentle transitions, gesture feedback, haptics where native.
- Fewer, well-justified changes beat many cosmetic ones.
- Polish never changes behavior, scope, copy or approved design decisions. Anything that would is a proposal for the developer, not a change.
