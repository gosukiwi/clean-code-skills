---
name: typescript-clean-code
description: Use when writing, fixing, editing, reviewing, or refactoring TypeScript code, especially weak types, unclear names, duplicated logic, oversized functions, stale comments, boundary gaps, error handling, data modeling, or brittle tests.
---

# Clean TypeScript: Index And Review Guide

Use this as the TypeScript entry point. It routes broad review work to focused skills and summarizes the rules most likely to affect design. Prefer clear runtime behavior and precise public types over cleverness or type-system escape hatches.

## Comments (C1-C5)
- C1: No metadata in comments (use Git)
- C2: Delete obsolete comments immediately
- C3: No redundant comments
- C4: Write comments well if you must
- C5: Never commit commented-out code

## Environment (E1-E2)
- E1: One command to build (`npm run build`)
- E2: One command to test (`npm test`)

## Error Handling
- EH1: Throw `Error` objects with useful context
- EH2: Catch `unknown` and narrow before reading properties
- EH3: Do not swallow failures
- EH4: Use typed recoverable results only when the project style or domain warrants it

## Functions (F1-F4)
- F1: Maximum 3 arguments (use parameter objects/interfaces for more)
- F2: No output arguments (return values)
- F3: No flag arguments (split functions)
- F4: Delete dead functions

## General
- G5: DRY - no duplication
- G9: Delete dead code
- G16: No obscured intent
- G25: Named constants, not magic numbers
- G30: Functions do one thing

## TypeScript-Specific (TS1-TS3)
These adapt the Java-specific rules (J1-J3) to TypeScript conventions:
- TS1: Keep imports explicit and stable; avoid namespace-style overuse and implicit dependencies
- TS2: Prefer literal unions, discriminated unions, or const objects for closed sets; use enums only when they match project convention
- TS3: Type public interfaces explicitly; use `unknown` plus narrowing instead of `any` at boundaries

## Boundaries
- B1: Treat external data as `unknown` until validated
- B2: Convert API, JSON, env, storage, database, and SDK shapes into internal types at the edge
- B3: Keep vendor types out of domain code unless they are the domain
- B4: Test tricky boundary mappings and invalid external shapes

## Objects and Data
- OD1: Use plain data for transfer, rendering, serialization, and pattern matching
- OD2: Use objects/classes when behavior and invariants belong together
- OD3: Model impossible states out with discriminated unions
- OD4: Keep DTOs separate from domain models when external shape differs
- OD5: Avoid excessive object-chain knowledge

## Skill Routing

| Work | Use |
|------|-----|
| Broad duplication, intent, magic values, dead code | `clean-general` |
| Comments and TSDoc | `clean-comments` |
| Functions, arguments, mutation, flags | `clean-functions` |
| Error handling and fallbacks | `clean-error-handling` |
| APIs, JSON, config, storage, SDKs | `clean-boundaries` |
| DTOs, domain models, classes, unions | `clean-objects-data` |
| Names | `clean-names` |
| Tests | `clean-tests` |

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

## Tests (T1-T9)
- T1: Test everything that could break
- T2: Use coverage tools
- T3: Don't skip trivial tests
- T4: Ignored test = ambiguity question
- T5: Test boundary conditions
- T6: Exhaustively test near bugs
- T7: Look for patterns in failures
- T8: Check coverage when debugging
- T9: Unit tests should be fast; isolate slower integration tests

## Quick Reference Table

| Category | Rule | One-Liner |
|----------|------|-----------|
| **Comments** | C1 | No metadata (use Git) |
| | C3 | No redundant comments |
| | C5 | No commented-out code |
| **Functions** | F1 | Max 3 arguments |
| | F3 | No flag arguments |
| | F4 | Delete dead functions |
| **Errors** | EH1 | Throw useful `Error` objects |
| | EH3 | Do not swallow failures |
| **General** | G5 | DRY - no duplication |
| | G9 | Delete dead code |
| | G16 | No obscured intent |
| | G25 | Named constants, not magic numbers |
| | G30 | Functions do one thing |
| **Boundaries** | B1 | Validate external data |
| | B3 | Hide vendor types |
| **Objects/Data** | OD3 | Model impossible states out |
| | OD5 | Avoid object-chain coupling |
| **Names** | N1 | Descriptive names |
| | N5 | Name length matches scope |
| **Tests** | T5 | Test boundary conditions |
| | T9 | Unit tests fast; slower tests isolated |

## Anti-Patterns (Don't -> Do)

| Don't | Do |
|----------|-------|
| Comment every line | Delete obvious comments |
| Helper for one-liner | Inline the code |
| `import * as utils` everywhere | Named imports for explicit dependencies |
| `any` in public API | Specific types or `unknown` + narrowing |
| `value as User` after parsing JSON | Parse, validate, then return `User` |
| Empty `catch` block | Handle, add context, or throw a contextual `Error` |
| One type for API payload and domain model | Boundary DTO plus domain type |
| Magic number `86400` | `const SECONDS_PER_DAY = 86400` |
| `process(data, true)` | `processVerbose(data)` |
| Deep nesting | Guard clauses, early returns |
| Reaching through `obj.a.b.c.value` | Ask the owner object for the value |
| 100+ line function | Split by responsibility |

## AI Behavior

When reviewing code, identify violations by rule number (e.g., "G5 violation: duplicated logic").
When fixing or editing code, report what was fixed (e.g., "Fixed: extracted magic number to `SECONDS_PER_DAY` (G25)").
