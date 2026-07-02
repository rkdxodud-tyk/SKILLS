---
name: maintenance-programming
description: Apply disciplined maintenance programming principles when modifying existing codebases. Use this skill whenever working in an established codebase — fixing bugs, extending features, patching legacy code, or making changes to production systems. Ensures minimal, safe, reversible changes that respect existing conventions.
---

# Maintenance Programming Principles

You are working in an **existing, living codebase**. Your primary duty is to
leave the system no worse than you found it, and ideally slightly better —
without introducing risk, churn, or surprise. Follow these principles for
every change.

## Core Mindset

1. **First, do no harm.** A maintenance change that breaks something
   unrelated is worse than no change at all.
2. **The codebase is the documentation of decisions.** Assume existing code
   works and is the way it is for a reason, until proven otherwise.
3. **You are a guest, not an architect.** Do not impose your preferred
   style, patterns, or libraries onto code that already has its own.

## Before Changing Anything

### Understand before you edit
- Read the surrounding code, not just the lines you're changing. Read the
  full function, its callers, and its tests before modifying it.
- Search for other usages of any function, constant, or type you touch
  (`grep`/glob for the symbol across the repo). Assume every public symbol
  has callers you haven't seen.
- Check for existing tests covering the area. Run them **before** making
  changes to establish a known-good baseline.
- Look at git history/blame when behavior seems strange — the commit message
  often explains the "why."

### Chesterton's Fence
- Never delete or "simplify" code you don't understand. If code looks
  useless, find out why it exists before removing it.
- If you cannot determine why something exists, leave it and flag it in
  your summary rather than removing it.

### Reproduce before you fix
- For bug fixes: reproduce the bug first (via a failing test if possible).
  A fix without a reproduction is a guess.
- Fix the **root cause**, not the symptom. If a fix requires touching the
  root cause in a risky way, apply the safe symptomatic fix, and clearly
  document the root cause for follow-up.

## Making the Change

### Minimal diff principle
- Make the **smallest change that correctly solves the problem**.
- Do not reformat, rename, reorder imports, or restyle code you aren't
  functionally changing. Unrelated diff noise makes review harder and
  blame useless.
- One logical change per commit/PR. If you discover a second problem,
  note it — don't fix it inline unless it blocks your task.

### Match existing conventions
- Mirror the file's existing style: naming, error handling, logging
  patterns, test structure, directory layout — even if you'd choose
  differently in a greenfield project.
- Use libraries and utilities already present in the codebase. Do **not**
  add new dependencies for problems the existing stack can solve.
- If the codebase has a helper/util for something, use it instead of
  reimplementing.

### Preserve contracts and behavior
- Preserve public APIs, function signatures, return types, error types,
  serialization formats, and observable behavior unless the change
  explicitly requires breaking them.
- If a breaking change is unavoidable, call it out explicitly and update
  all call sites in the same change.
- Watch for implicit contracts: log formats consumed by monitoring,
  ordering that callers depend on, timing/performance characteristics,
  null-handling quirks that callers work around.

### No opportunistic refactoring
- Do not refactor "while you're in there" unless:
  1. The refactor is required to make the fix safely, or
  2. The user explicitly asked for it.
- If you see refactoring opportunities, list them in your final summary
  as suggestions instead of doing them.

### Defensive changes
- Prefer additive changes (new function, new optional parameter, new code
  path behind a condition) over modifying shared code paths.
- When modifying shared code, enumerate all callers and verify each one
  still behaves correctly.
- Keep changes easy to revert: avoid mixing migrations, config changes,
  and logic changes in one commit when they can be separated.

## Testing and Verification

- **Write or update a test for every behavioral change.** For bug fixes,
  the test must fail before the fix and pass after.
- Run the existing test suite (or the relevant subset) after your change.
  Do not declare a task done with failing tests.
- Never "fix" a failing test by weakening its assertions to match broken
  behavior — determine which is wrong: the test or the code.
- Do not delete or skip tests to make a change pass. If a test is truly
  obsolete, explain why in your summary.
- Manually trace edge cases the tests don't cover: nulls, empty
  collections, concurrency, error paths.

## Legacy Code Specifics

- If code has no tests, add a **characterization test** (capturing current
  behavior, even if imperfect) before modifying it, when feasible.
- Prefer "sprout" techniques: put new logic in a new, well-tested function
  and call it from the legacy code, rather than surgery inside a large
  untested function.
- Treat commented-out code, dead flags, and odd workarounds as historical
  evidence. Investigate before disturbing.

## Communication and Traceability

- Write commit messages / summaries that explain **why**, not just what:
  the problem, the root cause, the fix, and the risk/rollback plan.
- Explicitly list in your final summary:
  - What files changed and why
  - Any behavior changes callers might notice
  - Anything you found suspicious but deliberately did not touch
  - Suggested follow-ups (refactors, missing tests, tech debt)
- Update comments and docs that your change makes stale. Do not add
  redundant comments restating the code.

## Hard Rules (never violate)

- ❌ Never rewrite a module wholesale when a targeted fix suffices.
- ❌ Never change code style/formatting across files you aren't modifying.
- ❌ Never remove error handling, retries, timeouts, or validation because
  they "seem unnecessary."
- ❌ Never introduce a new framework, language version, or dependency
  without explicit approval.
- ❌ Never silently change behavior that external systems may depend on.
- ❌ Never mark a task complete without running the relevant tests.

## Decision Checklist (run before finalizing)

- [ ] Did I read the callers of everything I changed?
- [ ] Is this the smallest correct diff?
- [ ] Does the change match existing conventions?
- [ ] Are public contracts and observable behavior preserved?
- [ ] Is there a test that proves the fix / covers the new behavior?
- [ ] Did the full relevant test suite pass?
- [ ] Is the change easy to review and easy to revert?
- [ ] Did I document anything I found but deliberately left alone?