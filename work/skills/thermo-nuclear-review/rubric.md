# Thermo-nuclear review rubric

## Baseline

> Perform a deep code-quality audit of these changes. Rethink how to structure and implement them to meaningfully improve quality without changing behavior. Improve abstractions and modularity, reduce spaghetti, improve succinctness and legibility. Be ambitious: if restructuring some of the codebase gives a clearly better implementation, propose it. Be thorough. Measure twice, cut once.

## Standards

0. **Be ambitious about structural simplification.** Don't stop at "this could be a bit cleaner". Look for a **code judo** move: a reorganization that uses the existing architecture better so whole branches, helpers, modes, conditionals or layers disappear. Prefer the version that feels inevitable in hindsight. Delete complexity rather than rearrange it.
1. **No file crosses 1,000 lines because of this change** without a strong reason. If it does, ask whether to decompose first.
2. **No spaghetti growth.** New ad-hoc conditionals, scattered special cases and one-off branches inside unrelated flows are design problems, not nits. Push the logic into its own abstraction, helper, state model or module.
3. **Clean the design, don't just accept working code.** If behavior can stay the same while the structure gets meaningfully simpler, push for it. Prefer removing moving pieces over spreading the same complexity around.
4. **Direct, boring code over magic.** Flag brittle or magical behavior, generic mechanisms hiding simple data shapes, and thin wrappers or pass-through helpers that add indirection without clarity.
5. **Types and boundaries.** Question needless optionality, `any`, `unknown` past the boundary, casts, and loosely shaped objects where an explicit typed model would make the code simpler. Silent fallbacks that paper over an unclear invariant mean the boundary should be explicit.
6. **The right layer, and reuse.** Flag feature logic leaking into shared code, implementation details leaking through APIs, and bespoke helpers duplicating an existing one. Judge placement against the project's conventions (its `CLAUDE.md` **Conventions** section, or the patterns in its code), not against personal defaults. A new file that breaks the project's existing layering is a finding. An existing project that differs from the `project-structure` defaults is not.
7. **Orchestration and atomicity.** Flag independent work serialized for no reason, and related updates that can leave state half-applied.

## Questions for every meaningful change

- Is there a code judo move that makes this dramatically simpler?
- Can it be reframed so fewer concepts, branches or layers are needed?
- Does it improve or worsen the local architecture?
- Did it add branching where a better model should exist? Are repeated conditionals hinting at a missing type or helper?
- Did a cohesive module become more coupled, more stateful or harder to scan?
- Is the logic in the right file and layer?
- Is each abstraction earning its keep, or is it just a wrapper?
- Did it add casts, optionality or ad-hoc shapes that hide the real invariant?

## Preferred remedies

Delete a layer of indirection rather than polish it. Reframe the state model so conditionals disappear. Move ownership so the feature becomes a natural extension of an existing abstraction. Turn special cases into a simpler default flow. Replace condition chains with a typed model or a dispatcher. Separate orchestration from business logic. Collapse duplicate branches. Reuse the canonical helper. Make type boundaries explicit so control flow gets simpler. Split large files into focused modules.

Don't settle for "maybe rename this" when the real issue is structural, or for a cleaner version of the same messy idea when a much simpler idea exists.

## Priority order for findings

1. Structural regressions
2. Missed code judo / dramatic simplifications
3. Spaghetti and branching growth
4. Boundary, abstraction and type-contract problems
5. File size and decomposition
6. Modularity
7. Legibility

Prefer a few high-conviction findings to a long list of cosmetic notes.

## Tone

Direct, serious and demanding. Not rude, but never soften a real maintainability problem into a mild suggestion. If the change makes the codebase messier, say so. If it missed a dramatic simplification, say that too.

## Approval bar

Approve only when there is:

- no clear structural regression
- no visible missed opportunity for a dramatically simpler implementation
- no unjustified file-size explosion
- no spaghetti growth from special-case branching
- no hacky or magical abstraction, and no needless wrapper, cast or optionality churn
- no boundary leak or duplicated canonical helper
- no missed obvious decomposition

Treat these as blockers unless clearly justified: keeping lots of incidental complexity when a code judo move would delete it, pushing a file past 1,000 lines, ad-hoc branching that tangles an existing flow, feature checks scattered across shared code, an unnecessary wrapper or cast-heavy contract, or a duplicated helper or wrong-layer placement.
