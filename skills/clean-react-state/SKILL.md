---
name: clean-react-state
description: Use when writing, fixing, editing, or refactoring React state, derived state, reducers, context, server state, loading/error/empty states, form state, or state ownership.
---

# Clean State

State should have one owner and one source of truth. Store the minimal facts that change over time; derive everything else.

## R6: Local First

Keep state close to where it is used. Lift state only when multiple components need to read or change it.

```tsx
// Bad - parent owns state used by one child
function Page() {
  const [isOpen, setIsOpen] = useState(false);
  return <DetailsPanel isOpen={isOpen} onOpenChange={setIsOpen} />;
}

// Good - child owns local UI state
function DetailsPanel() {
  const [isOpen, setIsOpen] = useState(false);
  // ...
}
```

## R5: Derived State

Do not store values that can be calculated from props, state, or query data during render.

```tsx
const completedItems = items.filter((item) => item.completed);
const hasCompletedItems = completedItems.length > 0;
```

Use `useMemo` only when the computation is expensive or referential stability matters.

## Reducers

Use `useReducer` when state transitions are related and event-like. Keep actions domain-specific.

```tsx
type CartAction =
  | { type: "item-added"; item: CartItem }
  | { type: "item-removed"; itemId: string }
  | { type: "discount-applied"; code: string };
```

Avoid reducers that simply proxy `setState` with generic actions like `{ type: "set-value" }`.

## Context

- Use context for values many descendants need.
- Split read-heavy state from write actions when it prevents unnecessary rerenders.
- Do not use context as a default replacement for props.

## R7: Server State

Server data, cache lifetime, background refresh, optimistic updates, and request status are not ordinary local UI state. Prefer the project's data-fetching library when one exists.

Components that render server state should account for every user-visible state the data source can produce: loading, error, empty, and success. Do not collapse missing data, failed data, and still-loading data into the same fallback unless the product behavior intentionally treats them the same.

## Common Mistakes

- Mirroring props into state without a reset strategy.
- Keeping both `selectedId` and `selectedItem` in state.
- Using global stores for state that belongs to one screen.
- Mixing remote cache state and transient UI state in one object.
- Rendering nullable server data as if loading, error, and empty states were equivalent.
