---
name: clean-python-async
description: Use when writing, fixing, or editing Python async flows, coroutines, asyncio tasks, retries, timeouts, cancellation, shared mutable state across awaits, race conditions, or flaky async tests.
---

# Clean Async Python

Async code should make ordering, ownership, and failure behavior explicit. Race-prone behavior deserves small units and direct tests.

## A1: Isolate Async Workflows

Keep orchestration separate from pure transformation. Parse, validate, map, and calculate in synchronous helpers when possible; let async functions coordinate I/O.

```python
async def import_users(file: AsyncUserFile, repository: UserRepository) -> None:
    rows = await file.read_rows()
    users = [parse_user_row(row) for row in rows]
    await repository.save_many(users)
```

## A2: Make Ordering Explicit

Use sequential `await` when order matters and `asyncio.gather` or `TaskGroup` when operations are independent. Do not create coroutines without awaiting or scheduling them.

```python
# Bad - coroutine objects are created but never awaited
for user in users:
    send_invite(user)

# Good - independent work is explicit
await asyncio.gather(*(send_invite(user) for user in users))
```

Prefer `asyncio.TaskGroup` when child task failures should cancel sibling work and propagate as a group.

## A3: Avoid Shared Mutable State Across Awaits

Mutable variables that survive across `await` boundaries invite races and stale assumptions. Prefer local values, immutable updates, locks with clear ownership, or a single owner for shared state.

```python
# Bad - concurrent calls can corrupt shared state
cached_user: User | None = None

async def get_user(user_id: str) -> User:
    global cached_user
    if cached_user is None:
        cached_user = await fetch_user(user_id)
    return cached_user

# Good - cache key and mutation ownership are explicit
user_cache: dict[str, asyncio.Task[User]] = {}

async def get_user(user_id: str) -> User:
    task = user_cache.get(user_id)
    if task is None:
        task = asyncio.create_task(fetch_user(user_id))
        user_cache[user_id] = task
    try:
        return await task
    except Exception:
        user_cache.pop(user_id, None)
        raise
```

If a cache stores tasks or futures, define the failure policy. Most request caches should evict failed tasks so one transient failure does not poison every future lookup.

## A4: Make Cancellation, Timeouts, Retries, And Fallbacks Visible

Async failure policy is behavior. Keep retry counts, timeout durations, cancellation handling, and fallback choices named and near the call that owns the policy.

```python
USER_LOOKUP_TIMEOUT_SECONDS = 2

async with asyncio.timeout(USER_LOOKUP_TIMEOUT_SECONDS):
    user = await fetch_user(user_id)
```

Do not hide broad fallbacks in shared clients unless every caller truly wants the same behavior. Do not swallow `CancelledError`; cancellation is control flow the caller owns.

## A5: Test Race-Prone Behavior

When code depends on ordering, cancellation, retry, timeout, or deduplication, test that behavior directly with controlled futures, fake clocks, or in-memory fakes.

```python
async def test_deduplicates_concurrent_user_fetches() -> None:
    fetch_user = AsyncMock(return_value=User(id="user-1"))
    users = create_user_loader(fetch_user)

    await asyncio.gather(users.get("user-1"), users.get("user-1"))

    fetch_user.assert_awaited_once()
```

Flaky async tests are usually a design signal: make the event, clock, future, or external dependency controllable.
