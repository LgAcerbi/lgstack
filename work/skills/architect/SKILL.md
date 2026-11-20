---
name: architect
description: Sketch types, signatures and module structure before writing code, then keep implementation honest against the sketch. Use for "architect this", "design this", or any non-trivial change where jumping to code would lock in the wrong shape. Stage 2 (Design).
---

# Architect

Stage 2 (Design). Settle the shape of the code before writing it. Follow [foundational-thinking](../principles/foundational-thinking.md), [type-system-discipline](../principles/type-system-discipline.md) and [subtract-before-you-add](../principles/subtract-before-you-add.md).

For a large or contested design, run the `arena` skill instead of sketching alone. If requirements are still fuzzy, run `grill` first.

## 1. Trace

Map every system the change touches: the modules, the data that flows through them, who owns each piece of state today, and the existing types and helpers you could reuse. Use `how` if the area is unfamiliar.

## 2. Sketch two shapes

Write two genuinely different sketches. Each one contains:

- The key TypeScript types (discriminated unions, branded ids, domain types parsed at boundaries)
- Function and module signatures, with no bodies
- Which module owns which state
- What callers need to know to use it

## 3. Pick

Compare the two against these questions:

- Does each piece of state have exactly one owner?
- Is there exactly one way to do each task?
- Do callers need to know the least?
- Can a bad state even be written?
- What does it delete or reuse versus add?

Pick one and say in a few lines why it won and what you gave up. If you find yourself adding workarounds to make a sketch fit, redesign it.

## 4. Save and gate

Save the chosen sketch and rationale to `docs/design/<slug>.md` in the project, unless the user prefers another place. Follow [stage-gates](../principles/stage-gates.md): present the sketch and stop for approval. The next stage is Build (`tdd`).

## During Build

The sketch is the contract. When implementation needs to deviate, stop, explain why, and get the user's approval. Then update the sketch and note who approved the change. Never edit the sketch just to justify code that already drifted.
