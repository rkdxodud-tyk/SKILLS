---
name: swe-principles
description: Enforces general software engineering best practices across all languages. Use when writing, reviewing, or refactoring any code — covers naming, function design, error handling, testing, comments, and code organization.
---

# General Software Engineering Principles

Apply these principles to ALL code you write or modify, regardless of language.

## Core Rules

### Simplicity First (KISS / YAGNI)
- Write the simplest code that solves the problem. Do not add abstractions, config options, or generalization for hypothetical future needs.
- Prefer boring, obvious solutions over clever ones. Code is read far more often than it is written.
- Three strikes rule: only extract an abstraction after the third duplication, not the first.

### Single Responsibility
- Each function does ONE thing. If you describe a function with "and", split it.
- Keep functions short — aim for under ~30 lines. Extract when logic has distinct phases.
- Modules/files group cohesive functionality. Avoid "utils" dumping grounds; name modules by domain.

### Naming
- Names must reveal intent: `daysSinceLastLogin` not `d`, `isEligibleForDiscount` not `check`.
- Booleans read as predicates: `isReady`, `hasPermission`, `canRetry`.
- Functions are verbs (`fetchUser`, `validateInput`); values are nouns.
- No abbreviations unless universally known (`id`, `url`, `db` are fine; `usrMgr` is not).
- Rename when a name stops matching reality — never let names lie.

### Error Handling
- Never silently swallow errors. Every catch/ignore must be justified with a comment.
- Fail fast: validate inputs at boundaries (API handlers, file parsers, user input), then trust data internally.
- Error messages must include context: what failed, with what input, and what the caller can do.
- Distinguish expected failures (return/result values) from bugs (exceptions/panics).

### Comments & Documentation
- Comments explain WHY, not WHAT. If the code needs a "what" comment, rewrite the code instead.
- Document invariants, non-obvious constraints, and workarounds (with links to issues if applicable).
- Delete commented-out code. Version control remembers it.
- Keep docs next to code. Update docs in the same change that alters behavior.

### Testing
- Every bug fix gets a regression test that fails without the fix.
- Test behavior, not implementation — tests should survive refactors.
- Cover: happy path, edge cases (empty, zero, max, unicode), and error paths.
- Tests must be deterministic. No sleeps, no real network, no shared mutable state between tests.
- Keep test names descriptive: `rejects_expired_token`, not `test3`.

### Dependencies & Boundaries
- Prefer standard library over a dependency; prefer a small dependency over a framework.
- Isolate I/O (network, disk, clock, randomness) behind interfaces so core logic is pure and testable.
- Do not leak third-party types across module boundaries unless intentional.

### Refactoring Discipline
- Never mix refactoring with behavior changes in the same commit.
- Leave code cleaner than you found it, but keep unrelated cleanup out of feature diffs.
- When modifying existing code, match its established style and conventions first.

## When Reviewing Code

Check in this order:
1. Correctness — does it handle edge cases and errors?
2. Security — injection, secrets in code, unvalidated input?
3. Clarity — will someone understand this in 6 months?
4. Tests — is the new behavior covered?
5. Style — only flag if it violates project conventions.

## Anti-Patterns to Reject

- God functions/classes doing everything
- Deep nesting (>3 levels) — use early returns and guard clauses
- Magic numbers/strings — extract named constants
- Mutable global state
- Copy-paste with small edits — parameterize instead
- Premature optimization without profiling data
