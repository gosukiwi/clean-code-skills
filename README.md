# Clean Code Skills for AI Agents

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-Compatible-blue)](https://agentskills.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Agent Skills for writing cleaner TypeScript, React, and CSS. The skills adapt
Robert C. Martin's Clean Code rules for modern TypeScript projects and add
focused guidance for React components, hooks, state, UI tests, and
maintainable styling.

## What's Included

| Track | Skill | Description | Rules |
|-------|-------|-------------|-------|
| All | `clean-code` | Top-level router for clean-code review across TypeScript, React, and CSS | Routes to relevant skills |
| TypeScript | `clean-as-you-go` | Orchestrator for small cleanup while editing existing TypeScript | Coordinates TypeScript skills |
| TypeScript | `clean-typescript` | Master TypeScript index and review guide | C1-C5, E1-E2, EH1-EH4, F1-F5, G1-G5, B1-B4, OD1-OD5, N1-N7, TS1-TS3, T1-T9 |
| TypeScript | `clean-typescript-comments` | Minimal, accurate comments and TSDoc | C1-C5 |
| TypeScript | `clean-typescript-functions` | Focused functions with clear call sites | F1-F5 |
| TypeScript | `clean-typescript-error-handling` | Exceptions, catch blocks, recoverable failures, and fallbacks | EH1-EH4 |
| TypeScript | `clean-typescript-boundaries` | APIs, JSON, env vars, storage, databases, SDKs, and external data | B1-B4 |
| TypeScript | `clean-typescript-objects-data` | DTOs, domain models, discriminated unions, and object boundaries | OD1-OD5 |
| TypeScript | `clean-typescript-general` | Duplication, intent, magic values, single responsibility, and dead code | G1-G5 |
| TypeScript | `clean-typescript-names` | Descriptive and unambiguous names | N1-N7 |
| TypeScript | `clean-typescript-tests` | Fast, boundary-aware TypeScript tests | T1-T9 |
| React | `clean-react` | Master React index and review guide | R1-R11 |
| React | `clean-react-components` | Component boundaries, props, JSX, and composition | R1-R3 |
| React | `clean-react-hooks` | Effects, dependencies, custom hooks, refs, and memoization | R4, R8 |
| React | `clean-react-state` | Local state, derived state, reducers, context, and server state | R5-R7 |
| React | `clean-react-tests` | React Testing Library and user-facing component tests | R10 |
| React | `clean-react-file-organization` | Owner folders, private dependencies, common folders, and CSS modules | R11 |
| CSS | `clean-css` | Tokens, layout, selectors, responsive behavior, and visual accessibility | CSS1-CSS8 |

Use `clean-code` for broad review across the full collection. Use the track
master skills for TypeScript or React review and individual skills for
targeted guidance.

## Installation

Skills are designed to be used together. The recommended way to install them
is with the install script. From this repo, run:

```bash
bin/install /path/to/your-project
```

The installer supports `.codex`, `.agents`, and `.claude` project directories.
If none exist, it prompts you to choose which one to create.

## Ask your agent

Alternatively, just open up your coding agent (e.g. Codex), and type:

    What's the best way to install these skills? https://github.com/gosukiwi/clean-code-react

## Manual

This will depend on the tool that you have but you can always just copy the
skills into `.codex/skills`, `.agents/skills`, or `.claude/skills`. It can be
project-level or user-level.

```bash
# Project-level
mkdir -p .codex/skills
cp -r skills/clean-code .codex/skills/
cp -r skills/typescript/* .codex/skills/
cp -r skills/react/* .codex/skills/
cp -r skills/css/* .codex/skills/
```

## Usage

Once installed, skills activate automatically based on context, but depending
on your agent you can call them manually with `/` or `$`.

For broad review, ask for `clean-code`, for example: "Review this using clean
code." The `clean-code` skill routes to every relevant TypeScript, React, and
CSS skill.

## Example

Before:

```tsx
function UserPanel({ u, admin, compact, loading, err }: any) {
  const [name, setName] = useState("");

  useEffect(() => {
    setName(u.first + " " + u.last);
  }, [u]);

  if (loading) return <div>Loading</div>;
  if (err) return <div>Error</div>;

  return (
    <div>
      <h2>{name}</h2>
      {admin ? <button>Delete</button> : null}
      {!compact ? <p>{u.email}</p> : null}
    </div>
  );
}
```

After:

```tsx
type User = {
  firstName: string;
  lastName: string;
  email: string;
};

type UserPanelProps = {
  user: User;
  showAdminActions?: boolean;
  variant?: "compact" | "full";
};

function UserPanel({
  user,
  showAdminActions = false,
  variant = "full",
}: UserPanelProps) {
  const fullName = `${user.firstName} ${user.lastName}`;

  return (
    <section aria-labelledby="user-panel-title">
      <h2 id="user-panel-title">{fullName}</h2>
      {variant === "full" ? <p>{user.email}</p> : null}
      {showAdminActions ? <UserAdminActions user={user} /> : null}
    </section>
  );
}
```

What improved:

- Public props are explicitly typed.
- Derived state moved out of `useEffect`.
- Names describe the domain.
- Rendering modes are explicit.
- Admin actions have their own component boundary.

## Rule Reference

### Comments (C1-C5)

| Rule | Principle |
|------|-----------|
| C1 | No metadata in comments |
| C2 | Delete obsolete comments |
| C3 | No redundant comments |
| C4 | Write comments well if needed |
| C5 | Never commit commented-out code |

### Functions (F1-F5)

| Rule | Principle |
|------|-----------|
| F1 | Too many arguments are a design smell |
| F2 | No output arguments |
| F3 | No flag arguments for behavior selection |
| F4 | Delete dead functions |
| F5 | Reduce nesting with early returns, helper functions, extraction, or refactoring |

### General

| Rule | Principle |
|------|-----------|
| G1 | No duplicated knowledge |
| G2 | No obscured intent |
| G3 | Named constants for domain values |
| G4 | Functions do one thing |
| G5 | Delete dead code |

### Error Handling (EH1-EH4)

| Rule | Principle |
|------|-----------|
| EH1 | Throw `Error` objects with useful context |
| EH2 | Catch `unknown` and narrow before reading properties |
| EH3 | Do not swallow failures |
| EH4 | Use typed recoverable results only when project style or domain warrants it |

### Boundaries (B1-B4)

| Rule | Principle |
|------|-----------|
| B1 | Treat external data as `unknown` until validated |
| B2 | Convert external shapes into internal types at the edge |
| B3 | Keep vendor types out of domain code unless they are the domain |
| B4 | Test tricky boundary mappings and invalid external shapes |

### Objects And Data (OD1-OD5)

| Rule | Principle |
|------|-----------|
| OD1 | Use plain data for transfer, rendering, serialization, and pattern matching |
| OD2 | Use objects/classes when behavior and invariants belong together |
| OD3 | Model impossible states out with discriminated unions |
| OD4 | Keep DTOs separate from domain models when external shape differs |
| OD5 | Avoid excessive object-chain knowledge |

### Names (N1-N7)

| Rule | Principle |
|------|-----------|
| N1 | Choose descriptive names |
| N2 | Names match abstraction level |
| N3 | Use standard nomenclature |
| N4 | Use unambiguous names |
| N5 | Name length matches scope |
| N6 | Avoid encodings |
| N7 | Names describe side effects |

### TypeScript (TS1-TS3)

| Rule | Principle |
|------|-----------|
| TS1 | Keep imports explicit and stable |
| TS2 | Prefer literal unions, discriminated unions, or const objects for closed sets |
| TS3 | Type public interfaces and avoid `any` at boundaries |

### Tests (T1-T9)

| Rule | Principle |
|------|-----------|
| T1 | Test behavior that could break |
| T2 | Use coverage tools as a guide |
| T3 | Do not skip trivial behavior |
| T4 | Skipped tests must expose a real ambiguity or external constraint |
| T5 | Test boundary conditions |
| T6 | Test near bugs exhaustively |
| T7 | Look for patterns in failures |
| T8 | Let coverage gaps reveal design issues |
| T9 | Keep unit tests fast and isolate slower integration tests |

### React (R1-R11)

| Rule | Principle |
|------|-----------|
| R1 | Components should have one reason to change |
| R2 | Prefer composition over boolean prop combinations |
| R3 | Keep JSX shallow enough to scan |
| R4 | Effects synchronize with external systems |
| R5 | Derive values during render when possible |
| R6 | Keep state as local as possible |
| R7 | Separate server state from local UI state |
| R8 | Custom hooks own reusable stateful behavior |
| R9 | Event handlers should describe user intent |
| R10 | React tests should assert user-visible behavior |
| R11 | Organize files by dependency ownership |

### CSS (CSS1-CSS8)

| Rule | Principle |
|------|-----------|
| CSS1 | Use tokens for repeated design values |
| CSS2 | Keep declarations purposeful |
| CSS3 | Prefer layout primitives over manual spacing and positioning |
| CSS4 | Keep selectors shallow and specificity low |
| CSS5 | Preserve focus, contrast, reduced motion, and states |
| CSS6 | Avoid static inline styles |
| CSS7 | Keep responsive behavior explicit and stable |
| CSS8 | Avoid duplicated style knowledge |
