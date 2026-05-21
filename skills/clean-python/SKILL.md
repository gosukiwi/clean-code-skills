---
name: clean-python
description: Use when writing, fixing, editing, or refactoring Python code, especially weak type hints, unclear names, duplicated logic, oversized functions, stale comments, boundary gaps, exception handling, data modeling, async flows, module structure, or brittle tests.
---

# Clean Python: Index And Review Guide

Use this as the Python entry point. It routes broad review work to focused skills and summarizes the rules most likely to affect design. Prefer explicit runtime behavior, typed public contracts, and simple module boundaries over clever language tricks.

## Comments (C1-C5)
- C1: No metadata in comments (use Git)
- C2: Delete obsolete comments immediately
- C3: No redundant comments
- C4: Write comments well if you must
- C5: Never commit commented-out code

## Error Handling (EH1-EH4)
- EH1: Raise useful exception objects with context
- EH2: Catch specific exceptions; avoid bare `except`
- EH3: Do not swallow failures
- EH4: Use typed recoverable results only when the project style or domain warrants it

## Functions (F1-F9)
- F1: Maximum 3 arguments unless the signature is a stable framework/API contract; use dataclasses or parameter objects for cohesive inputs
- F2: No output arguments (return values)
- F3: No flag arguments (split functions)
- F4: Delete dead functions
- F5: Reduce nesting with guard clauses, helper functions, extraction, or refactoring
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

## Python-Specific (PY1-PY5)
- PY1: Type public functions, class attributes, and boundary values; avoid untyped public APIs
- PY2: Use dataclasses, `TypedDict`, `Protocol`, enums, or literal unions to express real shapes and contracts
- PY3: Prefer context managers for resources; never rely on implicit cleanup
- PY4: Avoid mutable default arguments and hidden shared state
- PY5: Keep dynamic features (`getattr`, monkeypatching, reflection, `Any`, metaclasses) behind clear boundaries

## Boundaries (B1-B4)
- B1: Treat external data as untrusted until parsed and validated
- B2: Convert API, JSON, env, storage, database, and SDK shapes into internal types at the edge
- B3: Keep vendor types out of domain code unless they are the domain
- B4: Test tricky boundary mappings and invalid external shapes

## Objects and Data (OD1-OD7)
- OD1: Use plain data for transfer, rendering, serialization, and pattern matching
- OD2: Use objects/classes when behavior and invariants belong together
- OD3: Model impossible states out with explicit variants, enums, dataclasses, or typed unions
- OD4: Keep DTOs separate from domain models when external shape differs
- OD5: Avoid excessive object-chain knowledge
- OD6: Represent absence explicitly instead of vague `None` failures
- OD7: Replace repeated conditionals with a domain model, lookup, strategy, or exhaustive dispatch

## Skill Routing

| Work | Use |
|------|-----|
| Broad duplication, intent, magic values, dead code | `clean-general` |
| Comments and docstrings | `clean-general-comments` |
| Functions, arguments, mutation, flags | `clean-python-functions` |
| Modules, classes, file order, cohesion, coupling, dependency construction, over-abstraction | `clean-python-modules` |
| Coroutines, tasks, retries, timeouts, cancellation, race conditions | `clean-python-async` |
| Error handling and fallbacks | `clean-python-error-handling` |
| APIs, JSON, config, storage, databases, SDKs | `clean-python-boundaries` |
| DTOs, dataclasses, domain models, enums, typed unions | `clean-python-objects-data` |
| Names | `clean-general-names` |
| Tests | `clean-python-tests` |

## Rule Precedence (When Findings Overlap)

A single code region often matches multiple rules. Without precedence, the same issue gets reported 2-4 times under different IDs and the review becomes noise.

**Core rule:** If two findings have the **same Fix**, collapse to one finding under the most specific rule and cite the others in parentheses. If two findings have **different Fixes**, keep them separate even when they touch the same line.

**Specificity order — the more specific rule wins:**

- **B1-B4 (Boundaries) > PY1 / G2 / OD1** at the system edge. Unvalidated JSON, request data, env vars, or database rows crossing into domain code are **B1**, not "add type hints (PY1)" or "obscured intent (G2)".
- **M5 (empty abstraction) > M8 / G1** for pass-through wrappers. A module that re-exports or wraps without adding behavior is **M5**; the wide export surface (M8) and duplicated wrapper (G1) are downstream effects of the same mistake.
- **F-rules > G-rules** inside a function. A dead helper is **F4** (with G5 in parens), not the other way around. A function doing two things via a flag is **F3**, not G4.
- **F2 (no output arguments) > A3 (no shared mutable state across awaits)** for direct parameter mutation. A3 is reserved for genuine cross-call races: module-level mutable values read and written across awaits by concurrent callers.
- **A3 > PY4** for hidden shared state that crosses `await` boundaries. Cite **PY4** for the hidden state smell only when there is no async race-specific finding.
- **PY4 > F1/G2** for mutable defaults. The bug risk from `items=[]` or `options={}` is the primary finding.
- **G1 > G3** when a named constant exists but a literal is used at another call site.

**Rules that look similar but stay independent — report both:**

- **B1 vs B2/OD4** — B1 is "validate external input"; B2/OD4 is "separate DTOs from domain models". Different fixes.
- **EH3 vs EH1** — EH3 is structural (no catch / swallowed error); EH1 is informational (caught but no context). Different fixes.

## Python Precision

- Prefer concrete type hints at public boundaries and useful inference inside small private helpers.
- Avoid `Any`, broad `dict[str, Any]`, and unchecked casts unless crossing a boundary that cannot express the type.
- Use `Protocol` for behavioral dependencies and dataclasses or `TypedDict` for plain data shapes.
- Keep parsing, validation, database row mapping, and SDK adaptation at the edge of the system.
- Use `with`/`async with` for files, locks, connections, sessions, and transactions.

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

When reviewing code, use this skill for the first-pass sweep: identify violations by rule number (e.g., "PY4 violation: mutable default argument").

When writing detailed fixes or explanations for a specific category, read the corresponding sub-skill file before proceeding — for example, read `../clean-python-functions/SKILL.md` when addressing F-rule violations. If the Skill tool is available, invoke the skill by name instead. The sub-skills contain code examples and nuance that this index omits.

When fixing or editing code, report what was fixed (e.g., "Fixed: replaced mutable default with `None` and local list allocation (PY4)").
