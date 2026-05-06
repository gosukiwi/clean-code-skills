---
name: clean-typescript-error-handling
description: Use when writing, fixing, or editing TypeScript error handling, catch blocks, thrown values, null failures, swallowed errors, retries, fallbacks, or Result-style APIs.
---

# Clean Error Handling

Error handling is part of the public behavior. Keep normal flow readable, preserve context, and make recoverable failures explicit.

## EH1: Throw Useful Errors

Throw `Error` objects or project-specific subclasses. Do not throw strings or untyped objects.

```ts
// Bad
throw "User not found";

// Good
throw new Error(`User not found: ${userId}`);
```

## EH2: Catch Unknown

Treat caught values as `unknown` and narrow before reading properties.

```ts
try {
  await importUsers(file);
} catch (error: unknown) {
  if (error instanceof Error) {
    throw new Error(`Failed to import users from ${file}: ${error.message}`, {
      cause: error,
    });
  }

  throw new Error(`Failed to import users from ${file}`);
}
```

## EH3: Do Not Swallow Failures

Empty `catch` blocks hide system behavior. Either handle the failure, add context and throw an `Error`, or return a typed recoverable result.

```ts
// Bad
try {
  await saveSettings(settings);
} catch {}

// Good
try {
  await saveSettings(settings);
} catch (error: unknown) {
  logger.error({ error }, "Failed to save settings");
  if (error instanceof Error) {
    throw new Error("Failed to save settings", { cause: error });
  }

  throw new Error("Failed to save settings");
}
```

## EH4: Null Is Not an Error Strategy

Use `null` or `undefined` only when absence is an expected domain value. For recoverable failures, prefer a typed result when the project already uses that style.

```ts
type ParseResult<T> =
  | { ok: true; value: T }
  | { ok: false; reason: string };
```

Do not introduce a Result abstraction into a codebase that consistently uses exceptions unless the boundary or domain clearly benefits from typed recovery.

## Common Mistakes

- Catching and rethrowing without context or without narrowing `unknown`.
- Returning `undefined` for both "not found" and "failed to load."
- Logging an error and continuing with corrupted state.
- Using broad fallbacks that make bugs look like valid empty data.
