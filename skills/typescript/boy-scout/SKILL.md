---
name: boy-scout
description: Use when fixing, editing, changing, debugging, or reviewing existing TypeScript code, especially when an adjacent small cleanup is possible or the user asks for quick wins, polish, cleanup, or obvious improvements.
---

# The Boy Scout Rule

> "Always leave the campground cleaner than you found it."
> - Robert Baden-Powell

> "Always check a module in cleaner than when you checked it out."
> - Robert C. Martin, *Clean Code*

## The Philosophy

You don't have to make every module perfect. You simply have to make it **a little bit better** than when you found it.

If we all followed this simple rule:
- Our systems would gradually get better as they evolved
- Teams would care for the system as a whole
- The relentless deterioration of software would end

## When Working on Code

Every time you touch code, look for **at least one small improvement**:

### Quick Wins
- Rename a poorly named variable
- Delete a redundant comment
- Remove dead code or unused imports
- Replace a magic value with a named constant when the value has domain meaning
- Extract a deeply nested block into a well-named function

### Deeper Improvements
- Split a function that does multiple things
- Remove duplication
- Add missing boundary checks
- Improve test coverage near the changed behavior

## The Rule in Practice

```ts
// You're asked to fix a bug in this function:
function proc(d: number[], x: number[], flag = false): number[] {
  // process data
  for (const i of d) {
    if (i > 0) {
      if (flag) {
        x.push(i * 1.0825); // tax
      } else {
        x.push(i);
      }
    }
  }
  return x;
}

// Don't just fix the bug and leave.
// Leave it cleaner:
const TAX_RATE = 0.0825;

function processTaxablePositiveValues(values: readonly number[]): number[] {
  return values.filter((value) => value > 0).map((value) => value * (1 + TAX_RATE));
}

function processPositiveValues(values: readonly number[]): number[] {
  return values.filter((value) => value > 0);
}
```

If the caller controls the behavior, prefer separate functions over a boolean flag:

```ts
processPositiveValues(values);
processTaxablePositiveValues(values);
```

For compatibility changes, a small delegating wrapper can be acceptable while callers migrate:

```ts
function processPositiveValuesForLegacyCaller(
  values: readonly number[],
  shouldApplyTax = false,
): number[] {
  return shouldApplyTax ? processTaxablePositiveValues(values) : processPositiveValues(values);
}
```

**What changed:**
- Descriptive function names
- Clear parameter names
- Explicit TypeScript types
- Named constant for domain value
- No output argument mutation
- Boolean behavior isolated behind named functions

## Skill Orchestration

This skill coordinates with specialized skills based on what you're doing:

| Task | Trigger Skill |
|------|---------------|
| Writing/reviewing any TypeScript | `typescript-clean-code` (master) |
| Naming variables, functions, classes | `clean-names` |
| Writing or editing comments | `clean-comments` |
| Creating or refactoring functions | `clean-functions` |
| Reviewing code quality | `clean-general` |
| Writing or reviewing tests | `clean-tests` |

## The Mindset

**Don't:**
- Leave code worse than you found it
- Say "that's not my code"
- Wait for a dedicated refactoring sprint
- Make massive changes unrelated to your task

**Do:**
- Make one small improvement with every commit
- Fix what you see, even if you didn't break it
- Keep changes proportional to your task
- Leave a trail of quality improvements

## AI Behavior

When working on code:
1. Complete the requested task first
2. Identify at least one small cleanup opportunity
3. Apply the appropriate specialized skill
4. Note the improvement made if it is useful context

When reviewing code:
1. Load `typescript-clean-code` for comprehensive rule checking
2. Flag violations by rule number
3. Suggest incremental improvements, not complete rewrites

## The Boy Scout Promise

Every piece of code you touch gets a little better. Not perfect-just better.

Over time, better compounds into excellent.
