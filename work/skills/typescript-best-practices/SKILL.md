---
name: typescript-best-practices
description: Strict TypeScript rules. Use when reading or editing any .ts or .tsx file, and during Build. Stage 3 (Build).
paths: ["**/*.ts", "**/*.tsx"]
---

# TypeScript best practices

Apply [type-system-discipline](../principles/type-system-discipline.md) and [boundary-discipline](../principles/boundary-discipline.md) first. Follow the project's existing conventions where they are stricter than these rules.

## Check the compiler first

Before applying the rules, read `tsconfig.json`. Many rules below are only enforced by the compiler when strict mode is on.

- If `strict` is off or loosened (`strict: false`, `noImplicitAny: false`, `strictNullChecks: false`), say so in your report as a project-level risk. Do not flip it silently inside an unrelated change: turning on `strict` can surface hundreds of errors.
- Offer a gradual path as its own task: enable `strict` and fix errors module by module, or enable one flag at a time (`noImplicitAny`, then `strictNullChecks`, then the rest).
- Recommended on top of `strict`: `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitOverride`, `noFallthroughCasesInSwitch`.
- Code you write must type-check as if `strict` were on, even when it is off.

## Rules

| Rule | Summary |
| --- | --- |
| **Make bad states impossible** | |
| Discriminated unions | Model variants with a literal discriminant (`kind`). No bags of optional fields and booleans. |
| Branded types | Brand primitives that must not be mixed up: `string & { readonly __brand: "UserId" }`. A brand comes from a verifying type guard or parser, never from `as`. |
| Constructive modeling | Build shapes where the illegal value cannot be constructed: `[T, ...T[]]` for non-empty, `start` plus `durationMs` for a range. |
| Simplest total type | Keep `T[]` while every operation stays total. Strengthen only where the loose type forces `!`, a cast or a "never happens" throw. |
| Literal unions over `enum` | Use `"admin" \| "member"` or an `as const` object, not `enum`. Unions erase at compile time, narrow cleanly and work with schemas. |
| `undefined` for absence | Use `undefined` (optional `?`) for "not there". Use `null` only where an external format requires it (JSON APIs, databases), and convert at the boundary. Never allow both for the same field. |
| **Trust nothing from outside** | |
| `unknown` over `any` | External data is `unknown` until parsed. |
| Parse, don't coerce | `Number()`, `parseInt` and friends are not validation. Parse path params, query strings and headers strictly (exact format, then convert) and reject the rest. |
| Types at the root | When a bug comes from a value of the wrong type, fix the type that allowed it (often an `any` in a domain type), not just the call site. |
| Schemas before guards | Use the project's schema library (zod or similar) and infer types from it (`z.infer`). Do not add a new library for one guard. |
| **Don't lie to the compiler** | |
| No `as` casts | Cast only after validation. Prefer `satisfies`, which checks without widening. |
| Narrowing order | Discriminant switch, then `in`, then `typeof` / `instanceof`, then a user-defined guard. `as` is the last resort. |
| Honest type guards | A guard named `isX` must actually verify X. A lying guard is worse than `as`. |
| Exhaustiveness | In `default` arms, write `const _exhaustive: never = x;` so new variants fail to compile until handled. |
| **State and mutation** | |
| `readonly` by default | Mark shared data `readonly` (`readonly User[]`, `Readonly<T>`). Copy before sorting or reversing (`toSorted`, `[...xs].sort()`). Never let a getter hand out a mutable reference to internal state. |
| No exported mutable state | No `export let`. Expose functions that own the state instead. |
| **Errors** | |
| Expected failures are values | "Not found", "invalid input" and "conflict" are normal outcomes. Return them as a typed result (`T \| undefined`, or `{ ok: true, value } \| { ok: false, error }`). Throw only for bugs and broken invariants. |
| `catch` is `unknown` | Treat the caught value as `unknown` and narrow it (`e instanceof Error`) before reading `.message`. Never `catch (e: any)`. |
| Never swallow errors | A `catch` must rethrow, translate to a typed error, or handle a specific known case. Never return fake success (`res.json([])`) or log and continue. |
| **Async** | |
| No floating promises | Every promise is awaited, returned, or explicitly handled with `.catch`. Mark intentional fire-and-forget with `void` plus error handling. |
| No async where it is ignored | Don't pass `async` callbacks to things that ignore the returned promise (`forEach`, event emitters, older Express handlers). Use `for...of` with `await`, or `Promise.all` for parallel work. |
| Parallel when independent | Use `Promise.all` for independent work. Don't `await` in a loop unless order matters. |
| **Maintainability** | |
| Derive, don't redeclare | Reach for `Pick`, `Omit`, `Parameters`, `ReturnType`, `Awaited` and `typeof` before writing a new interface. |
| Object arguments | Pass an object instead of several positional arguments, so call sites read clearly. Skip this on hot paths. |
| Real tests | Do not mock what you can run. See the `tdd` skill. |
| No stray logging | No `console.log` in shipped code. Use the project's logger with enough context to debug. |

## Let the linter enforce it

Rules a machine can check should be checked by a machine. If the project uses `typescript-eslint`, these rules back the table above (all need type-aware linting except `no-explicit-any` and `no-non-null-assertion`):

| Rule | Enforces |
| --- | --- |
| `@typescript-eslint/no-explicit-any` | `unknown` over `any` |
| `@typescript-eslint/no-unsafe-assignment`, `no-unsafe-member-access`, `no-unsafe-call` | `any` not leaking through |
| `@typescript-eslint/consistent-type-assertions` (`assertionStyle: "never"` or `objectLiteralTypeAssertions: "never"`) | No `as` casts |
| `@typescript-eslint/switch-exhaustiveness-check` | Exhaustiveness |
| `@typescript-eslint/no-floating-promises` | No floating promises |
| `@typescript-eslint/no-misused-promises` | No async where it is ignored |
| `@typescript-eslint/use-unknown-in-catch-callback-variable` | `catch` is `unknown` (with `useUnknownInCatchVariables`, part of `strict`) |
| `@typescript-eslint/no-non-null-assertion` | Simplest total type (no `!`) |
| `@typescript-eslint/prefer-readonly` | `readonly` by default |
| `no-console` | No stray logging |

When the project has no such setup, mention it in your report as a gap. Don't add a linter inside an unrelated change.

Examples for every rule: [patterns.md](patterns.md).
