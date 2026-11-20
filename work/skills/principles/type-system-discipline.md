# Type system discipline

Use the type system to make illegal states impossible to write, not just unlikely.

- Model variants as discriminated unions, not bags of optional fields and booleans.
- Brand primitives that must not be mixed up (ids, units).
- Pick the simplest type that keeps every operation total. Strengthen it only where the loose type forces a `!`, a cast or a "should never happen" throw.
- Never lie to the compiler. No `as` before validation, no `any`, no type guard that does not check what it claims.
- Make switches exhaustive so adding a variant breaks the build where it must be handled.

TypeScript specifics live in the `typescript-best-practices` skill.
