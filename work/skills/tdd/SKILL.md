---
name: tdd
description: Test-driven development with a red-green loop in vertical slices, testing at agreed seams. Use when building a feature or fixing a bug test-first, on "red-green", or when Build starts after an approved design. Stage 3 (Build).
---

# Test-driven development

Stage 3 (Build). TDD is the red → green loop. This skill makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop. Every section applies on every cycle.

Before starting, read the approved design sketch (usually `docs/design/<slug>.md` from `architect`) and any `GLOSSARY.md`, so test names and interface vocabulary match the domain. Follow [test-behavior-not-implementation](../principles/test-behavior-not-implementation.md) and [subtract-before-you-add](../principles/subtract-before-you-add.md). Use the project's existing test runner (Vitest, Jest, Playwright and so on). Do not add a new one.

## What a good test is

Tests verify behavior through public interfaces, not implementation details. Code can change entirely and the tests should not. A good test reads like a specification: "user can check out with a valid cart" says exactly what capability exists, and it survives refactors because it ignores internal structure.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for when to mock.

## Seams: where tests go

A **seam** is the public boundary you test at, the interface where you observe behavior without reaching inside. Tests live at seams, never against internals.

**Test only at agreed seams.** Before writing any test, list the seams you plan to test and confirm them with the user. Give each a one-line note on what it catches and what it misses. You cannot test everything. Agreeing on seams up front puts the effort on critical paths and complex logic instead of every edge case.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private functions, or verifies through a side channel (querying the database instead of the interface). The tell: it breaks on a refactor that did not change behavior.
- **Tautological**: the assertion recomputes the expected value the way the code does (`expect(add(a, b)).toBe(a + b)`), so it passes by construction. Expected values come from an independent source: a literal, a worked example, the spec.
- **Horizontal slicing**: writing all the tests first, then all the code. Bulk tests check imagined behavior and lock in structure before you understand the problem. Work in **vertical slices**: one test, one implementation, repeat. Each test is a tracer bullet that responds to what the last cycle taught you.

## Rules of the loop

- **Red before green.** Write the failing test first and run it to see it fail for the right reason. Then write only enough code to pass. No anticipating future tests, no speculative features.
- **A test that passes on first run is not red.** It proves nothing about the change. Keep it only as a labeled regression guard, and never count it as the reproduction.
- **Isolate state.** Every test starts from a known state it sets up itself (reset in-memory stores, seed fixtures in `beforeEach`). A test must pass alone, in any order, and after any other test. Never depend on seed data another test might mutate.
- **Fix the cause, then the type that let it happen.** For a bug, find the root cause per [fix-root-causes](../principles/fix-root-causes.md). If a loose type allowed it (an `any`, a missing brand, a coercion at a boundary), tighten that type too, or list it as open work with the reason. Parse inputs strictly per [boundary-discipline](../principles/boundary-discipline.md).
- **Probe the edges of the fix.** Before calling green, try the inputs next to the bug: malformed, empty, out-of-range and alternate formats of the same value. Add a test for any that misbehave.
- **Edit with the Edit tool.** Make code changes with Edit or Write, not `sed` or other shell rewrites, so every change is exact and reviewable.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Types count.** Run the type checker each cycle along with the tests. Apply `typescript-best-practices` to the production code.
- **Refactoring is not part of the loop.** It happens after the loop, in `deslop` and `no-comments`.
- **Design drift stops the loop.** If a test shows the approved sketch is wrong, stop and raise it, as `architect` describes.

## Finish

When every agreed seam is covered and green, run the full test suite, type check and lint, and report the real output per [prove-it-works](../principles/prove-it-works.md). Then run `deslop` and `no-comments` on the diff, and finish with [stage-gates](../principles/stage-gates.md). The next stage is Test (`blast-radius`, then `interrogate`).

When `tdd` runs to fix findings from the Test stage, turn each accepted finding into a failing test first, using the finding's repro command as the starting point.
