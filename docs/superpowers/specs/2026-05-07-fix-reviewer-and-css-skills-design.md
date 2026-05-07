# Fix Code Reviewer File Organization Detection

## Problem

The `clean-code-reviewer` dispatches style reviews based on file content types but never checks file *placement*. When a PR creates files in the wrong location (violating ownership rules), the reviewer doesn't load `clean-react-file-organization` because the routing is content-based, not location-based.

## Target File

`skills/clean-code-reviewer/SKILL.md`

## Fix: Code Reviewer — File Organization Check

### Location in skill

"File Inspection" section (after the file classification table) and Style Pass "Step 1" routing table.

### Change

Add a new classification step: after running `git diff --name-only`, also run `git diff --diff-filter=AR <base>` to detect Added or Renamed files. When any `.ts`, `.tsx`, `.jsx`, or `.module.css` files appear in that list, the Style Pass must also load `../clean-react-file-organization/SKILL.md` and evaluate whether file placement follows ownership rules.

### Addition to the File Inspection table

| Pattern | Track |
|---------|-------|
| New or moved `.ts`, `.tsx`, `.jsx`, `.module.css` files (detected via `--diff-filter=AR`) | File Organization |

### Addition to Style Pass Step 1 routing table

| Files | Read |
|-------|------|
| New/moved `.ts`, `.tsx`, `.jsx`, `.module.css` | `../clean-react-file-organization/SKILL.md` |

### Rationale

File organization violations only occur when files are created or moved. Existing files that are only edited don't change location, so this check only fires when relevant. The file-organization skill covers both React and general TypeScript ownership patterns.

## Scope

- One edit, one file (`skills/clean-code-reviewer/SKILL.md`)
- No new skills created
