---
name: react-clean-code
description: Use when writing, fixing, editing, reviewing, or refactoring React components, hooks, state, effects, JSX, or React tests in TypeScript projects.
---

# Clean React: Complete Reference

Use this with the TypeScript clean-code skills. React quality comes from small components, explicit data flow, minimal effects, clear state ownership, and behavior-focused tests.

## Core Rules

- R1: Components should have one reason to change.
- R2: Prefer composition over boolean prop combinations.
- R3: Keep JSX shallow enough to scan; extract named components for meaningful sections.
- R4: Effects synchronize with external systems; do not use them for ordinary data derivation.
- R5: Derive values during render when possible; do not mirror props or computed values into state.
- R6: Keep state as local as possible and lift it only when multiple owners need it.
- R7: Separate server/cache state from local UI state.
- R8: Custom hooks own reusable stateful behavior, not arbitrary helper code.
- R9: Event handlers should describe user intent and keep mutation paths obvious.
- R10: Tests should assert user-visible behavior, accessibility, and state transitions.

## Skill Routing

| Work | Use |
|------|-----|
| Component props, JSX, conditional rendering | `clean-components` |
| Effects, dependency arrays, custom hooks | `clean-hooks` |
| Local state, derived state, reducers, server state | `clean-state` |
| React Testing Library tests | `clean-react-tests` |
| Names, comments, functions, general TypeScript | TypeScript clean-code skills |

## Review Checklist

- [ ] Components have clear boundaries and names.
- [ ] Props express one coherent concept.
- [ ] Boolean props do not create many rendering modes.
- [ ] State is not duplicated from props or derived data.
- [ ] Effects are necessary and have correct dependencies.
- [ ] Custom hooks expose a small, stable API.
- [ ] Tests use accessible queries and real user interactions.
