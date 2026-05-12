---
name: clean-code-reviewer
description: Use when reviewing code, pull requests, branches, diffs, or changed files for quality, correctness, security, performance, and style issues.
---

# Clean Code Reviewer

Orchestrates a full code review by inspecting the diff, selecting relevant skills, and dispatching concurrent review passes for style and correctness.

## Review Target

If the user specifies what to review, use that target exactly. Valid targets include specific files, directories, commit ranges, branches, pull requests, changed files, or the full codebase.

If the user does not specify a target, follow Diff Detection below to determine what to review. Before starting, state the selected target briefly. Ask only when no meaningful target can be inferred.

## Diff Detection

For diff-based review targets, determine the comparison to inspect:

1. Run `git status`. If there are staged or unstaged changes, review the working tree diff (`git diff` and `git diff --cached`).
2. If the working tree is clean, find the base branch. Detect it with `git symbolic-ref refs/remotes/origin/HEAD` (strips the `refs/remotes/origin/` prefix), falling back to `main` then `master` if that fails. Diff the current branch against it: `git diff $(git merge-base HEAD <base>)..HEAD`.
3. If the branch has no commits ahead of the base, there is nothing to auto-detect. Ask the user what they want reviewed: specific files, a directory, or the full codebase.
4. Run `git diff --name-only` (with the appropriate target) to get the list of changed files.

## Full Codebase Review

When the user asks to review the entire codebase (or a large directory), a diff-based review does not apply. Instead:

1. **Prioritize by risk.** Review in this order:
   - Entry points and public API surfaces (exported modules, route handlers, SDK interfaces)
   - Authentication, authorization, and security-sensitive code
   - Complex business logic and state management
   - Data access, boundary code, and external integrations
   - Utilities, helpers, and shared components
2. **Batch by module.** Group files by their owning module or directory. Review one batch at a time, or dispatch concurrent agents per batch.
3. **Focus on structural issues.** Without a diff to narrow scope, prioritize findings that affect architecture, correctness, and security over minor style issues.
4. **Cap findings per batch.** Report the top 5-10 findings per module to keep the review actionable. Flag if a module needs deeper follow-up.
5. **Produce a summary.** After all batches, produce a prioritized list of the most impactful findings across the codebase.

## File Inspection

From the changed file list, classify each file:

| Pattern | Track |
|---------|-------|
| `.ts`, `.tsx` (no JSX) | TypeScript |
| `.tsx` with React imports or JSX | React + TypeScript |
| `.css`, `.module.css`, `.scss`, styled templates, Tailwind classes | CSS |
| Test files (`*.test.*`, `*.spec.*`) | Tests |
| New or moved `.ts`, `.tsx`, `.jsx`, `.module.css` files | File Organization |

To detect new or moved files, run `git diff --diff-filter=AR` (with the same target used above). If any `.ts`, `.tsx`, `.jsx`, or `.module.css` files appear, add "File Organization" to the tracks for the Style Pass.

## Dispatch Concurrent Reviews

Run two review passes in parallel:

### Style Pass

**Step 1 — Read the index skill(s)** based on file classification:

| Files | Read |
|-------|------|
| `.ts`, `.tsx` (TypeScript) | `../clean-typescript/SKILL.md` |
| React (JSX / React imports) | `../clean-react/SKILL.md` + `../clean-typescript/SKILL.md` |
| `.css`, `.module.css`, `.scss`, styled, Tailwind | `../clean-css/SKILL.md` |
| Test files (`*.test.*`, `*.spec.*`) | `../clean-typescript-tests/SKILL.md` or `../clean-react-tests/SKILL.md` |
| New/moved `.ts`, `.tsx`, `.jsx`, `.module.css` | `../clean-react-file-organization/SKILL.md` |

**Step 2 — Read relevant sub-skills.** For any TypeScript or React files, always load `clean-typescript-names`, `clean-typescript-comments`, and `clean-typescript-general` — naming, comment hygiene, and general code quality apply to all code regardless of diff content. For all other sub-skills, scan the diff and load only those matching the rule categories you observe. Do not load all sub-skills.

All skills are installed as siblings in the same directory. Reference TypeScript sub-skills with the sibling path pattern `../clean-typescript-{topic}/SKILL.md` and React sub-skills with `../clean-react-{topic}/SKILL.md`. Use the Skill Routing table in each index to map rule codes to sub-skill names.

**Step 3 — Apply and report.** Apply all loaded skills to the diff. Report findings with rule IDs.

### Correctness Pass

Load `clean-code-reviewer-correctness`. Apply it to the full diff. This pass reviews for functional bugs, security vulnerabilities, performance issues, and test coverage gaps. It ignores style.

## Merging Results

Combine findings from both passes into a single report.

**Deduplication rule:** If the correctness pass flags a code region, suppress any style finding that overlaps the same region. The correctness fix will resolve the style issue — reporting both is noise. Only keep the style finding if it addresses a genuinely independent concern at the same location.

## Output Format

```text
## Style

[Findings grouped by file]

## Correctness

[Findings grouped by file]

## Verdict

[Approve | Approve with suggestions | Request changes]
[One-sentence summary referencing finding codes, e.g. "Request changes — [S1] is a security hole; [C2] will break existing callers."]
```

Each finding uses this format, with its code as the leading identifier:

```text
[S1] Severity: Block | Fix | Suggest
Category: Style (rule-id)
Location: path/to/file:line
Issue: What is wrong and why it matters.
Fix: Concrete next step.
```

```text
[C1] Severity: Block | Fix | Suggest
Category: Correctness | Security | Performance | Test Coverage
Location: path/to/file:line
Issue: What is wrong and why it matters.
Fix: Concrete next step.
```

Style findings are numbered [S1], [S2], … in order of severity. Correctness findings are numbered [C1], [C2], … in order of severity. Numbers are assigned after merging so the user can reference any finding by code (e.g. "fix S2 and C1").

## Severity Definitions

- **Block**: Will break production, cause data loss, introduce a security vulnerability, or silently corrupt behavior. Must be fixed before merge.
- **Fix**: Likely bug, likely vulnerability, or missing coverage for critical behavior. Should be fixed before merge.
- **Suggest**: Potential improvement worth considering. Does not block merge.

## Verdict Rules

- Any **Block** finding → Request changes
- Any **Fix** finding → Request changes
- Only **Suggest** findings → Approve with suggestions
- No findings → Approve

After the verdict, always close with:

> Reply with the codes you want fixed (e.g. `fix S1 C2`), `all` to address everything, or `blocks` to fix only Block findings.

If there are no findings, omit this prompt.

## AI Behavior

- Do not review generated files, lock files, or vendored dependencies.
- Do not flag issues in code that was not changed unless the change breaks it.
- Keep the review focused. A 10-line diff should not produce 30 findings.
- When unsure whether something is a bug or intentional, use Suggest severity and say why.
- If the diff is too large to review thoroughly in one pass, say so and focus on the highest-risk files.
