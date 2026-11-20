---
name: arena
description: Have several subagents attempt the same design or artifact in parallel, judge them against a rubric, pick a base and graft in the best parts of the others. Use for "arena this", or when one attempt at a non-trivial design would lock in the wrong shape. Stage 2 (Design).
---

# Arena

Stage 2 (Design). One attempt anchors on its first idea. Several independent attempts show you the design space.

Track the phases in the task list: Frame, Fan out, Judge, Pick, Graft, Check.

## A. Frame

Every candidate gets the same prompt, so the prompt is the contract.

1. State the artifact each candidate produces, usually an `architect`-style sketch: types, signatures, module ownership and rationale.
2. Write a rubric of 3 to 6 concrete, gradeable criteria for this task. Only the judges see it, not the candidates.
3. Choose 3 candidates. Mix models where you can (for example `opus`, `sonnet` and the session model) for real diversity. If an attempt needs to write code, give it its own worktree with `isolation: "worktree"`.

## B. Fan out

Spawn all candidates in one message, in the background. Each gets the task, the relevant files and constraints, and this instruction: produce the artifact plus a short rationale naming the alternatives you considered and rejected. If one fails to produce anything, continue without it and note the dropout.

## C. Judge

When all candidates are done, spawn one read-only judge with fresh context, ideally on a different model from yours. It scores each candidate, by label, on every rubric criterion and recommends a base.

## D. Pick a base

Read every candidate end to end yourself. Score each one criterion by criterion, then compare with the judge. If you agree, that confirms the pick. If you disagree, re-read both rationales before deciding. Prefer the base a future maintainer can extend most easily, then the smaller API.

## E. Graft

Walk each losing candidate once for the one or two ideas worth porting. Fold them in by hand so the result stays coherent. Do not paste mechanically.

- If all candidates converged on one shape, that is strong agreement. Ship that shape.
- If they diverged wildly, the frame was under-specified. Reframe and rerun instead of averaging.

## F. Check and gate

Check the final artifact against the rubric and [prove-it-works](../principles/prove-it-works.md) where code is involved. Then present a short synthesis note: the base and why, the judge's verdict, each graft and its source, what was rejected, and any dropouts. Follow [stage-gates](../principles/stage-gates.md) and stop for approval before Build.
