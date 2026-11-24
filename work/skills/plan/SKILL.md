---
name: plan
description: Run the Plan stage: understand an area of the code before changing it, using how, why or teach as the question needs. Called as /work:plan <area or question>. Use for "plan this", "help me understand X before I change it", or the start of work in an unfamiliar area. Stage 1 (Plan). Changes nothing.
argument-hint: <area or question>
---

# Plan

Stage 1 (Plan). Build a shared mental model of the area. Change nothing.

The area or question: $ARGUMENTS

If it is empty, ask which area and stop.

## Pick the skill

| The user needs | Run |
| --- | --- |
| How it works today, with citations | `how` |
| Why it was built this way, from history | `why` |
| A plain, full understanding of both | `teach` (it runs `how` and `why`) |
| The project's conventions before adding a feature | `how` in convention mode (`project-structure` checklist) |

When the request is about an upcoming change, run `how` and add `why` only where the history explains a constraint the change must respect.

Stop at the end with [stage-gates](../principles/stage-gates.md): the mental model, what is still unknown, and the next stage (`design` for a new shape, `build` for a small, well-specified change).
