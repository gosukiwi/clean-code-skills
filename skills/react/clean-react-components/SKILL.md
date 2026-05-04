---
name: clean-react-components
description: Use when writing, fixing, editing, reviewing, or refactoring React components, JSX, props, discriminated union props, conditional rendering, loading/error/empty states, render purity, component boundaries, or component composition.
---

# Clean Components

React components are functions with UI responsibilities. Keep each component focused, make props readable at the call site, and use composition when variants start multiplying.

## Component Boundaries

A component should usually own one of these jobs:

- A reusable primitive, like `Button` or `TextField`
- A domain display, like `InvoiceSummary`
- A workflow section, like `ShippingAddressStep`
- A route or screen that coordinates smaller components

If a component fetches data, transforms it, manages several UI modes, and renders a large tree, split coordination from presentation.

## Props

```tsx
// Bad - call site meaning is unclear
<UserCard user={user} true false "compact" />

// Good - one named concept
<UserCard user={user} variant="compact" showActions={false} />
```

- Use object props for components; positional arguments do not apply to JSX.
- Keep prop names domain-specific.
- Avoid passing large objects when the component needs only a few fields, unless the object itself is the domain concept.
- Avoid boolean prop combinations that create unrelated modes.

When modes are mutually exclusive, model props as a discriminated union so invalid combinations cannot be rendered.

```tsx
// Bad - caller can pass contradictory props
type BannerProps = {
  isLoading?: boolean;
  error?: Error;
  message?: string;
};

// Good - each mode owns the props it needs
type BannerProps =
  | { status: "loading" }
  | { status: "error"; error: Error }
  | { status: "ready"; message: string };
```

## Composition Over Flags

```tsx
// Bad - one component owns unrelated modes
<Panel isModal isDismissible hasFooter />

// Good - structure communicates behavior
<Modal onDismiss={closeModal}>
  <PanelFooter actions={actions} />
</Modal>
```

Boolean props are fine for simple visual toggles. They are a smell when combinations create different components hiding behind one name.

## Conditional Rendering

- Prefer guard clauses for empty, loading, and error states.
- Keep nested ternaries out of JSX.
- Extract named render sections only when the name adds meaning.
- Render explicit loading, error, empty, and success states when remote or nullable data can be in those states.

```tsx
if (status === "loading") {
  return <Spinner />;
}

if (status === "error") {
  return <ErrorMessage error={error} />;
}

return <OrderDetails order={order} />;
```

## Render Purity

Rendering should only calculate UI. Do not mutate data, write storage, start timers, subscribe, navigate, log analytics, or trigger network work during render. Put effects in event handlers, `useEffect`, data-fetching boundaries, or framework loaders/actions.

## Common Mistakes

- Creating a `components/` dumping ground with no domain ownership.
- Extracting one-line components that hide simple JSX.
- Passing `className` through every layer instead of creating clear layout boundaries.
- Memoizing components before measuring a real render problem.
- Mutating arrays or objects while preparing JSX.
- Calling navigation, storage, analytics, or request code during render.
