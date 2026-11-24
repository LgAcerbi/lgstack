---
name: design
description: Run the whole Design stage: grill if the requirements are fuzzy, then architect (or arena for large or contested designs). Called as /work:design <feature or change>. Use for "design this", "settle the shape", or before building any non-trivial change. Stage 2 (Design).
argument-hint: <feature or change>
---

# Design

Stage 2 (Design). Settle the shape before code exists.

The change: $ARGUMENTS

If it is empty, ask what to design and stop.

## Steps

1. **`grill`**, only if the requirements are fuzzy, contested or missing edge cases. Say in one line why it runs or is skipped.
2. **`architect`** for the sketch. Use **`arena`** instead when the design is large, has several credible shapes, or the user asks for alternatives. Placement follows `project-structure`.

Announce which of these run and which are skipped, with reasons, before starting. Stop at the end with [stage-gates](../principles/stage-gates.md): the approved sketch, what is open, and `build` as the next stage.
