---
name: clean-code-reviewer
description: Use when reviewing code, pull requests, branches, diffs, or changed files for quality, correctness, security, performance, and style issues.
---

# Clean Code Reviewer

Orchestrates a full code review by inspecting the diff, selecting relevant skills, and dispatching concurrent review passes for style and correctness.

## Diff Detection

Determine what to review:

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

## Dispatch Concurrent Reviews

Run two review passes in parallel:

### Style Pass

Load the relevant clean-code skills based on file classification:

- TypeScript files → `clean-typescript`
- React files → `clean-react` and `clean-typescript`
- CSS/styling changes → `clean-css`
- Test files → `clean-typescript-tests` or `clean-react-tests`

Apply the loaded skills to the diff. Report findings with rule IDs from those skills.

### Correctness Pass

Load `clean-code-reviewer-correctness`. Apply it to the full diff. This pass reviews for functional bugs, security vulnerabilities, performance issues, and test coverage gaps. It ignores style.

## Merging Results

Combine findings from both passes into a single report. Deduplicate if both passes flag the same location. Prefer the correctness finding when they overlap, since functional bugs outrank style.

## Output Format

```text
## Style

[Findings grouped by file]

## Correctness

[Findings grouped by file]

## Verdict

[Approve | Approve with suggestions | Request changes]
[One-sentence summary of the most important finding, or confirmation that the change looks good]
```

Each finding uses this format:

```text
Severity: Block | Fix | Suggest
Category: Style (rule-id) | Correctness | Security | Performance | Test Coverage
Location: path/to/file:line
Issue: What is wrong and why it matters.
Suggested fix: Concrete next step.
```

## Severity Definitions

- **Block**: Will break production, cause data loss, introduce a security vulnerability, or silently corrupt behavior. Must be fixed before merge.
- **Fix**: Likely bug, likely vulnerability, or missing coverage for critical behavior. Should be fixed before merge.
- **Suggest**: Potential improvement worth considering. Does not block merge.

## Verdict Rules

- Any **Block** finding → Request changes
- Any **Fix** finding → Request changes
- Only **Suggest** findings → Approve with suggestions
- No findings → Approve

## AI Behavior

- Do not review generated files, lock files, or vendored dependencies.
- Do not flag issues in code that was not changed unless the change breaks it.
- Keep the review focused. A 10-line diff should not produce 30 findings.
- When unsure whether something is a bug or intentional, use Suggest severity and say why.
- If the diff is too large to review thoroughly in one pass, say so and focus on the highest-risk files.
