---
name: clean-general
description: Use when writing, fixing, editing, or reviewing TypeScript with duplicated logic, magic values, unclear one-liners, mixed responsibilities, clutter, arbitrary code, or inconsistent abstraction levels.
---

# General Clean Code Principles

## Critical Rules

**G1: No Duplicated Knowledge**

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

**G2: No Obscured Intent**

Don't be clever. Be clear.

```ts
// Bad - what does this do?
return ((x & 0x0f) << 4) | (y & 0x0f);

// Good - obvious intent
return packCoordinates(x, y);
```

**G3: Replace Magic Values with Named Constants**

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

**G4: Functions Should Do One Thing**

If you can extract another function, your function does more than one thing.

**G5: Delete Dead Code**

Delete unused branches, helpers, exports, and unreachable code. Git preserves history; dead code makes future design decisions harder to read.

## Review Checklist

When reviewing AI-generated code, verify:
- [ ] No duplicated knowledge (G1)
- [ ] Clear intent, no magic values (G2, G3)
- [ ] Functions do one thing (G4)
- [ ] Dead code removed (G5)
