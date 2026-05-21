---
name: clean-python-tests
description: Use when writing, fixing, editing, or refactoring Python tests, especially slow or flaky tests, skipped or focused tests, happy-path-only coverage, missing boundaries, brittle fixtures, pytest fixture misuse, monkeypatch overuse, coverage gaps, or multi-concept tests.
---

# Clean Python Tests

## T1: Insufficient Tests

Test everything that could possibly break. Use coverage tools as a guide, not a goal.

```python
# Bad - only tests happy path
def test_divide() -> None:
    assert divide(10, 2) == 5

# Good - tests edge cases too
def test_divide_normal() -> None:
    assert divide(10, 2) == 5

def test_divide_by_zero() -> None:
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

def test_divide_negative() -> None:
    assert divide(-10, 2) == -5
```

## T2: Use a Coverage Tool

Coverage tools report gaps in your testing strategy. Don't ignore them.

```bash
pytest --cov
```

Aim for meaningful coverage, not 100%.

## T3: Don't Skip Trivial Tests

Trivial tests document behavior and catch regressions. They're worth more than their cost.

```python
def test_user_default_role() -> None:
    user = User("Alice")
    assert user.role == "member"
```

## T4: An Ignored Test Is a Question About an Ambiguity

Don't use `pytest.mark.skip`, `pytest.mark.xfail`, or broad marker exclusions to hide problems. Either fix the test or document the real external constraint.

```python
# Bad - hiding a problem
@pytest.mark.skip(reason="flaky")
def test_async_operation() -> None:
    ...

# Good - explicit external constraint
@pytest.mark.skip(reason="requires Redis; see CONTRIBUTING.md")
def test_cache_invalidation_with_redis() -> None:
    ...
```

## T5: Test Boundary Conditions

Bugs congregate at boundaries. Test them explicitly.

```python
def test_pagination_boundaries() -> None:
    items = list(range(100))

    assert paginate(items, page=1, page_size=10) == items[:10]
    assert paginate(items, page=10, page_size=10) == items[90:100]
    assert paginate(items, page=11, page_size=10) == []

    with pytest.raises(ValueError):
        paginate(items, page=0, page_size=10)

    assert paginate([], page=1, page_size=10) == []
```

## T6: Exhaustively Test Near Bugs

When you find a bug, write tests for all similar cases. Bugs cluster.

```python
@pytest.mark.parametrize(
    ("year", "month", "expected"),
    [
        (2024, 1, 31),
        (2024, 2, 29),
        (2023, 2, 28),
        (2024, 4, 30),
        (2024, 12, 31),
    ],
)
def test_month_boundaries(year: int, month: int, expected: int) -> None:
    assert last_day_of_month(year, month) == expected
```

## T7: Patterns of Failure Are Revealing

When tests fail, look for patterns. They often point to deeper issues.

If all async tests fail intermittently, the problem usually is not just the tests. It may be shared state, uncontrolled scheduling, real time, or hidden I/O.

## T8: Test Coverage Patterns Can Be Revealing

Look at which code paths are untested. Often they reveal design problems. If you can't easily test a function, it probably does too much or constructs dependencies internally.

## T9: Tests Should Be Fast Enough To Run

Slow tests don't get run. Keep unit tests fast and isolate slower integration tests so developers can run the right feedback loop intentionally.

```python
# Bad - hits real database in a unit test
def test_user_creation() -> None:
    db = connect_to_database()
    user = db.create_user("Alice")
    assert user.name == "Alice"

# Good - uses in-memory fake
def test_user_creation() -> None:
    db = InMemoryUserRepository()
    user = db.create_user("Alice")
    assert user.name == "Alice"
```

## T10: Prefer Test Data Builders

Use test data builders or small factory helpers when setup objects are large, repeated, or full of irrelevant fields. Builders keep tests focused on the one fact that matters.

```python
# Bad - noisy fixture hides the behavior under test
order = Order(
    id="order-1",
    status="paid",
    customer_id="customer-1",
    line_items=[],
    discounts=[],
    created_at=datetime(2026, 1, 1, tzinfo=UTC),
)

# Good - default valid object, test overrides the relevant fact
order = build_order(status="paid")
```

Inline literals are fine when the shape is tiny and every field matters to the assertion. Avoid broad builders that hide important setup or create invalid domain objects by default.

## T11: Test Behavior Contracts, Not Incidental Implementation

Tests should fail when behavior breaks, not when harmless implementation choices change. Assert public outputs, state transitions, side effects at boundaries, interactions, and other observable outcomes. Avoid tests that only lock in private helper calls, intermediate data shape, algorithm steps, ordering of internal operations, or other details that can change without changing the contract.

Implementation-detail assertions are appropriate only when the detail is the contract, protects against a real bug, or covers a boundary where there is no better observable signal.

## Python Test Organization

- Keep pytest fixtures focused. A fixture that builds a whole world for one assertion hides the test's intent; report this as T11 when it obscures the behavior contract, or T1 when it masks missing coverage.
- Prefer explicit test setup over autouse fixtures except for truly global test infrastructure; report broad autouse setup as T11 when tests no longer show their relevant dependencies.
- Use `monkeypatch` at boundaries. Heavy monkeypatching inside domain code is often a T11 signal that the test is locking incidental implementation instead of behavior.
- Use `tmp_path`, in-memory fakes, and dependency injection for filesystem and I/O tests.
- Mark integration tests distinctly so fast unit tests stay easy to run.

## F.I.R.S.T. Principles

- **Fast**: Tests should run quickly
- **Independent**: Tests shouldn't depend on each other
- **Repeatable**: Same result every time, any environment
- **Self-Validating**: Pass or fail, no manual inspection
- **Timely**: Written before or with the code, not after

## One Concept Per Test

Multiple assertions are acceptable when they verify one behavior. Split the test when assertions describe different concepts, state transitions, or responsibilities.
