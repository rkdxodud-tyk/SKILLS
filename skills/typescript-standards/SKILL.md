---
name: typescript-standards
description: Enforces TypeScript best practices including strict typing, type-driven design, and modern idioms. Use when writing, reviewing, or refactoring TypeScript or JavaScript code, or configuring TS projects.
---

# TypeScript Standards

Apply these rules to all TypeScript code. Extend, don't replace, general SWE principles.

## Type System Rules

### Strictness
- Assume `strict: true`. Also prefer `noUncheckedIndexedAccess: true` and `exactOptionalPropertyTypes: true` in new projects.
- **Never use `any`.** Use `unknown` for truly unknown data, then narrow with type guards.
- No non-null assertions (`!`) except in tests or where invariant is documented one line above.
- No `as` casts to silence errors. If a cast is unavoidable, comment why it is safe. Prefer `satisfies` for shape-checking without widening.

### Type Design
- Make illegal states unrepresentable. Use discriminated unions instead of optional-field soup:

  ```ts
  // Bad
  type State = { loading?: boolean; data?: User; error?: Error };
  // Good
  type State =
    | { status: "loading" }
    | { status: "success"; data: User }
    | { status: "error"; error: Error };
  ```

- Prefer union literal types over enums: `type Role = "admin" | "editor" | "viewer"`.
- Use `readonly` on properties and `ReadonlyArray<T>` / `readonly T[]` for data that shouldn't mutate.
- Prefer `type` for unions/intersections/utilities; `interface` for object shapes meant to be extended or implemented.
- Derive types from values where possible: `as const` + `typeof` + `keyof` beats duplicating literals.
- Use branded types for IDs that must not be mixed: `type UserId = string & { __brand: "UserId" }`.

### Validation at Boundaries
- All external data (API responses, JSON.parse, env vars, form input) is `unknown` until validated.
- Use a schema validator (zod, valibot, etc. — match project's existing choice) at the boundary; infer static types from schemas to avoid drift.
- Never trust type assertions on fetched data: `res.json() as User` is a lie.

## Language Idioms

### Functions & Control Flow
- Prefer `const` everywhere; `let` only when reassignment is required; never `var`.
- Use optional chaining (`?.`) and nullish coalescing (`??`); never `||` for defaults where `0`, `""`, or `false` are valid values.
- Exhaustiveness-check switches on unions:

  ```ts
  default: {
    const _exhaustive: never = value;
    throw new Error(`Unhandled case: ${_exhaustive}`);
  }
  ```

- Prefer pure functions and immutable updates (spread, `toSorted`, `toReversed`) over mutation.

### Async
- No floating promises — every promise is awaited, returned, or explicitly `void`-ed with a comment.
- Use `Promise.all` / `Promise.allSettled` for independent async work; don't await sequentially in loops without reason.
- Handle rejections. `async` functions that can fail should either document what they throw or return a result type.
- `AbortController` for cancellable operations (fetches, long-running tasks).

### Modules & Project Layout
- Use ESM (`import`/`export`); no `require` in new code.
- Use `import type { ... }` for type-only imports.
- No barrel files (`index.ts` re-exporting everything) in large projects — they hurt tree-shaking and create circular imports.
- No circular dependencies. If two modules need each other, extract the shared piece.

## Error Handling
- Throw `Error` subclasses, never strings or plain objects.
- Catch clauses receive `unknown` — narrow before use:

  ```ts
  catch (err) {
    if (err instanceof ApiError) { ... }
    else throw err; // don't swallow unknowns
  }
  ```

- For expected failures in core logic, consider `Result`-style returns (`{ ok: true, value } | { ok: false, error }`) over exceptions.

## Testing
- Type tests matter: use `expectTypeOf`/`assertType` (vitest) or `tsd` for complex generics and public API types.
- Mock at module boundaries, not deep internals. Prefer dependency injection over module mocking.
- Match the project's test runner; do not introduce a second one.

## Anti-Patterns to Reject
- `any` (including implicit via untyped deps), `@ts-ignore` (use `@ts-expect-error` with a reason if truly needed)
- Classes for stateless logic — plain functions and modules suffice
- Overly clever generic gymnastics that no one can read — a small amount of duplication beats an unreadable type
- Default exports in libraries (breaks refactoring and auto-imports); prefer named exports
- Mutating function parameters
