---
name: clean-typescript-modules
description: Use when writing, fixing, or editing TypeScript modules, classes, file structure, declaration order, vertical formatting, dependency direction, cohesion, coupling, dependency construction, temporal coupling, public exports, wiring, or over-abstraction.
---

# Clean TypeScript Modules

Modules should read as small, cohesive units. Keep related code close, expose one clear concept, and avoid abstractions that do not carry real responsibility.

## M1: Keep Declarations Close To Use

Declare variables near the code that needs them. A value defined at the top of a function but used much later forces the reader to remember stale context.

```ts
// Bad - formatter is irrelevant until much later
function sendDigest(users: User[], now: Date) {
  const formatter = new Intl.DateTimeFormat("en-US");

  const activeUsers = users.filter((user) => user.active);
  const recipients = activeUsers.map((user) => user.email);
  const subject = `Digest for ${recipients.length} users`;

  return {
    recipients,
    subject,
    generatedAt: formatter.format(now),
  };
}

// Good - declaration sits with the idea that uses it
function sendDigest(users: User[], now: Date) {
  const activeUsers = users.filter((user) => user.active);
  const recipients = activeUsers.map((user) => user.email);
  const subject = `Digest for ${recipients.length} users`;
  const formatter = new Intl.DateTimeFormat("en-US");

  return {
    recipients,
    subject,
    generatedAt: formatter.format(now),
  };
}
```

Module-level constants are fine when they define a true policy or when their uses span the file. When a const's uses cluster in one region, declare it next to that region — being at module scope doesn't exempt it from proximity.

```ts
// Bad - top-of-file flag, only use is many lines later
const ENABLE_SCHEDULER = process.env["ENABLE_SCHEDULER"] === "true";
const app = express();
// ...many lines of route setup...
if (ENABLE_SCHEDULER) cron.schedule("0 8 * * 1", runWeeklyReport);

// Good - flag lives next to its only consumer
const app = express();
// ...many lines of route setup...
const ENABLE_SCHEDULER = process.env["ENABLE_SCHEDULER"] === "true";
if (ENABLE_SCHEDULER) cron.schedule("0 8 * * 1", runWeeklyReport);
```

Do not lift locals only to make a function look shorter.

## M2: Order Code For Top-Down Reading

Put the public operation first, then the private helpers it uses. A reader should be able to skim the file from high-level intent to implementation detail.

```ts
export function priceOrder(order: Order): Money {
  return applyDiscounts(calculateSubtotal(order), order.discounts);
}

function calculateSubtotal(order: Order): Money {
  // ...
}

function applyDiscounts(subtotal: Money, discounts: Discount[]): Money {
  // ...
}
```

Prefer local consistency when the project already has a strong file-order convention.

## M3: Keep Modules Cohesive

A module should have one reason to change. Split files that mix unrelated policies, infrastructure, DTO mapping, rendering helpers, and domain calculations.

```ts
// Bad - one file changes for pricing, HTTP, and email policy
export async function fetchOrder() {}
export function calculateOrderTotal() {}
export function formatReceiptEmail() {}

// Good - each file has a focused reason to change
// order-client.ts, order-pricing.ts, receipt-email.ts
```

## M4: Keep Dependency Direction Obvious

Higher-level domain code should not depend on low-level details unless that detail is the domain. Pass behavior through small interfaces or functions when it prevents vendor or infrastructure coupling.

```ts
// Bad - domain calculation knows the SDK shape
function calculateInvoiceTotal(invoice: Stripe.Invoice): number {
  return invoice.lines.data.reduce((total, line) => total + line.amount, 0);
}

// Good - boundary code maps vendor data into domain data first
function calculateInvoiceTotal(invoice: Invoice): number {
  return invoice.lines.reduce((total, line) => total + line.amountCents, 0);
}
```

This complements boundary rules: validate and map external data at the edge, then keep inner modules stable.

## M5: Avoid Empty Abstractions

Do not add wrappers, managers, services, base classes, or helpers that only rename one call without hiding complexity, protecting invariants, or improving a real boundary.

```ts
// Bad - no responsibility, just another place to click through
function getUserName(user: User): string {
  return user.name;
}

// Good - the abstraction carries a domain rule
function getDisplayName(user: User): string {
  return user.preferredName ?? user.name;
}
```

Delete abstractions that no longer earn their place. Small direct code is cleaner than a maze of thin indirection.

## M6: Separate Construction From Use

Keep dependency construction, environment/config reads, SDK clients, database connections, clocks, random generators, and other external state near application boundaries. Domain behavior should receive ready-to-use dependencies instead of constructing them internally.

```ts
// Bad - business behavior is mixed with construction and config
async function sendReceipt(order: Order) {
  const payments = new StripePayments(process.env.STRIPE_KEY);
  const emailer = new SendGridEmailer(process.env.SENDGRID_KEY);

  const receipt = await payments.createReceipt(order);
  await emailer.send(order.customerEmail, receipt);
}

// Good - construction happens at the edge; behavior uses explicit dependencies
async function sendReceipt(order: Order, payments: Payments, emailer: Emailer) {
  const receipt = await payments.createReceipt(order);
  await emailer.send(order.customerEmail, receipt);
}
```

Do not add dependency injection ceremony for simple values or harmless local objects. This rule matters most when construction touches I/O, config, time, randomness, vendor SDKs, persistence, or anything that makes behavior hard to test or change.

## M7: Avoid Temporal Coupling

Avoid APIs that require callers to remember a hidden sequence such as `init()`, then `load()`, then `run()`. Return ready-to-use objects from factories, or model the states explicitly so invalid order is unrepresentable.

```ts
// Bad - caller must know the required order
const importer = new Importer();
await importer.connect();
await importer.loadSchema();
await importer.run(file);

// Good - setup returns the usable dependency
const importer = await createImporter(config);
await importer.run(file);
```

## M8: Keep Public Exports Small And Intentional

Every export becomes part of the module's design surface. Export the operation, type, or component callers are meant to use; keep private helpers private until another owner has a real need for them.

```ts
// Bad - implementation details become dependencies
export function normalizeLineItem() {}
export function calculateSubtotal() {}
export function applyInvoiceDiscounts() {}

// Good - one intentional public operation
export function calculateInvoiceTotal() {}
```
