---
name: clean-typescript-boundaries
description: Use when writing, fixing, editing, or reviewing TypeScript code that touches APIs, JSON, environment variables, storage, databases, browser APIs, SDKs, generated clients, or other external boundaries.
---

# Clean Boundaries

Boundary code converts uncertain external behavior into explicit internal contracts. Keep unsafe shapes at the edge and pass typed values inward.

## Validate At The Edge

External data is `unknown` until validated. Parse and normalize once, then use domain types internally.

```ts
// Bad - assertion lets invalid data into the system
const user = JSON.parse(responseText) as User;

// Good - boundary handles parsing, validation, and source context
const user = parseUserResponse(responseText);

function parseUserResponse(responseText: string): User {
  const payload = parseJson(responseText, "user response");
  return parseUser(payload);
}
```

Boundary parsers should reject invalid JSON, reject invalid shape, fill intentional defaults, and name the external source in errors.

## Do Not Leak Vendor Types

Wrap awkward SDKs and generated clients behind small adapters when their types or behavior should not shape the whole app.

```ts
type PaymentReceipt = {
  id: string;
  totalCents: number;
};

interface Payments {
  charge(request: ChargeRequest): Promise<PaymentReceipt>;
}
```

Keep vendor-specific fields in the adapter unless the domain truly needs them.

## Make Configuration Typed

Read environment variables, feature flags, and storage values once at startup or module boundaries. Convert strings into typed config before the rest of the app uses them.

## Boundary Tests

Use focused tests around boundaries that are easy to misunderstand:

- Parser rejects invalid external shapes.
- Adapter maps SDK responses into domain types.
- Missing config fails with a useful message.
- Browser/storage fallbacks behave consistently.

## Common Mistakes

- Letting `any` from a client library spread through application code.
- Repeating JSON shape checks in many call sites.
- Treating generated API models as domain models by default.
- Hiding unsafe parsing behind helper names like `getUser()` without validation.
