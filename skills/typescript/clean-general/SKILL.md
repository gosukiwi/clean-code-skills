---
name: clean-general
description: Use when writing, fixing, editing, or reviewing TypeScript with duplicated logic, magic values, unclear one-liners, mixed responsibilities, clutter, arbitrary code, or inconsistent abstraction levels.
---

# General Clean Code Principles

## Critical Rules

**G5: DRY (Don't Repeat Yourself)**

Every piece of knowledge has one authoritative representation.

```ts
// Bad - duplication
const taxRate = 0.0825;
const caTotal = subtotal * 1.0825;
const nyTotal = subtotal * 1.07;

// Good - single source of truth
const TAX_RATES: Record<string, number> = { CA: 0.0825, NY: 0.07 };
function calculateTotal(subtotal: number, state: string): number {
  return subtotal * (1 + TAX_RATES[state]);
}
```

**G16: No Obscured Intent**

Don't be clever. Be clear.

```ts
// Bad - what does this do?
return ((x & 0x0f) << 4) | (y & 0x0f);

// Good - obvious intent
return packCoordinates(x, y);
```

**G25: Replace Magic Numbers with Named Constants**

```ts
// Bad
if (elapsedTime > 86400) {
  // ...
}

// Good
const SECONDS_PER_DAY = 86400;
if (elapsedTime > SECONDS_PER_DAY) {
  // ...
}
```

**G30: Functions Should Do One Thing**

If you can extract another function, your function does more than one thing.

**G9: Delete Dead Code**

Delete unused branches, helpers, exports, and unreachable code. Git preserves history; dead code makes future design decisions harder to read.

## Review Checklist

When reviewing AI-generated code, verify:
- [ ] No duplication (G5)
- [ ] Clear intent, no magic numbers (G16, G25)
- [ ] Functions do one thing (G30)
- [ ] Dead code removed (G9)
