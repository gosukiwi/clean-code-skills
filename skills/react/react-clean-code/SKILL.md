---
name: react-clean-code
description: Use when writing, fixing, editing, reviewing, or refactoring React components, hooks, state, effects, JSX, or React tests in TypeScript projects.
---

# Clean React: Index And Review Guide

Use this with the TypeScript clean-code skills. React quality comes from small components, explicit data flow, minimal effects, clear state ownership, ownership-based file organization, and behavior-focused tests.

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
- R11: Organize files by dependency ownership; keep private dependencies with their owner and shared dependencies in the nearest `common/`.

## Skill Routing

| Work | Use |
|------|-----|
| Component props, JSX, conditional rendering | `clean-components` |
| Effects, dependency arrays, custom hooks | `clean-hooks` |
| Local state, derived state, reducers, server state | `clean-state` |
| React Testing Library tests | `clean-react-tests` |
| Owner folders, shared dependencies, CSS modules | `clean-file-organization` |
| Names, comments, functions, general TypeScript | TypeScript clean-code skills |

## Review Checklist

- [ ] Components have clear boundaries and one reason to change (R1).
- [ ] Props express one coherent concept and avoid mode explosions (R2).
- [ ] JSX is shallow enough to scan (R3).
- [ ] Effects synchronize with external systems and have correct dependencies (R4).
- [ ] State is not duplicated from props or derived data (R5).
- [ ] State ownership is local where possible and lifted only when needed (R6).
- [ ] Server/cache state is separate from local UI state (R7).
- [ ] Custom hooks expose a small, stable API for reusable stateful behavior (R8).
- [ ] Event handlers describe user intent and keep mutation paths obvious (R9).
- [ ] Tests use accessible queries and real user interactions (R10).
- [ ] Files follow dependency ownership, with private dependencies colocated and shared dependencies in nearest `common/` (R11).
