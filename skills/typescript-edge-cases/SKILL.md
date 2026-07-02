---
name: typescript-edge-cases
description: Handle TypeScript/JavaScript edge cases around type narrowing, null vs undefined, numeric quirks, async pitfalls, and unsound type escape hatches. Use when writing or reviewing TypeScript code.
---

# TypeScript Edge Cases

TypeScript's types erase at runtime and the type system is deliberately unsound
in places. Apply on every TS task.

## The type system lies — know where

- **`as` assertions bypass checking.** Prefer type guards, `satisfies`, or
  schema validation. Every `as` (except `as const`) needs justification.
- **External data is untyped.** `JSON.parse`, `fetch().json()`, `localStorage`
  return `any`-ish data. Validate at the boundary with zod/valibot or manual
  guards — never just annotate `const data: User = await res.json()`.
- **Array index access is optimistic.** `arr[i]` is typed `T` but may be
  `undefined`. Enable `noUncheckedIndexedAccess` and handle it.
- Records lie the same way: `Record<string, T>` lookups can be `undefined`.
- Variance holes: mutable arrays are covariant (unsound); method-syntax params
  are bivariant. Writing to a widened array can corrupt types silently.
- `Object.keys(obj)` returns `string[]`, not `keyof typeof obj` — for a reason
  (excess properties exist at runtime). Don't blindly cast it back.

## null, undefined, and falsy traps

- `strict: true` always (includes `strictNullChecks`).
- `??` vs `||`: `value || fallback` swallows `0`, `""`, `false`. Default to `??`.
- Optional chaining short-circuits to `undefined`, never `null` — but the
  property itself might be `null`. `a?.b` can still be `null`.
- `!` (non-null assertion) is `as` in disguise; justify or replace with a check.
- Optional property (`x?: T`) vs `x: T | undefined` differ under
  `exactOptionalPropertyTypes` — decide which semantics you mean.

## Numbers

- All numbers are float64: integers safe only up to `Number.MAX_SAFE_INTEGER`
  (2^53 − 1). IDs from databases can exceed this — use `string` or `bigint`.
- `0.1 + 0.2 !== 0.3`; never use floats for money (use integer cents or a
  decimal library).
- `NaN !== NaN`; use `Number.isNaN` (not global `isNaN`, which coerces).
- `parseInt("08")` is fine but `parseInt` stops at first invalid char:
  `parseInt("12px") === 12`. `Number("")` is `0`. Prefer explicit validation.
- `typeof NaN === "number"`, `-0 === 0` but `Object.is(-0, 0) === false`.
- `Array(3)` creates holes, `.map` skips them. Use `Array.from({length: 3})`.
- `sort()` without comparator sorts numbers lexicographically: `[1,10,2]`.

## Strings & equality

- `.length` counts UTF-16 code units: `"👍".length === 2`. Use
  `[...str].length` for code points, `Intl.Segmenter` for graphemes.
- `===` on objects compares identity. Structural comparison needs deep equal.
- Always `===`; `==` only for the idiom `x == null` (checks null and undefined).

## Async pitfalls

- **Floating promises**: an unawaited promise swallows errors. Enable
  `@typescript-eslint/no-floating-promises`. `void promise` only when intended.
- `forEach(async ...)` does not await — use `for...of` (sequential) or
  `Promise.all(arr.map(...))` (parallel).
- `Promise.all` fails fast and abandons the rest; use `Promise.allSettled`
  when partial success matters.
- Unhandled rejection in a `.then` chain vs try/catch around `await` — mixing
  styles creates gaps.
- Race conditions: state read before `await` may be stale after; concurrent
  fetches can resolve out of order (track a request ID or use AbortController).
- `async` functions never throw synchronously except… they can before the first
  `await` in some patterns — always handle rejection, not just try/catch at call site.

## Narrowing gotchas

- Narrowing doesn't survive across closures or `await` for mutable variables —
  copy to a `const` first.
- `typeof x === "object"` includes `null`. Check `x !== null` too.
- Discriminated unions: always handle exhaustiveness with a `never` check:
  ```ts
  default: { const _exhaustive: never = value; throw new Error(...); }
  ```
- `in` narrowing on unions with optional props can mislead; prefer discriminant fields.

## Config baseline

Require in `tsconfig.json`:
```json
{
  "strict": true,
  "noUncheckedIndexedAccess": true,
  "noImplicitOverride": true,
  "noFallthroughCasesInSwitch": true,
  "exactOptionalPropertyTypes": true
}
```

## Testing edge cases in TS

- Test with: empty string/array, `null`, `undefined`, `0`, `-0`, `NaN`,
  emoji strings, `MAX_SAFE_INTEGER + 1`.
- Test rejected promises and aborted requests, not just resolutions.
- For validation code, test that *invalid* shapes are rejected (types won't).
