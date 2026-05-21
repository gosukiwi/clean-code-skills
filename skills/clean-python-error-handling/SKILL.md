---
name: clean-python-error-handling
description: Use when writing, fixing, or editing Python exception handling, bare except blocks, broad catches, raised values, None failures, swallowed errors, retries, fallbacks, or Result-style APIs.
---

# Clean Python Error Handling

Error handling is part of the public behavior. Keep normal flow readable, preserve context, and make recoverable failures explicit.

## EH1: Raise Useful Exceptions

Raise exception objects with useful context. Choose built-in exceptions or project-specific subclasses that match the failure.

```python
# Bad
raise Exception("User not found")

# Good
raise UserNotFoundError(f"User not found: {user_id}")
```

When wrapping an exception, preserve the cause.

```python
try:
    return parse_user(payload)
except ValueError as error:
    raise InvalidUserPayloadError(f"Invalid user payload from {source}") from error
```

## EH2: Catch Specific Exceptions

Avoid bare `except` and broad `except Exception` unless you are at a process boundary that logs, adds context, and re-raises or exits.

```python
# Bad - catches KeyboardInterrupt/SystemExit too
try:
    import_users(file_path)
except:
    logger.exception("Failed to import users")

# Good - catches the expected failure and preserves context
try:
    import_users(file_path)
except OSError as error:
    raise UserImportError(f"Failed to read users from {file_path}") from error
```

## EH3: Do Not Swallow Failures

Empty `except` blocks hide system behavior. Either handle the failure, add context and raise, or return a typed recoverable result.

```python
# Bad
try:
    save_settings(settings)
except OSError:
    pass

# Good
try:
    save_settings(settings)
except OSError as error:
    logger.exception("Failed to save settings")
    raise SettingsSaveError("Failed to save settings") from error
```

## EH4: None Is Not an Error Strategy

Use `None` only when absence is an expected domain value. For recoverable failures, prefer the project's established result style or raise a useful exception.

```python
@dataclass(frozen=True)
class ParseSuccess:
    value: User

@dataclass(frozen=True)
class ParseFailure:
    reason: str

ParseResult = ParseSuccess | ParseFailure
```

Do not introduce a Result abstraction into a codebase that consistently uses exceptions unless the boundary or domain clearly benefits from typed recovery.

## Common Mistakes

- Catching and rethrowing without context or `raise ... from error`.
- Returning `None` for both "not found" and "failed to load."
- Logging an error and continuing with corrupted state.
- Using broad fallbacks that make bugs look like valid empty data.
- Catching `Exception` around too much code and obscuring the operation that failed.
