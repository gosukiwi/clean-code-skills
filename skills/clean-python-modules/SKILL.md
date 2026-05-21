---
name: clean-python-modules
description: Use when writing or reviewing Python modules — variables declared far from their use, module-level state with a single use site, files mixing unrelated responsibilities, domain code coupled to vendor SDKs, pass-through wrappers or empty abstractions, dependency construction inside business logic, hidden init/load/run sequences, broad __all__ exports, or package structure issues.
---

# Clean Python Modules

Modules should read as small, cohesive units. Keep related code close, expose one clear concept, and avoid abstractions that do not carry real responsibility.

## M1: Keep Declarations Close To Use

Declare variables near the code that needs them. A value defined at the top of a function but used much later forces the reader to remember stale context.

```python
# Bad - formatter is irrelevant until much later
def send_digest(users: list[User], now: datetime) -> Digest:
    formatter = DateFormatter("en_US")
    active_users = [user for user in users if user.active]
    recipients = [user.email for user in active_users]
    subject = f"Digest for {len(recipients)} users"
    return Digest(recipients=recipients, subject=subject, generated_at=formatter.format(now))

# Good - declaration sits with the idea that uses it
def send_digest(users: list[User], now: datetime) -> Digest:
    active_users = [user for user in users if user.active]
    recipients = [user.email for user in active_users]
    subject = f"Digest for {len(recipients)} users"
    formatter = DateFormatter("en_US")
    return Digest(recipients=recipients, subject=subject, generated_at=formatter.format(now))
```

Module-level constants are fine when they define a true policy or when their uses span the file. When a constant's uses cluster in one region, declare it next to that region.

## M2: Order Code For Top-Down Reading

Put the public operation first, then the private helpers it uses. A reader should be able to skim the file from high-level intent to implementation detail.

```python
def price_order(order: Order) -> Money:
    return apply_discounts(calculate_subtotal(order), order.discounts)

def calculate_subtotal(order: Order) -> Money:
    ...

def apply_discounts(subtotal: Money, discounts: list[Discount]) -> Money:
    ...
```

Prefer local consistency when the project already has a strong file-order convention.

## M3: Keep Modules Cohesive

A module should have one reason to change. Split files that mix unrelated policies, infrastructure, DTO mapping, CLI parsing, HTTP handlers, and domain calculations.

```python
# Bad - one file changes for pricing, HTTP, and email policy
async def fetch_order() -> OrderDto: ...
def calculate_order_total(order: Order) -> Money: ...
def format_receipt_email(order: Order) -> str: ...

# Good - each file has a focused reason to change
# order_client.py, order_pricing.py, receipt_email.py
```

## M4: Keep Dependency Direction Obvious

Higher-level domain code should not depend on low-level details unless that detail is the domain. Pass behavior through small protocols or functions when it prevents vendor or infrastructure coupling.

```python
# Bad - domain calculation knows the SDK shape
def calculate_invoice_total(invoice: stripe.Invoice) -> int:
    return sum(line.amount for line in invoice.lines.data)

# Good - boundary code maps vendor data into domain data first
def calculate_invoice_total(invoice: Invoice) -> int:
    return sum(line.amount_cents for line in invoice.lines)
```

This complements boundary rules: validate and map external data at the edge, then keep inner modules stable.

## M5: Avoid Empty Abstractions

Do not add wrappers, managers, services, base classes, or helpers that only rename one call without hiding complexity, protecting invariants, or improving a real boundary.

```python
# Bad - no responsibility, just another place to click through
def get_user_name(user: User) -> str:
    return user.name

# Good - the abstraction carries a domain rule
def get_display_name(user: User) -> str:
    return user.preferred_name or user.name
```

Delete abstractions that no longer earn their place. Small direct code is cleaner than thin indirection.

## M6: Separate Construction From Use

Keep dependency construction, environment/config reads, SDK clients, database connections, clocks, random generators, and other external state near application boundaries. Domain behavior should receive ready-to-use dependencies instead of constructing them internally.

```python
# Bad - business behavior is mixed with construction and config
async def send_receipt(order: Order) -> None:
    payments = StripePayments(os.environ["STRIPE_KEY"])
    emailer = SendGridEmailer(os.environ["SENDGRID_KEY"])
    receipt = await payments.create_receipt(order)
    await emailer.send(order.customer_email, receipt)

# Good - construction happens at the edge; behavior uses explicit dependencies
async def send_receipt(order: Order, payments: Payments, emailer: Emailer) -> None:
    receipt = await payments.create_receipt(order)
    await emailer.send(order.customer_email, receipt)
```

Do not add dependency injection ceremony for simple values or harmless local objects. This rule matters most when construction touches I/O, config, time, randomness, vendor SDKs, persistence, or anything that makes behavior hard to test or change.

## M7: Avoid Temporal Coupling

Avoid APIs that require callers to remember a hidden sequence such as `connect()`, then `load_schema()`, then `run()`. Return ready-to-use objects from factories, use context managers, or model the states explicitly so invalid order is unrepresentable.

```python
# Bad - caller must know the required order
importer = Importer()
await importer.connect()
await importer.load_schema()
await importer.run(file)

# Good - setup returns the usable dependency
importer = await create_importer(config)
await importer.run(file)
```

## M8: Keep Public Exports Small And Intentional

Every public function, class, module variable, or `__all__` entry becomes part of the package's design surface. Export the operation or type callers are meant to use; keep helpers private until another owner has a real need for them.

```python
# Bad - implementation details become dependencies
__all__ = ["normalize_line_item", "calculate_subtotal", "apply_invoice_discounts", "calculate_invoice_total"]

# Good - one intentional public operation
__all__ = ["calculate_invoice_total"]
```

## Python Module Notes

- Keep import-time work cheap and predictable. Avoid network calls, database connections, and config validation during import unless the module is explicitly a startup boundary.
- Prefer absolute imports within packages unless local convention differs.
- Do not use module-level mutable state as a hidden dependency for domain behavior (PY4). If that state is read or written across `await` boundaries, report the async race as A3 first.
