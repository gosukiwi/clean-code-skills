---
name: clean-python-objects-data
description: Use when writing, fixing, or editing Python data models, dataclasses, TypedDicts, Protocols, enums, DTOs, classes, optional fields, None absence, repeated conditionals, impossible states, object-chain access, or dynamic data structures.
---

# Clean Python Objects And Data

Choose the shape that matches the job. Plain data is best for transfer, rendering, serialization, and pattern matching. Objects or classes are useful when behavior and invariants belong together.

## OD1-OD2: Plain Data Versus Behavior

```python
# Good - behavior belongs with the object
class Employee(Protocol):
    def calculate_pay(self) -> Money:
        ...

@dataclass(frozen=True)
class SalariedEmployee:
    salary: Money

    def calculate_pay(self) -> Money:
        return self.salary
```

Use classes with methods when they protect invariants or enable true polymorphism. Prefer dataclasses, `TypedDict`, tuples, or plain values when data is mostly read, rendered, serialized, or passed across boundaries.

## OD3: Model Impossible States Out

Avoid bags of optional fields that can contradict each other.

```python
# Bad - many invalid combinations are possible
@dataclass(frozen=True)
class Employee:
    type: Literal["SALARIED", "HOURLY"]
    salary: Money | None = None
    hours: Decimal | None = None
    rate: Money | None = None

# Good - each variant has exactly the fields it needs
@dataclass(frozen=True)
class SalariedEmployee:
    salary: Money

@dataclass(frozen=True)
class HourlyEmployee:
    hours: Decimal
    rate: Money

Employee = SalariedEmployee | HourlyEmployee
```

### Exhaustive Dispatch

When dispatching on variants, keep the fallback explicit so new variants cannot silently produce the wrong behavior.

```python
def calculate_pay(employee: Employee) -> Money:
    match employee:
        case SalariedEmployee(salary=salary):
            return salary
        case HourlyEmployee(hours=hours, rate=rate):
            return hours * rate

    assert_never(employee)

def assert_never(value: Never) -> Never:
    raise AssertionError(f"Unhandled employee type: {value!r}")
```

## OD4: DTOs Versus Domain Models

Keep external DTOs separate from domain models when names, nullability, units, defaults, mutability, or invariants differ. Convert at the boundary, then use domain types internally.

## OD5: Avoid Excessive Object-Chain Knowledge

```python
# Bad - reaching through another object's internals
output_dir = context.options.scratch_dir.absolute_path

# Good - ask the owner object
output_dir = context.get_scratch_dir()
```

Do not count dots mechanically. Access through API response data can be fine; coupling to another object's internals is the problem.

## OD6: Represent Absence Explicitly

Use `None` only when absence is an expected domain value and the caller can handle it plainly. Do not use `None` as a vague failure channel for parse errors, permission failures, missing configuration, or invalid state.

```python
# Bad - caller cannot tell whether the user is absent or loading failed
def load_user(user_id: str) -> User | None:
    ...

# Good - absence is the domain concept
def find_user(user_id: str) -> User | None:
    ...

# Good - failures raise useful exceptions
def load_user(user_id: str) -> User:
    ...
```

If the absence reason matters, use the project's established result style or raise an error with context.

## OD7: Replace Repeated Conditionals

If the same `match`, mode check, type check, or status conditional appears in multiple places, the design knowledge is duplicated. Centralize it in a domain operation, lookup table, strategy, or exhaustive dispatch.

```python
# Bad - the same status branching spreads through the codebase
if invoice.status == "paid":
    ...

# Good - one owner expresses the policy
if can_send_receipt(invoice):
    ...
```

## Python Data Notes

- Use `@dataclass(frozen=True)` for immutable domain values when mutation is not part of the concept.
- Use `Protocol` for behavior contracts and `TypedDict` for dictionary-shaped boundary data when a full model is unnecessary.
- Avoid `dict[str, Any]` as an internal domain type.
- Do not add classes with no invariants, state, behavior, or polymorphic role.

## Common Mistakes

- Using one type for API payloads, forms, database rows, and domain objects.
- Adding optional fields instead of creating a new variant.
- Using classes with no invariants, state, or polymorphic behavior.
- Letting callers reach through nested objects instead of exposing the needed operation.
- Returning `None` for several unrelated failure cases.
- Copying the same status/type conditional into multiple modules.
