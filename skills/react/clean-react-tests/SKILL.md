---
name: clean-react-tests
description: Use when writing, fixing, editing, or refactoring React tests with Testing Library, user-event, component rendering, accessibility queries, async UI, mocks, brittle fixtures, test data builders, or behavior coverage.
---

# Clean React Tests

React tests should describe what the user can observe and do. Prefer Testing Library queries that match accessible UI and user-event interactions that match real workflows.

## Query Priority

Prefer:

1. `getByRole` with accessible name
2. `getByLabelText`
3. `getByText` for non-interactive copy
4. `getByTestId` only when user-facing queries are not practical

```tsx
// Bad - coupled to markup
expect(container.querySelector(".save-button")).toBeTruthy();

// Good - coupled to user-visible behavior
expect(screen.getByRole("button", { name: "Save" })).toBeEnabled();
```

## Interactions

Use `userEvent` for user workflows. Use `fireEvent` only for low-level events that user-event cannot express.

```tsx
const user = userEvent.setup();

await user.type(screen.getByLabelText("Email"), "ada@example.com");
await user.click(screen.getByRole("button", { name: "Invite" }));

expect(await screen.findByText("Invitation sent")).toBeInTheDocument();
```

## Behavior Contracts

Assert what the user can observe or do: visible content, enabled or disabled controls, focus, selected state, expanded state, loading state, error messages, navigation, and other meaningful outcomes.

Do not test styling implementation details unless they are the public contract or the bug is specifically visual. Avoid assertions for CSS classes, `display: grid` vs `display: flex`, exact DOM nesting, spacing values, or layout primitives when the same user behavior remains intact.

```tsx
// Bad - locks in an incidental layout implementation
expect(screen.getByTestId("actions")).toHaveStyle({ display: "grid" });

// Good - checks the behavior the layout supports
expect(screen.getByRole("button", { name: "Save" })).toBeVisible();
expect(screen.getByRole("button", { name: "Cancel" })).toBeVisible();
```

## Async UI

- Use `findBy...` for elements that appear asynchronously.
- Use `waitFor` for assertions that need polling.
- Avoid arbitrary sleeps and timers unless the component behavior is timer-based.

## Mocking

- Mock network and browser boundaries, not the component under test.
- Keep mocks close to the behavior being tested.
- Keep mocked data realistic; use the Test Data guidance for large fixtures.

## Test Data

Use builders for repeated props, query results, provider state, and domain objects with many required fields. A test should override the fields that matter to the behavior and inherit valid defaults for everything else.

```tsx
// Bad - fixture noise hides the state being tested
render(<OrderSummary order={{ id: "1", status: "paid", lineItems: [], discounts: [] }} />);

// Good - the relevant state is the visible part
render(<OrderSummary order={buildOrder({ status: "paid" })} />);
```

Keep builders realistic and small. Inline JSX props are fine when every field matters or the object shape is tiny.

## Coverage

Test:

- Loading, empty, success, and error states
- Disabled and pending interactions
- Important accessibility states
- Boundary cases around conditional rendering

## Common Mistakes

- Asserting implementation details like component state, hook calls, or CSS classes without user impact.
- Locking tests to incidental style choices like grid vs flexbox, exact spacing, or DOM shape.
- Using snapshots as the primary assertion for interactive UI.
- Forgetting to await user interactions and async assertions.
- Testing child components again in every parent test instead of checking integration behavior.
- Building large inline props or query responses full of irrelevant fields.
