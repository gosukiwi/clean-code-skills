---
name: clean-general-names
description: Use when naming or renaming identifiers in any language, especially cryptic names, ambiguous parameters, type or scope encodings, misleading side effects, or requests for clearer names.
---

# Clean Names

## N1: Choose Descriptive Names

Names should reveal intent. If a name requires a comment, it doesn't reveal its intent.

```python
# Bad - what is d?
d = 86400

# Good - obvious meaning
SECONDS_PER_DAY = 86400

# Bad - what does this function do?
def proc(values: list[int]) -> list[int]:
    return [value for value in values if value > 0]

# Good - intent is clear
def filter_positive_numbers(numbers: list[int]) -> list[int]:
    return [number for number in numbers if number > 0]
```

## N2: Choose Names at the Appropriate Level of Abstraction

Don't pick names that communicate implementation; choose names that reflect the level of abstraction of the class or function.

```python
# Bad - too implementation-specific
def get_user_ids_to_names_dict():
    ...

# Good - abstracts the data structure
def get_user_directory():
    ...
```

## N3: Use Standard Nomenclature Where Possible

Use terms from the domain, design patterns, or well-known conventions.

```python
# Good - uses pattern name
class UserFactory:
    def create(self, data: object) -> "User":
        ...

# Good - uses domain term
def calculate_amortization(principal: Decimal, rate: Decimal, term: int) -> Decimal:
    ...
```

## N4: Unambiguous Names

Choose names that make the workings of a function or variable unambiguous.

```python
# Bad - ambiguous
def rename(source: str, target: str) -> None:
    ...

# Good - clear what's being renamed
def rename_file(old_path: str, new_path: str) -> None:
    ...
```

## N5: Use Longer Names for Longer Scopes

Short names are fine for tiny scopes. Longer scopes need longer, more descriptive names.

```python
# Good - short name for tiny scope
total = sum(n for n in numbers)

# Good - longer name for module-level constant
MAX_RETRY_ATTEMPTS_BEFORE_FAILURE = 5

# Bad - short name at module level
MAX = 5
```

## N6: Avoid Encodings

Don't encode type or scope information into names. Modern editors make this unnecessary.

```python
# Bad - Hungarian notation
str_name = "Alice"
list_users: list[str] = []
n_count = 0

# Good - clean names
name = "Alice"
users: list[str] = []
count = 0

# Bad - scope/type prefixes that repeat what tools know
self._list_of_user_records = []

# Good - name the concept
self.user_records = []
```

## N7: Names Should Describe Side Effects

If a function does something beyond what its name suggests, the name is misleading.

```python
config_store: dict[str, str] = {}

# Bad - name doesn't mention file creation
def get_config(config_path: str) -> dict[str, object]:
    if config_path not in config_store:
        config_store[config_path] = "{}"
    return json.loads(config_store.get(config_path, "{}"))

# Good - name reveals behavior
def get_or_create_config(config_path: str) -> dict[str, object]:
    if config_path not in config_store:
        config_store[config_path] = "{}"
    return json.loads(config_store.get(config_path, "{}"))
```
