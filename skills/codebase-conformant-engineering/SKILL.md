---
name: codebase-conformant-engineering
description: Enforces disciplined, repository-aware engineering behavior. Use this skill whenever modifying an existing codebase — before writing any code, when designing tests, when placing helper functions, and when deciding a task is complete. Prevents pattern violations, unnecessary abstractions, duplicated logic, misplaced helpers, narrow fixes, and premature completion claims.
---

# Codebase-Conformant Engineering

## Purpose

This skill exists because agents frequently produce code that *looks* plausible but
does not fit the project it lives in. The root causes are always the same: not
reading the repo carefully, inventing instead of reusing, optimizing for "make the
test pass," and declaring victory before acceptance criteria are actually met.

Follow every phase below **in order**. Do not skip phases to save time. The cost of
skipping is always higher than the cost of reading.

---

## Phase 1: Mandatory Codebase Investigation (BEFORE writing any code)

You may not write a single line of implementation code until you have completed
this phase and can answer every question below.

### 1.1 Read the relevant code — actually read it

- Locate and read the files you will modify, **plus** the files that call them,
  **plus** at least 2–3 files that solve *analogous* problems in the same repo.
- If the user references a "reference project," "existing implementation," or
  similar — **open it and read it first**. This is not optional. An explicit
  instruction to inspect something means inspect it before doing anything else.

### 1.2 Answer the pattern-discovery checklist

Write out (in your working notes or plan) answers to:

- [ ] How does this project already solve problems like this one? Cite specific files/functions.
- [ ] What architectural layers exist, and which layer does my change belong in?
- [ ] Is there an existing generic/shared utility that already does (or should do) what I need?
- [ ] How are tests structured here? Data-driven? Table-driven? Fixture-based? Generic harnesses?
- [ ] Where do helpers of this kind live? (e.g., `utils/`, `common/`, a base class, a shared module)
- [ ] What naming, error-handling, and dependency conventions does the surrounding code use?

**If you cannot answer these, you have not read enough. Go back and read more.**

### 1.3 Restate constraints

List every explicit constraint the user gave (e.g., "reuse the architecture",
"keep tests generic", "no domain-specific code in core", "verify with existing
tools"). You will check your work against this list in Phase 4. Treat these as
hard requirements, not suggestions.

---

## Phase 2: Design for Fit, Not for Novelty

### 2.1 Reuse before invent — the strict priority order

When you need functionality, resolve it in this order. Only fall through when
you have *verified* the previous option doesn't exist:

1. **Use** an existing function/class/pattern as-is.
2. **Extend** an existing generic mechanism (add a parameter, a config entry, a data row).
3. **Generalize** existing near-duplicate code into a shared utility (if the project's conventions support this).
4. **Create new code** — only with justification, and following the repo's established style exactly.

### 2.2 Prohibited moves

- ❌ Introducing a new abstraction, layer, wrapper, or pattern when an established one exists.
- ❌ Writing a domain-specific implementation of something the project handles generically.
- ❌ Copy-pasting or re-implementing logic that exists (or clearly belongs) in a shared module.
- ❌ Putting helper code in the module where you happen to be working, when the project has a designated home for such helpers. **Placement is determined by responsibility, not convenience.**

### 2.3 The generalization test

Before writing a fix or feature, ask: *"Does this solve the class of problem, or
only the instance in front of me?"*

- If the failing case is one instance of a broader behavior, fix the broader
  behavior. A fix that passes one scenario while leaving sibling scenarios broken
  is a defect, not a fix.
- If you deliberately scope a fix narrowly, say so explicitly and explain why.

---

## Phase 3: Tests That Match the Project

- **Mirror the existing test architecture.** If the framework supports
  data-driven / table-driven / parameterized tests, add *data*, not new
  bespoke test functions.
- **Never** write narrow, project-specific test scaffolding when a generic
  harness already covers the pattern. Adding a case should usually mean adding
  a row, fixture, or config entry.
- Test the *behavior*, not the specific input that happened to fail. Include
  boundary and sibling cases so the general solution is actually exercised.
- Use the project's existing test runners, linters, and verification tooling —
  the ones the user or repo docs specify — not ad-hoc verification scripts.

---

## Phase 4: Verification and Definition of Done

A task is **not complete** until all of the following are true. "It compiles and
one test passes" is never sufficient.

### 4.1 Acceptance criteria audit

- Re-read the original task and your Phase 1.3 constraint list.
- For **each** criterion and constraint, write down: *met / not met / partially
  met*, with concrete evidence (test output, file path, command result).
- If anything is "partially met" or unverified, the task is **not done**. Either
  finish it or report the gap explicitly. **Never silently skip a criterion.**

### 4.2 Conformance self-review

Before declaring completion, verify:

- [ ] My code follows the patterns identified in Phase 1 (cite the pattern it mirrors).
- [ ] I introduced zero unnecessary new abstractions.
- [ ] I duplicated zero existing functionality.
- [ ] Every helper lives in the architecturally correct module.
- [ ] Tests follow the project's existing test style (generic/data-driven where applicable).
- [ ] The fix addresses the general behavior, not just the reported instance.
- [ ] I ran the project's existing verification tools and they pass.
- [ ] Every explicit user instruction has been honored — checked one by one.

### 4.3 Honest reporting

- Report what you verified and *how* (exact commands, exact results).
- Explicitly state anything you did not do, could not verify, or interpreted
  ambiguously. A short "done, except X which needs Y" is vastly better than a
  false "done."

---

## Anti-Patterns Reference (what this skill exists to prevent)

| Anti-pattern | Required behavior instead |
|---|---|
| Ignoring existing project patterns | Discover and mirror them (Phase 1) |
| Inventing new abstractions | Reuse → extend → generalize → only then create (2.1) |
| Bespoke project-specific tests | Add data/cases to existing generic test patterns (Phase 3) |
| Helpers in the wrong module | Place by responsibility, per repo structure (2.2) |
| Duplicating generic functions | Search first; reuse or extend shared code (2.1) |
| Narrow "make it pass" fixes | Solve the class of problem; test siblings (2.3) |
| Superficial completion | Per-criterion audit with evidence (4.1) |
| Skimming the codebase | Answer the discovery checklist before coding (1.2) |
| Ignoring explicit instructions | Restate constraints up front, audit at the end (1.3, 4.2) |
| Plausible but non-compliant code | Conformance self-review naming the mirrored pattern (4.2) |

---

## Operating Principle

> **When in doubt, the repository is the authority.** Existing, working code in
> the project outweighs your general knowledge of "how this is usually done."
> Your job is to make changes that a longtime maintainer would recognize as
> native to the codebase — not to showcase an alternative approach.

If the repo's pattern seems genuinely wrong or the constraints conflict, **stop
and ask the user** rather than silently deviating.