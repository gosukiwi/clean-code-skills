---
name: clean-hooks
description: Use when writing, fixing, editing, reviewing, or refactoring React hooks, custom hooks, useEffect, dependency arrays, stale closures, subscriptions, refs, or memoization.
---

# Clean Hooks

Hooks should make stateful behavior explicit. Effects are for synchronization with systems outside React, not for deriving ordinary render data.

## Effects

Use `useEffect` for:

- Subscriptions and cleanup
- Browser APIs and imperative widgets
- Network calls when the project does not use a data-fetching library
- Synchronizing React state with an external system

Do not use `useEffect` for values that can be computed during render.

```tsx
// Bad - derived state via effect
const [fullName, setFullName] = useState("");

useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// Good - derived during render
const fullName = `${firstName} ${lastName}`;
```

## Dependencies

- Treat the dependency array as a correctness contract.
- Do not suppress exhaustive-deps without explaining the external invariant.
- Stabilize callbacks only when identity matters to a child, subscription, or memoized computation.
- Prefer moving logic inside the effect over hiding dependencies.

## Custom Hooks

Create a custom hook when behavior is reusable and stateful, or when it gives a complex effect a clear boundary.

```tsx
function useWindowSize(): WindowSize {
  const [size, setSize] = useState(getWindowSize);

  useEffect(() => {
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);

    function handleResize() {
      setSize(getWindowSize());
    }
  }, []);

  return size;
}
```

Keep hook APIs small. Return named objects for multiple values unless tuple ordering is a local convention.

## Refs

Use refs for imperative handles, DOM nodes, and mutable values that should not trigger render. Do not use refs to sidestep normal state flow.

## Common Mistakes

- Using effects to copy props into state.
- Adding empty dependency arrays to "run once" while reading changing values.
- Using `useMemo` or `useCallback` everywhere without a measured reason.
- Returning unstable object shapes from hooks that consumers put into dependency arrays.
