---
name: principles
description: The engineering principles every `work` skill follows. Load when another work skill names a principle, or when the user asks what rules this workflow follows.
---

# Principles

These are the rules behind every `work` skill. Each one has its own file. Read the ones a skill names before acting on them.

| Principle | One line |
| --- | --- |
| [stage-gates](stage-gates.md) | Stop at the end of every stage and get the user's approval before the next. |
| [subtract-before-you-add](subtract-before-you-add.md) | Delete first, change the least, add only what the evidence justifies. |
| [foundational-thinking](foundational-thinking.md) | Get data shapes and ownership right before writing logic. |
| [type-system-discipline](type-system-discipline.md) | Types make impossible states unrepresentable. |
| [boundary-discipline](boundary-discipline.md) | Validate where data enters. Trust types inside. |
| [prove-it-works](prove-it-works.md) | "It compiles" is not evidence. Run the real thing. |
| [fix-root-causes](fix-root-causes.md) | Fix the cause, never guard the symptom. |
| [test-behavior-not-implementation](test-behavior-not-implementation.md) | Tests check what callers see, not how the code does it. |

## The stages

The `work` skill (`/work <request>`) picks the route through these stages automatically. Each skill can also be called on its own, and each stage has a skill that runs all of it: `plan`, `design`, `build`, `test`. Any code change after a finished route goes back through `build` and `test`.

| Stage | Skills | Ends with |
| --- | --- | --- |
| 0. Kickoff: start a new project | `kickoff` | A scaffolded project with strict defaults, a CLAUDE.md and a passing walking-skeleton test |
| 1. Plan: understand the code | `how`, `why`, `teach` | A shared mental model of the area |
| 2. Design: settle the shape | `grill`, `architect`, `arena` | An approved sketch of types, signatures and modules |
| 3. Build: write it test-first, then clean | `tdd`, `typescript-best-practices`, `deslop`, `no-comments` | A tested, clean diff |
| 4. Test: vet the change and attack the diff | `blast-radius`, `benchmark-checklist`, `interrogate`, `thermo-nuclear-review` | Proven risks and findings for the user to accept or reject. Fixes go back to Build. |

Reviewing someone else's MR or PR: `review` (report-only; drafts comments, posts only with approval).

Used across stages: `project-structure` (where code lives: scaffold in new projects, follow conventions in existing ones) and `typescript-best-practices`.

Not covered yet: opening your own PR, and deploy.
