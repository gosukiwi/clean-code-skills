---
name: clean-typescript-objects-data
description: Use when writing, fixing, editing, or reviewing TypeScript data models, DTOs, discriminated unions, classes, object boundaries, optional fields, null or undefined absence, repeated conditionals, impossible states, or object-chain access.
---

# Clean Objects And Data

Choose the shape that matches the job. Plain data is best for transfer, rendering, serialization, and pattern matching. Objects or classes are useful when behavior and invariants belong together.

## Plain Data Versus Behavior

```ts
// Good - behavior belongs with the object
interface Employee {
  calculatePay(): number;
}

class SalariedEmployee implements Employee {
  constructor(private readonly salary: number) {}
  calculatePay(): number {
    return this.salary;
  }
}
```

Use classes or objects with methods when they protect invariants or enable true polymorphism. Prefer plain data when values are mostly read, rendered, serialized, or passed across boundaries.

## Model Impossible States Out

Avoid bags of optional fields that can contradict each other.

```ts
// Bad - many invalid combinations are possible
type Employee = {
  type: "SALARIED" | "HOURLY";
  salary?: number;
  hours?: number;
  rate?: number;
};

// Good - each variant has exactly the fields it needs
type Employee =
  | { type: "SALARIED"; salary: number }
  | { type: "HOURLY"; hours: number; rate: number };
```

## Exhaustive Dispatch

When dispatching on a discriminated union, make exhaustiveness compiler-enforced. Always include a `never` case that throws so adding a new union member breaks compilation until every dispatch site handles it.

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
    default:
      return assertNever(employee);
  }
}

function assertNever(value: never): never {
  throw new Error(`Unhandled employee type: ${JSON.stringify(value)}`);
}
```

## DTOs Versus Domain Models

Keep external DTOs separate from domain models when names, nullability, units, or invariants differ. Convert at the boundary, then use domain types internally.

## Avoid Excessive Object-Chain Knowledge

```ts
// Bad - reaching through another object's internals
const outputDir = context.options.scratchDir.absolutePath;

// Good - ask the owner object
const outputDir = context.getScratchDir();
```

Do not count dots mechanically. Optional chaining through API response data can be fine; coupling to another object's internals is the problem.

## Represent Absence Explicitly

Use `null` or `undefined` only when absence is an expected domain value and the caller can handle it plainly. Do not use `null` as a vague failure channel for parse errors, permission failures, missing configuration, or invalid state.

```ts
// Bad - caller cannot tell why there is no user
function findUser(id: string): User | null {
  // ...
}

// Good - absence is the domain concept
function findUser(id: string): User | undefined {
  // ...
}
```

For recoverable failures, use the project's established result style or throw an error with context.

## Replace Repeated Conditionals

If the same `switch`, mode check, or type conditional appears in multiple places, the design knowledge is duplicated. Centralize it in a domain operation, lookup table, strategy, or exhaustive discriminated-union dispatch.

```ts
// Bad - the same status branching spreads through the codebase
if (invoice.status === "paid") {
  // ...
}

// Good - one owner expresses the policy
if (canSendReceipt(invoice)) {
  // ...
}
```

## Common Mistakes

- Using one type for API payloads, forms, database rows, and domain objects.
- Adding optional fields instead of creating a new variant.
- Using classes with no invariants, state, or polymorphic behavior.
- Letting callers reach through nested objects instead of exposing the needed operation.
- Returning `null` for several unrelated failure cases.
- Copying the same status/type conditional into multiple modules.
