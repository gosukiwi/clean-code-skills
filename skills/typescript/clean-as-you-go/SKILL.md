---
name: clean-as-you-go
description: Use when fixing, editing, changing, or debugging existing TypeScript code, especially when an adjacent small cleanup is possible or the user asks for quick wins, polish, cleanup, or obvious improvements.
---

# The Boy Scout Rule

Make small, proportional improvements near the code you already need to touch. Do not turn a narrow task into an unrelated rewrite.

## When Working on Code

Every time you touch code, look for **at least one small improvement**:

### Quick Wins
- Rename a poorly named variable
- Delete a redundant comment
- Remove dead code or unused imports
- Replace a magic value with a named constant when the value has domain meaning
- Extract a deeply nested block into a well-named function
- Move a local variable declaration next to the code that uses it
- Inline a pointless wrapper or give an abstraction real responsibility

### Deeper Improvements
- Split a function that does multiple things
- Remove duplication
- Simplify a complex conditional with a named predicate
- Add missing boundary checks
- Add context to swallowed or vague errors
- Make async ordering, retry, timeout, or cancellation behavior explicit
- Move SDK, config, clock, random, or database construction out of domain logic
- Move unsafe external data parsing to a boundary
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
| Writing/reviewing any TypeScript | `clean-typescript` (master) |
| Naming variables, functions, classes | `clean-typescript-names` |
| Writing or editing comments | `clean-typescript-comments` |
| Creating or refactoring functions | `clean-typescript-functions` |
| File order, module cohesion, classes, dependency construction, or over-abstraction | `clean-typescript-modules` |
| Promises, async ordering, retries, timeouts, cancellation, races | `clean-typescript-async` |
| Error handling, fallbacks, catch blocks | `clean-typescript-error-handling` |
| APIs, JSON, env, storage, SDKs | `clean-typescript-boundaries` |
| DTOs, domain models, unions, object boundaries | `clean-typescript-objects-data` |
| Reviewing code quality | `clean-typescript-general` |
| Writing or reviewing tests | `clean-typescript-tests` |

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
1. Load `clean-typescript` for broad TypeScript review and skill routing
2. Flag violations by rule number
3. Suggest incremental improvements, not complete rewrites
