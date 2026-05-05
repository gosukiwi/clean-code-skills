---
name: clean-css
description: Use when writing, fixing, editing, or refactoring CSS, CSS Modules, CSS-in-JS, styled-components, StyleX, Tailwind class usage, inline style props, design tokens, layout, visual accessibility, or responsive UI styling.
---

# Clean CSS: Index And Review Guide

CSS should express design intent through the project's styling system, keep layout relationships predictable, and preserve accessibility across states and viewport sizes.

## First Check Project Conventions

Before reviewing or editing styles, identify the local styling approach:

- CSS files or CSS Modules
- CSS variables, design tokens, theme objects, or utility classes
- Tailwind or another utility framework
- styled-components, Emotion, StyleX, or inline `style={{}}` props
- existing spacing, color, typography, radius, shadow, z-index, and breakpoint conventions

Prefer the project's established tokens and patterns. Do not invent a new token scale or styling abstraction for a narrow change.

## Core Rules

- CSS1: Use design tokens or existing utilities for repeated design values.
- CSS2: Keep CSS declarations purposeful; remove defaults, overrides, and no-op properties.
- CSS3: Prefer layout primitives over manual spacing and positioning.
- CSS4: Keep selectors shallow and specificity low.
- CSS5: Preserve visible focus, contrast, reduced motion, and disabled states.
- CSS6: Avoid static inline styles and arbitrary class strings.
- CSS7: Keep responsive behavior explicit and stable.
- CSS8: Avoid duplicated style knowledge.

## CSS1: Tokens For Design Values

Use theme tokens, CSS variables, or established utilities for colors, spacing, type, radius, shadows, z-index, and reusable transition values.

```css
/* Bad */
.card {
  color: #374151;
  padding: 1.875rem;
  border-radius: 13px;
  z-index: 100;
}

/* Good */
.card {
  color: var(--color-text-muted);
  padding: var(--space-lg);
  border-radius: var(--radius-md);
  z-index: var(--z-popover);
}
```

Hardcoded values are acceptable when they are intrinsic geometry or genuinely dynamic: `0`, `1px` hairlines, SVG coordinates, `clip-path` points, transform offsets, measured canvas values, animation keyframe coordinates, and one-off media query breakpoints that match project convention.

## CSS2: Purposeful Declarations

Every declaration should have a visible effect in context.

Remove declarations that:

- Repeat browser defaults without intent.
- Are overridden later in the same rule.
- Are nullified by display mode, positioning, or cascade.
- Duplicate a shorthand already present.

Prefer shorthand when it keeps intent clear and the project's styling system supports it. This applies to `margin`, `padding`, `border`, `background`, `transition`, `font`, and `inset`.

```css
/* Bad */
.button {
  display: flex;
  flex-direction: row;
  background: none;
  background-color: transparent;
}

.badge {
  z-index: 2;
}

.panel {
  padding-top: var(--space-sm);
  padding-right: var(--space-md);
  padding-bottom: var(--space-sm);
  padding-left: var(--space-md);
}

/* Good */
.button {
  display: flex;
  background: none;
}

.badge {
  position: relative;
  z-index: var(--z-raised);
}

.panel {
  padding: var(--space-sm) var(--space-md);
}
```

Be careful with framework resets, browser normalization, and CSS-in-JS constraints. A value that looks like a default may still be intentional if it counters a reset, inherited style, or third-party component. Some systems, such as StyleX, may restrict shorthand support; treat shorthand as a suggestion in those cases.

## CSS3: Layout Primitives Over Manual Spacing

Use flexbox, `gap`, alignment, and intrinsic sizing before margins, absolute positioning, or magic offsets. Prefer flexbox for layout when possible; reserve grid for cases that truly need two-dimensional row and column control.

```css
/* Bad */
.item {
  margin-bottom: var(--space-md);
}

.item:last-child {
  margin-bottom: 0;
}

/* Good */
.list {
  display: flex;
  flex-direction: column;
  gap: var(--space-md);
}
```

Margins between unrelated sections are fine. This rule targets repeated sibling spacing, manual alignment, and offsets that encode layout relationships in children.

Reserve `position: absolute` and `position: fixed` for overlays, anchored UI, measured placement, decorative layers, and intentionally removed-from-flow elements. If `top`, `right`, `bottom`, or `left` are used to align ordinary siblings, prefer flexbox or grid.

## CSS4: Low Specificity

Keep selectors shallow and local. Prefer one clear class over descendant chains.

```css
/* Bad */
.page .content .sidebar .nav .item .link {
  color: var(--color-link);
}

/* Good */
.navLink {
  color: var(--color-link);
}
```

Avoid `!important`. It usually means the cascade boundary, selector ownership, or component API is wrong. Only tolerate it for documented third-party overrides where the project already uses that escape hatch.

## CSS5: Visual Accessibility

Interactive elements need visible states:

- `:focus-visible` for keyboard focus
- hover and active states where the component uses pointer affordance
- disabled styles that do not rely on color alone
- sufficient text and UI contrast
- reduced-motion behavior for large motion or continuous animation

```css
/* Bad */
.trigger:focus {
  outline: none;
}

/* Good */
.trigger:focus-visible {
  outline: 2px solid var(--color-focus-ring);
  outline-offset: 2px;
}
```

Do not remove focus outlines without replacing them. If animation is more than a small transition, check `prefers-reduced-motion`.

## CSS6: Avoid Static Inline Styles

Static style values belong in the styling system, not `style={{}}`.

```tsx
// Bad
<button style={{ padding: "0.5rem 1rem", borderRadius: "8px" }} />

// Good
<button className={styles.button} />
```

Inline styles are acceptable for values computed at runtime: measured overlay coordinates, user-selected colors, chart dimensions, CSS custom property bridges, and canvas or animation values that cannot be known statically.

For Tailwind or utility classes, avoid long arbitrary-value strings when an existing utility or tokenized class communicates the same intent.

## CSS7: Responsive Stability

Styles should survive real content and common viewport sizes.

Check for:

- Text that can overflow buttons, cards, table cells, tabs, and narrow columns.
- Fixed widths or heights where content should determine size.
- `100vh` on mobile without accounting for dynamic browser chrome.
- Viewport-scaled font sizes that make text too small or too large.
- Layout shifts caused by hover, focus, loading, or validation states.
- Images, canvases, and iframes without stable dimensions or aspect ratios.

Prefer `minmax`, `clamp` for dimensions, `max-width`, `min-width: 0`, `overflow-wrap`, `aspect-ratio`, and container-aware layouts. Do not scale font size directly with viewport width unless the project already has a carefully bounded typography utility.

## CSS8: No Duplicated Style Knowledge

If the same group of three or more declarations appears in multiple places, check whether the project has a shared primitive, mixin, token, utility, or component style that should own it.

Duplication across files matters too: inputs, cards, modals, focus rings, empty states, and form errors should usually share one source of design knowledge.

Do not extract prematurely when two blocks only look similar by coincidence. Extract when the repeated styles represent the same design concept.

## Review Process

When reviewing a branch:

1. Determine the requested or default base branch, then run `git diff <base> --name-only`.
2. Inspect changed `.css`, `.module.css`, `.scss`, `.tsx`, `.jsx`, styled template literals, `stylex.create`, Tailwind class strings, and `style={{}}` props.
3. Read changed hunks in context before flagging a rule.
4. Report findings by severity and rule ID.

## Severity

- **Block**: Accessibility regression, broken responsive layout, unsafe `!important`, invisible focus, or likely user-visible breakage.
- **Fix**: Token drift, no-op declarations, duplicated style knowledge, brittle positioning, static inline styles, or avoidable specificity.
- **Suggest**: Shorthand, minor consistency, optional extraction, or style polish already near the changed code.

## AI Behavior

When reviewing code, identify violations by rule number, for example: "CSS5 violation: custom button removes focus outline without a replacement."

When editing code, preserve the existing styling system and report what changed, for example: "Fixed: replaced child margins with parent `gap` (CSS3)."

When unsure whether a value is arbitrary or project-standard, inspect nearby styles before flagging it.
