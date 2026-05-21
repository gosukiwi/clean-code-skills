---
name: clean-python-boundaries
description: Use when writing, fixing, or editing Python code that touches APIs, JSON, environment variables, storage, databases, framework request objects, SDKs, generated clients, subprocesses, files, or other external boundaries.
---

# Clean Python Boundaries

Boundary code converts uncertain external behavior into explicit internal contracts. Keep unsafe shapes at the edge and pass typed values inward.

## B1: Validate At The Edge

External data is untrusted until validated. Parse and normalize once, then use domain types internally.

```python
# Bad - unchecked dict shape spreads through the system
user = json.loads(response_text)
return User(id=user["id"], email=user["email"])

# Good - boundary handles parsing, validation, and source context
def parse_user_response(response_text: str) -> User:
    payload = parse_json_object(response_text, "user response")
    return parse_user(payload)
```

Boundary parsers should reject invalid JSON, reject invalid shape, fill intentional defaults, and name the external source in errors.

## B2: Make Configuration Typed

Read environment variables, feature flags, and storage values once at startup or module boundaries. Convert strings into typed config before the rest of the app uses them.

```python
@dataclass(frozen=True)
class AppConfig:
    port: int
    database_url: str

def load_config(env: Mapping[str, str]) -> AppConfig:
    return AppConfig(
        port=parse_port(env.get("PORT"), source="PORT"),
        database_url=required_env(env, "DATABASE_URL"),
    )
```

## B3: Do Not Leak Vendor Types

Wrap awkward SDKs and generated clients behind small adapters when their types or behavior should not shape the whole app.

```python
@dataclass(frozen=True)
class PaymentReceipt:
    id: str
    total_cents: int

class Payments(Protocol):
    async def charge(self, request: ChargeRequest) -> PaymentReceipt:
        ...
```

Keep vendor-specific fields in the adapter unless the domain truly needs them.

## B4: Boundary Tests

Use focused tests around boundaries that are easy to misunderstand:

- Parser rejects invalid external shapes.
- Adapter maps SDK responses into domain types.
- Missing config fails with a useful message.
- File, subprocess, framework request, or storage fallbacks behave consistently.

## Python Boundary Notes

- Avoid passing `dict[str, Any]` inward after parsing. Convert to dataclasses, `TypedDict`, Pydantic models, attrs classes, or project-local domain types.
- Keep ORM models out of domain logic unless persistence is the domain. Map rows to domain values at repository boundaries.
- Use context managers for files, transactions, locks, HTTP sessions, and temporary resources.
- Validate subprocess arguments and avoid shell strings built from untrusted data.

## Common Mistakes

- Letting `Any` from a client library spread through application code.
- Repeating JSON shape checks in many call sites.
- Treating generated API models, ORM rows, or framework request objects as domain models by default.
- Hiding unsafe parsing behind helper names like `get_user()` without validation.
