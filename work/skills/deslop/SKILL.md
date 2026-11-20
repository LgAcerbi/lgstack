---
name: deslop
description: Remove AI-generated slop from the current diff without changing behavior. Use after a build, for "deslop", "clean up this diff", or before handing a change for review. Stage 3 (Build).
---

# Deslop

Stage 3 (Build). Check the diff against the branch's base and remove slop introduced on this branch. The base is the PR's base branch, else the branch this one was cut from, else the default branch. Include uncommitted changes.

## Look for

- Comments that narrate the code or do not match the file's style (deeper comment cleanup is `no-comments`)
- Defensive checks, `try/catch` blocks or fallbacks on trusted code paths (see [boundary-discipline](../principles/boundary-discipline.md))
- `any` and `as` casts that only exist to silence the compiler
- Deep nesting that early returns would flatten
- New helpers that duplicate something that already exists in the codebase
- Unused exports, parameters, variables and dead branches
- Anything else inconsistent with the surrounding code

## Guardrails

- Keep behavior unchanged unless you are fixing a clear bug. Report any bug fix separately.
- Make small, focused edits, not broad rewrites.
- After editing, rerun the checks that verified the change (tests, type check, lint) and report their results.
- Revert any edit whose check fails.
- Finish with a 1 to 3 sentence summary.
