---
name: build
description: Run the whole Build stage: tdd, then deslop and no-comments, then hand off to the Test stage. Called as /work:build <what to build or change>. Use for "build this", "implement the approved design", or a follow-up change to code that was already built and tested. Stage 3 (Build).
argument-hint: <what to build or change>
---

# Build

Stage 3 (Build). Write the change test-first, then clean it, then send it to Test. Never skip Test for a change that touched code.

The change: $ARGUMENTS

If it is empty, use the approved design (`docs/design/<slug>.md` from `architect`) or the findings the user accepted at the last Test gate. If neither exists, ask what to build and stop.

## Steps

1. **`tdd`**: red-green in vertical slices at agreed seams. For accepted Test findings, each finding's repro becomes the first failing test. `typescript-best-practices` applies to every `.ts`/`.tsx` edit.
2. **`deslop`** on the diff.
3. **`no-comments`** on the diff.
4. Run the full test suite, type check and lint once more, and report the real output.

Run these without stopping between them. Then follow [stage-gates](../principles/stage-gates.md): summarize the diff and test results, and name `test` as the next stage, with the level it would pick (`light` or `full`, per the `test` skill) and why.

If the change turns out to need a new shape that was never designed, stop and propose `design` first.
