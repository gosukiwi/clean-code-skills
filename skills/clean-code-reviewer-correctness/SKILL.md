---
name: clean-code-reviewer-correctness
description: Use when checking code for functional correctness, backwards compatibility, logic errors, security vulnerabilities, performance issues, or missing test coverage — not style.
---

# Clean Code Reviewer: Correctness

Reviews changed code for functional issues that break behavior, introduce vulnerabilities, degrade performance, or leave critical paths untested. This skill intentionally ignores style, naming, formatting, and structure — those belong to the style skills.

## Review Scope

Only flag issues in:
- Code that was changed in the diff
- Existing code that is now broken by the change (e.g., a caller that passes arguments the new signature no longer accepts)

Do not review unchanged code unless the change directly breaks it.

## Correctness

Look for:

- **Backwards compatibility**: Will existing callers break? Changed return types, removed exports, renamed public APIs, changed default parameter values, narrowed accepted input types, altered event shapes or callback signatures.
- **Logic errors**: Off-by-one, wrong comparison operator (`<` vs `<=`), inverted conditions, short-circuit evaluation that skips side effects, integer overflow in calculations.
- **Missing null/undefined checks**: Accessing properties on values that can be null, optional chaining that silently produces undefined where the caller expects a value.
- **Unhandled states**: Switch/if chains that miss a discriminant, async code that doesn't handle rejection, promises that can resolve to unexpected shapes.
- **Behavioral mismatch**: The code does something different from what the commit message, PR description, function name, or surrounding comments say it does.
- **Type narrowing gaps**: Assertions (`as`) that skip runtime validation, `unknown` that is never narrowed, generic constraints that are too wide for the actual usage.
- **Python typing gaps with runtime impact**: Untyped public contracts, unchecked `Any`/dict payloads, mutable default arguments, or casts that hide invalid external data.
- **Broken contracts**: Functions that violate their documented or implied contract — a `find` that can return `undefined` but the caller assumes it always finds, a sort comparator that isn't consistent.

## Security

Look for:

- **Injection**: SQL concatenation with user input, `innerHTML` or `dangerouslySetInnerHTML` with unsanitized data, shell command construction from user strings, unsafe `subprocess(..., shell=True)`, template literal injection in queries.
- **Exposed secrets**: API keys, tokens, passwords, or private keys hardcoded in source, committed `.env` files, secrets logged or included in error messages sent to clients.
- **Unsafe evaluation**: `eval()`, `exec()`, `new Function()`, `setTimeout(string)`, dynamic `import()` with user-controlled paths, or unsafe deserialization such as `pickle`/`yaml.load` on untrusted input.
- **Prototype pollution**: Object spread or assignment from unvalidated external input without freezing or allowlisting keys.
- **Insecure randomness**: `Math.random()` or Python `random` used for tokens, session IDs, or anything security-sensitive (should use cryptographic randomness such as Web Crypto, Node `crypto`, or Python `secrets`).
- **Missing authentication/authorization checks**: New endpoints or routes that skip auth middleware, privilege escalation paths where role checks are absent.

## Performance

Look for:

- **O(n²) or worse**: Nested loops over the same growing collection, `.find()` or `.includes()` inside `.filter()` or `.map()` over large arrays (use a Set or Map).
- **N+1 patterns**: Fetching related data inside a loop instead of batching, one query per item in a list.
- **Unbounded growth**: Arrays/lists, maps/dicts, or caches that grow without eviction, event listeners registered without cleanup, subscriptions that never unsubscribe.
- **Unnecessary re-renders** (React): Missing `key` props, unstable object/array/function references in dependency arrays or props, expensive computation in render without memoization when the component renders frequently.
- **Blocking the main thread/event loop**: Synchronous file I/O in async paths, heavy computation without yielding, `JSON.parse`/`json.loads` on unbounded input without a worker or streaming parser.
- **Missing pagination/limits**: Queries or API calls that fetch all records without limit, unbounded `SELECT *` on tables that grow.

## Test Coverage Risk

Look for:

- **Changed behavior without tests**: New code paths, modified conditionals, or altered business logic that lack corresponding test coverage.
- **Removed tests**: Tests deleted or disabled (`.skip`) that covered behavior which still exists in production code.
- **Untested error paths**: Catch blocks, fallback branches, retry logic, or timeout handling that has no test exercising it.
- **Critical paths without assertions**: Authentication, payment, data mutation, or permission checks that have no dedicated test.
- **Flaky test indicators**: Tests that depend on timing, global state, or execution order without isolation.

## Output Format

For each finding:

```text
Severity: Block | Fix | Suggest
Category: Correctness | Security | Performance | Test Coverage
Location: path/to/file:line
Issue: What is wrong and why it matters.
Suggested fix: Concrete next step.
```

## Severity Definitions

- **Block**: Will break production, cause data loss, introduce an exploitable vulnerability, or silently corrupt behavior for existing users. Examples: removed public export that other packages import, SQL injection with user input, infinite loop on a common code path.
- **Fix**: Likely bug or likely vulnerability that has a plausible path to production impact. Examples: off-by-one in pagination, missing null check on a value that is nullable in practice, `Math.random()` for a session token.
- **Suggest**: Potential issue that depends on context or is low-probability but worth the author's attention. Examples: possible performance issue on a collection that is currently small, a missing test for a non-critical path, a race condition that only manifests under high concurrency.

## AI Behavior

- Be precise. "This might have a bug" is not useful. Say what the bug is and under what input it triggers.
- Assume the author is competent. If something looks intentional but risky, ask rather than assert.
- Do not flag style issues. No opinions on naming, formatting, comment presence, abstraction level, or file structure.
- Do not flag theoretical issues that require impossible inputs. Focus on inputs that can actually reach the code.
- When flagging backwards compatibility, name the specific caller or contract that breaks.
- When flagging missing tests, be specific about what scenario is untested, not just "needs more tests."
- If the diff is large, prioritize: security > correctness > performance > test coverage.
