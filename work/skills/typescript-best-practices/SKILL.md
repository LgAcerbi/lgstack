---
name: typescript-best-practices
description: Strict TypeScript rules. Use when reading or editing any .ts or .tsx file, and during Build. Stage 3 (Build).
paths: ["**/*.ts", "**/*.tsx"]
---

# TypeScript best practices

Apply [type-system-discipline](../principles/type-system-discipline.md) and [boundary-discipline](../principles/boundary-discipline.md) first. Follow the project's existing conventions where they are stricter than these rules.

| Rule | Summary |
| --- | --- |
| Discriminated unions | Model variants with a literal discriminant (`kind`). No bags of optional fields and booleans. |
| Branded types | Brand primitives that must not be mixed up: `string & { readonly __brand: "UserId" }`. A brand comes from a verifying type guard or parser, never from `as`. |
| Constructive modeling | Build shapes where the illegal value cannot be constructed: `[T, ...T[]]` for non-empty, `start` plus `durationMs` for a range. |
| Simplest total type | Keep `T[]` while every operation stays total. Strengthen only where the loose type forces `!`, a cast or a "never happens" throw. |
| `unknown` over `any` | External data is `unknown` until parsed. |
| Parse, don't coerce | `Number()`, `parseInt` and friends are not validation. Parse path params, query strings and headers strictly (exact format, then convert) and reject the rest. |
| Types at the root | When a bug comes from a value of the wrong type, fix the type that allowed it (often an `any` in a domain type), not just the call site. |
| Schemas before guards | Use the project's schema library (zod or similar) and infer types from it (`z.infer`). Do not add a new library for one guard. |
| No `as` casts | Cast only after validation. Prefer `satisfies`, which checks without widening. |
| Narrowing order | Discriminant switch, then `in`, then `typeof` / `instanceof`, then a user-defined guard. `as` is the last resort. |
| Honest type guards | A guard named `isX` must actually verify X. A lying guard is worse than `as`. |
| Exhaustiveness | In `default` arms, write `const _exhaustive: never = x;` so new variants fail to compile until handled. |
| Derive, don't redeclare | Reach for `Pick`, `Omit`, `Parameters`, `ReturnType`, `Awaited` and `typeof` before writing a new interface. |
| Object arguments | Pass an object instead of several positional arguments, so call sites read clearly. Skip this on hot paths. |
| Real tests | Do not mock what you can run. See the `tdd` skill. |
| No stray logging | No `console.log` in shipped code. Use the project's logger with enough context to debug. |

Examples for each rule: [patterns.md](patterns.md).
