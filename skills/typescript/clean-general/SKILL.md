---
name: clean-general
description: Use when writing, fixing, editing, or reviewing TypeScript with duplicated logic, magic values, unclear one-liners, long conditionals, mixed responsibilities, artificial coupling, or excessive object-chain knowledge.
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

**G23: Prefer Polymorphism or Exhaustive Dispatch to If/Else**

```ts
// Bad - will grow forever
function calculatePay(employee: {
  type: "SALARIED" | "HOURLY" | "COMMISSIONED";
  salary?: number;
  hours?: number;
  rate?: number;
  base?: number;
  commission?: number;
}): number {
  if (employee.type === "SALARIED") {
    return employee.salary ?? 0;
  } else if (employee.type === "HOURLY") {
    return (employee.hours ?? 0) * (employee.rate ?? 0);
  } else if (employee.type === "COMMISSIONED") {
    return (employee.base ?? 0) + (employee.commission ?? 0);
  }
  return 0;
}

// Good - open/closed principle
interface Employee {
  calculatePay(): number;
}

class SalariedEmployee implements Employee {
  constructor(private readonly salary: number) {}
  calculatePay(): number {
    return this.salary;
  }
}

class HourlyEmployee implements Employee {
  constructor(
    private readonly hours: number,
    private readonly rate: number,
  ) {}
  calculatePay(): number {
    return this.hours * this.rate;
  }
}

class CommissionedEmployee implements Employee {
  constructor(
    private readonly base: number,
    private readonly commission: number,
  ) {}
  calculatePay(): number {
    return this.base + this.commission;
  }
}
```

For plain data in TypeScript, a discriminated union plus exhaustive dispatch is often clearer than classes.

```ts
type Employee =
  | { type: "SALARIED"; salary: number }
  | { type: "HOURLY"; hours: number; rate: number }
  | { type: "COMMISSIONED"; base: number; commission: number };

function calculatePay(employee: Employee): number {
  switch (employee.type) {
    case "SALARIED":
      return employee.salary;
    case "HOURLY":
      return employee.hours * employee.rate;
    case "COMMISSIONED":
      return employee.base + employee.commission;
  }
}
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

**G36: Avoid Excessive Object-Chain Knowledge**

```ts
// Bad - reaching through multiple objects
const outputDir = context.options.scratchDir.absolutePath;

// Good - ask the owner object
const outputDir = context.getScratchDir();
```

Do not count dots mechanically. Optional chaining through API response data can be fine; coupling to another object's internals is the problem.

## Enforcement Checklist

When reviewing AI-generated code, verify:
- [ ] No duplication (G5)
- [ ] Clear intent, no magic numbers (G16, G25)
- [ ] Polymorphism, exhaustive dispatch, or clear data tables over growing conditionals (G23)
- [ ] Functions do one thing (G30)
- [ ] No excessive object-chain knowledge (G36)
- [ ] Boundary conditions handled (G3)
- [ ] Dead code removed (G9)
