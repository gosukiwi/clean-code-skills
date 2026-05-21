# Clean Code Skills for AI Agents

This repository is a skill set — a collection of Agent Skills that teach AI
coding agents to write cleaner TypeScript, Python, React, and CSS. The skills
adapt Robert C. Martin's Clean Code rules for modern TypeScript and Python
projects.

## Structure

- `skills/` — all skill definitions, organized by track
  - `clean-code` — top-level router across all tracks
  - `clean-code-reviewer/` — orchestrates full code review (style + correctness)
  - `clean-code-reviewer-correctness/` — correctness-only review pass
  - `clean-general*` — language-neutral skills (general rules, names, comments)
  - `clean-typescript-*` — TypeScript skills (functions, modules, async, errors, boundaries, tests, etc.)
  - `clean-python-*` — Python skills (functions, modules, async, errors, boundaries, data, tests, etc.)
  - `clean-react-*` — React skills (components, hooks, state, tests, file organization)
  - `clean-css` — CSS skill (tokens, layout, accessibility, responsive)
- `docs/` — specs and design documents

## How Skills Work

Skills are markdown files with YAML frontmatter (`name` + `description`). They
are installed into a project's `.claude/skills/`, `.codex/skills/`, or
`.agents/skills/` directory. AI agents load them based on context matching
against the `description` field.

## When Editing Skills

- Each skill has a `SKILL.md` as its entry point
- Skills may reference sibling skills from inside a skill directory via relative
  paths (e.g., `../clean-react/SKILL.md`)
- The `clean-code-reviewer` dispatches concurrent style and correctness passes
- Rule IDs are stable (e.g., R11, CSS6, F3, PY4) — do not renumber without updating all references
- Test skill changes with AI agents before considering them complete
