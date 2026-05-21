---
name: clean-code
description: Use when writing, fixing, editing, or refactoring TypeScript, Python, React, or CSS code. Not for PR or diff reviews — use clean-code-reviewer for those.
---

# Clean Code: Collection Router

Use this as the top-level entry point for the clean-code skill collection. It should activate the relevant track skills for the files and concerns in front of you, then report findings with the specialized rule IDs.

## Route By Work

| Work | Use |
|------|-----|
| Broad TypeScript quality, functions, modules, async flows, boundaries, data modeling, errors, tests | `clean-typescript` plus `clean-general`, `clean-general-names`, and `clean-general-comments` |
| Broad Python quality, functions, modules, async flows, boundaries, data modeling, errors, tests | `clean-python` plus `clean-general`, `clean-general-names`, and `clean-general-comments` |
| React components, JSX, hooks, effects, state, React tests, file ownership | `clean-react` plus `clean-typescript`, `clean-general`, `clean-general-names`, and `clean-general-comments` |
| CSS, CSS Modules, CSS-in-JS, Tailwind classes, inline styles, tokens, layout, visual accessibility | `clean-css` |

## Editing Process

When editing code with clean-code expectations:

1. Apply the requested change first.
2. Use the relevant track skill before making quality improvements.
3. Keep cleanup proportional to the touched code.
4. Report the concrete rule-backed improvements made.

## AI Behavior

Do not treat `clean-code` as a replacement for the specialized skills. It is a router.

For broad edits, include every relevant track instead of choosing only one. For example, a `.tsx` component with CSS Modules should use `clean-typescript`, `clean-react`, `clean-css`, and the general skills. A Python API handler should use `clean-python` and the general skills.
