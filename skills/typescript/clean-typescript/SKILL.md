---
name: clean-typescript
description: Use when writing, fixing, editing, or refactoring TypeScript code, especially weak types, unclear names, duplicated logic, oversized functions, stale comments, boundary gaps, error handling, data modeling, async flows, module structure, or brittle tests.
---

# Clean TypeScript: Index And Review Guide

Use this as the TypeScript entry point. It routes broad review work to focused skills and summarizes the rules most likely to affect design. Prefer clear runtime behavior and precise public types over cleverness or type-system escape hatches.

## Comments (C1-C5)
- C1: No metadata in comments (use Git)
- C2: Delete obsolete comments immediately
- C3: No redundant comments
- C4: Write comments well if you must
- C5: Never commit commented-out code

## Error Handling (EH1-EH4)
- EH1: Throw `Error` objects with useful context
- EH2: Catch `unknown` and narrow before reading properties
- EH3: Do not swallow failures
- EH4: Use typed recoverable results only when the project style or domain warrants it

## Functions (F1-F9)
- F1: Maximum 3 arguments (use parameter objects/interfaces for more)
- F2: No output arguments (return values)
- F3: No flag arguments (split functions)
- F4: Delete dead functions
- F5: Reduce nesting with early returns, helper functions, extraction, or refactoring
- F6: Keep one level of abstraction per function
- F7: Name and simplify complex conditions
- F8: Separate commands from queries
- F9: Keep side effects explicit and isolated

## General (G1-G5)
- G1: No duplicated knowledge
- G2: No obscured intent
- G3: Named constants, not magic values
- G4: Functions do one thing
- G5: Delete dead code

## Modules (M1-M8)
- M1: Keep declarations close to use
- M2: Order code for top-down reading
- M3: Keep modules cohesive
- M4: Keep dependency direction obvious
- M5: Avoid empty abstractions
- M6: Separate construction from use
- M7: Avoid temporal coupling
- M8: Keep public exports small and intentional

## Async (A1-A5)
- A1: Isolate async workflows
- A2: Make ordering explicit
- A3: Avoid shared mutable state across awaits
- A4: Make cancellation, timeouts, retries, and fallbacks visible
- A5: Test race-prone behavior

## TypeScript-Specific (TS1-TS3)
- TS1: Keep imports explicit and stable; avoid namespace-style overuse and implicit dependencies
- TS2: Prefer literal unions, discriminated unions, or const objects for closed sets; use enums only when they match project convention
- TS3: Type public interfaces explicitly; use `unknown` plus narrowing instead of `any` at boundaries

## Boundaries (B1-B4)
- B1: Treat external data as `unknown` until validated
- B2: Convert API, JSON, env, storage, database, and SDK shapes into internal types at the edge
- B3: Keep vendor types out of domain code unless they are the domain
- B4: Test tricky boundary mappings and invalid external shapes

## Objects and Data (OD1-OD7)
- OD1: Use plain data for transfer, rendering, serialization, and pattern matching
- OD2: Use objects/classes when behavior and invariants belong together
- OD3: Model impossible states out with discriminated unions
- OD4: Keep DTOs separate from domain models when external shape differs
- OD5: Avoid excessive object-chain knowledge
- OD6: Represent absence explicitly instead of vague `null` failures
- OD7: Replace repeated conditionals with a domain model, lookup, strategy, or exhaustive union dispatch

## Skill Routing

| Work | Use |
|------|-----|
| Broad duplication, intent, magic values, dead code | `clean-typescript-general` |
| Comments and TSDoc | `clean-typescript-comments` |
| Functions, arguments, mutation, flags | `clean-typescript-functions` |
| Modules, classes, file order, cohesion, coupling, dependency construction, over-abstraction | `clean-typescript-modules` |
| Promises, retries, timeouts, cancellation, race conditions | `clean-typescript-async` |
| Error handling and fallbacks | `clean-typescript-error-handling` |
| APIs, JSON, config, storage, SDKs | `clean-typescript-boundaries` |
| DTOs, domain models, classes, unions | `clean-typescript-objects-data` |
| Names | `clean-typescript-names` |
| Tests | `clean-typescript-tests` |

## TypeScript Precision
- Prefer `satisfies` when validating object literals without widening useful literal types.
- Avoid `as` assertions unless crossing a boundary that cannot express the type; narrow first when possible.
- Keep unsafe parsing at the edge of the system and pass typed values inward.

## Names (N1-N7)
- N1: Choose descriptive names
- N2: Right abstraction level
- N3: Use standard nomenclature
- N4: Unambiguous names
- N5: Name length matches scope
- N6: No encodings
- N7: Names describe side effects

## Tests (T1-T11)
- T1: Test everything that could break
- T2: Use coverage tools
- T3: Don't skip trivial tests
- T4: Ignored test = ambiguity question
- T5: Test boundary conditions
- T6: Exhaustively test near bugs
- T7: Look for patterns in failures
- T8: Check coverage when debugging
- T9: Unit tests should be fast; isolate slower integration tests
- T10: Prefer test data builders over brittle inline fixtures
- T11: Test behavior contracts, not incidental implementation details

## AI Behavior

When reviewing code, use this skill for the first-pass sweep: identify violations by rule number (e.g., "G1 violation: duplicated logic").

When writing detailed fixes or explanations for a specific category, invoke the corresponding sub-skill via the Skill tool before proceeding — for example, invoke `clean-typescript-functions` when addressing F-rule violations. The sub-skills contain code examples and nuance that this index omits.

When fixing or editing code, report what was fixed (e.g., "Fixed: extracted magic value to `SECONDS_PER_DAY` (G3)").
