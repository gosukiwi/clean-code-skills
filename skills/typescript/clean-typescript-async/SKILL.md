---
name: clean-typescript-async
description: Use when writing, fixing, editing, or reviewing TypeScript async flows, promises, retries, timeouts, cancellation, shared mutable state across awaits, race conditions, or flaky async tests.
---

# Clean Async TypeScript

Async code should make ordering, ownership, and failure behavior explicit. Race-prone behavior deserves small units and direct tests.

## A1: Isolate Async Workflows

Keep orchestration separate from pure transformation. Parse, validate, map, and calculate in synchronous helpers when possible; let async functions coordinate I/O.

```ts
async function importUsers(file: FileHandle, repository: UserRepository) {
  const rows = await file.readRows();
  const users = rows.map(parseUserRow);
  await repository.saveMany(users);
}
```

## A2: Make Ordering Explicit

Use sequential `await` when order matters and `Promise.all` when operations are independent. Do not rely on array callbacks with hidden promise behavior.

```ts
// Bad - promises are created but not awaited by forEach
users.forEach(async (user) => {
  await sendInvite(user);
});

// Good - independent work is explicit
await Promise.all(users.map((user) => sendInvite(user)));
```

## A3: Avoid Shared Mutable State Across Awaits

Mutable variables that survive across `await` boundaries invite races and stale assumptions. Prefer local values, immutable updates, or a single owner for shared state.

```ts
// Bad - concurrent calls can corrupt shared state
let cachedUser: User | undefined;

async function getUser(id: string) {
  if (!cachedUser) {
    cachedUser = await fetchUser(id);
  }
  return cachedUser;
}

// Good - cache key and mutation ownership are explicit
const userCache = new Map<string, Promise<User>>();

function getUser(id: string) {
  const existing = userCache.get(id);
  if (existing) {
    return existing;
  }

  const request = fetchUser(id).catch((error: unknown) => {
    userCache.delete(id);
    throw error;
  });
  userCache.set(id, request);
  return request;
}
```

If a cache stores promises, define the failure policy. Most request caches should evict rejected promises so one transient failure does not poison every future lookup.

## A4: Make Cancellation, Timeouts, Retries, And Fallbacks Visible

Async failure policy is behavior. Keep retry counts, timeout durations, abort signals, and fallback choices named and near the call that owns the policy.

```ts
const USER_LOOKUP_TIMEOUT_MS = 2_000;

await fetchUser(userId, {
  signal: AbortSignal.timeout(USER_LOOKUP_TIMEOUT_MS),
});
```

Do not hide broad fallbacks in shared clients unless every caller truly wants the same behavior.

## A5: Test Race-Prone Behavior

When code depends on ordering, cancellation, retry, timeout, or deduplication, test that behavior directly with controlled promises, fake timers, or in-memory fakes.

```ts
test("deduplicates concurrent user fetches", async () => {
  const fetchUser = vi.fn().mockResolvedValue({ id: "user-1" });
  const users = createUserLoader(fetchUser);

  await Promise.all([users.get("user-1"), users.get("user-1")]);

  expect(fetchUser).toHaveBeenCalledTimes(1);
});
```

Flaky async tests are usually a design signal: make the event, clock, promise, or external dependency controllable.
