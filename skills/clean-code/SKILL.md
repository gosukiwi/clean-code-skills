---
name: clean-code
description: Use when writing, fixing, editing, reviewing, or refactoring code with clean code expectations across TypeScript, React, CSS, tests, names, functions, comments, boundaries, state, components, hooks, layout, or styling.
---

# Clean Code: Collection Router

Use this as the top-level entry point for the clean-code skill collection. It should activate the relevant track skills for the files and concerns in front of you, then report findings with the specialized rule IDs.

## Route By Work

| Work | Use |
|------|-----|
| Broad TypeScript quality, names, functions, comments, boundaries, data modeling, errors, tests | `typescript-clean-code` |
| React components, JSX, hooks, effects, state, React tests, file ownership | `react-clean-code` plus `typescript-clean-code` |
| CSS, CSS Modules, CSS-in-JS, Tailwind classes, inline styles, tokens, layout, visual accessibility | `css-clean-code` |
| Small cleanup near existing TypeScript edits | `boy-scout` plus the relevant focused TypeScript skill |

## Review Process

When asked to review using clean code:

1. Inspect the changed files and identify their languages, frameworks, and styling systems.
2. Load each relevant track skill:
   - TypeScript files: `typescript-clean-code`
   - React files: `react-clean-code` and `typescript-clean-code`
   - Styling changes: `css-clean-code`
3. Follow the track skill routing tables for focused skills when a concern is specific.
4. Report findings by severity with rule IDs from the relevant skill.

## Review Output

Lead with findings. For each finding, include:

```text
Severity: Block | Fix | Suggest
Rule: <rule-id>
Location: path/to/file:line
Issue: What is wrong and why it matters.
Suggested fix: Concrete next step.
```

If there are no findings, say that clearly and note any meaningful residual risk, such as missing tests or unverified visual behavior.

## Editing Process

When editing code with clean-code expectations:

1. Apply the requested change first.
2. Use the relevant track skill before making quality improvements.
3. Keep cleanup proportional to the touched code.
4. Report the concrete rule-backed improvements made.

## AI Behavior

Do not treat `clean-code` as a replacement for the specialized skills. It is a router.

For broad reviews, include every relevant track instead of choosing only one. For example, a `.tsx` component with CSS Modules should use `typescript-clean-code`, `react-clean-code`, and `css-clean-code`.
