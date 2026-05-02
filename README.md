# Clean Code Skills for AI Agents

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-Compatible-blue)](https://agentskills.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Agent Skills for writing cleaner TypeScript and React. The skills adapt Robert C. Martin's Clean Code rules for modern TypeScript projects and add React-specific guidance for components, hooks, state, and UI tests.

## What's Included

| Track | Skill | Description | Rules |
|-------|-------|-------------|-------|
| TypeScript | `boy-scout` | Orchestrator for small cleanup while editing existing TypeScript | Coordinates TypeScript skills |
| TypeScript | `typescript-clean-code` | Master TypeScript clean-code reference | C1-C5, E1-E2, F1-F4, G1-G36, N1-N7, TS1-TS3, T1-T9 |
| TypeScript | `clean-comments` | Minimal, accurate comments and TSDoc | C1-C5 |
| TypeScript | `clean-functions` | Focused functions with clear call sites | F1-F4 |
| TypeScript | `clean-general` | DRY, intent, boundaries, coupling, and abstractions | G5, G16, G23, G25, G30, G36 |
| TypeScript | `clean-names` | Descriptive and unambiguous names | N1-N7 |
| TypeScript | `clean-tests` | Fast, boundary-aware TypeScript tests | T1-T9 |
| React | `react-clean-code` | Master React clean-code reference | R1-R10 |
| React | `clean-components` | Component boundaries, props, JSX, and composition | R1-R3 |
| React | `clean-hooks` | Effects, dependencies, custom hooks, refs, and memoization | R4, R8 |
| React | `clean-state` | Local state, derived state, reducers, context, and server state | R5-R7 |
| React | `clean-react-tests` | React Testing Library and user-facing component tests | R10 |

Use the master skills for comprehensive coverage or individual skills for targeted enforcement.

## Installation

Copy the skills you want into your agent's skills directory. For React projects, install both TypeScript and React tracks.

## Ask your agent

The best way is to just open up your coding agent, such as Codex, and type:

    What's the best way to install these skills? https://github.com/gosukiwi/clean-code-react


## Manual

This will depend on the tool that you have but you can always just copy the
skills into `.agent/skills` and it should work. It can be project-level or
user-level.

```bash
# Project-level
mkdir -p .agent/skills
cp -r skills/typescript/* .agent/skills/
cp -r skills/react/* .agent/skills/
```

## Usage

Once installed, skills activate automatically based on context, but depending
on your agent you can call them manually with `/` or `$`.

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

### Functions (F1-F4)

| Rule | Principle |
|------|-----------|
| F1 | Too many arguments are a design smell |
| F2 | No output arguments |
| F3 | No flag arguments for behavior selection |
| F4 | Delete dead functions |

### General (G1-G36)

| Rule | Principle |
|------|-----------|
| G5 | DRY: no duplicated knowledge |
| G16 | No obscured intent |
| G23 | Prefer polymorphism, exhaustive dispatch, or data tables over growing conditionals |
| G25 | Named constants for domain values |
| G30 | Functions do one thing |
| G36 | Avoid excessive object-chain knowledge |

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
| T9 | Keep tests fast |

### React (R1-R10)

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
