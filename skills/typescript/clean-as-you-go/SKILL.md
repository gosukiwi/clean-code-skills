---
name: clean-as-you-go
description: Use when fixing, editing, changing, or debugging existing TypeScript code alongside a small cleanup opportunity near the touched code.
---

# The Boy Scout Rule

Leave the code cleaner than you found it — but only the code you already had to touch.

## The Rule

1. Complete the requested task first.
2. Scan the *immediate vicinity* of your change (same function, same block, nearby declarations) for one proportional improvement.
3. Apply `clean-typescript` rules to the touched code.
4. Do not touch code outside your change scope.

**Proportional means:** the cleanup matches the change size. A one-line fix warrants a renamed variable, not a refactored module.

## What to Look For

- Poorly named variable or parameter
- Redundant comment or dead code
- Magic value that should be a named constant
- Deeply nested block extractable into a named function
- Output argument mutation replaceable with a return value
- Boolean flag hiding two separate responsibilities

## Example

```ts
// Asked to fix a bug in:
function proc(d: number[], x: number[], flag = false): number[] {
  for (const i of d) {
    if (i > 0) {
      if (flag) { x.push(i * 1.0825); }
      else { x.push(i); }
    }
  }
  return x;
}

// Fix the bug AND leave it cleaner:
const TAX_RATE = 0.0825;

function processPositiveValues(values: readonly number[]): number[] {
  return values.filter((value) => value > 0);
}

function processTaxablePositiveValues(values: readonly number[]): number[] {
  return values.filter((value) => value > 0).map((value) => value * (1 + TAX_RATE));
}
```

The cleanup — descriptive names, named constant, no mutation, boolean flag removed — is proportional because it stays within the same function.

## AI Behavior

1. Complete the task first.
2. Scan the touched function or block for one improvement worth making.
3. Apply it only if it stays within the change scope.
4. Briefly note what was improved.

If the surrounding code is already clean, say so and move on. Do not manufacture a change to satisfy the rule.
