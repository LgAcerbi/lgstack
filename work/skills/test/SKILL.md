---
name: test
description: Run the whole Test stage on the current changes, in order, with one gate at the end. Called as /work:test [light|full]. Use after finishing or changing an implementation, for "test this", "test everything", "run the test stage", or whenever code changed since the last Test run. Stage 4 (Test). Reports findings, never fixes them.
argument-hint: [light|full] [what to focus on]
---

# Test

Stage 4 (Test). Vet the current changes and attack them, running the Test skills in order. This is the single definition of the Test stage; the router and `build` both point here.

Arguments: $ARGUMENTS

## 1. Scope

- **The diff:** everything changed against the base branch (the PR's base, else the branch this one was cut from, else the default branch), including uncommitted and untracked files. If there are no changes, say so and stop.
- **Follow-up run:** if a Test run already happened in this conversation, name what changed since then. `blast-radius` and `thermo-nuclear-review` still look at the whole diff; `interrogate` aims its adversaries at the new changes first.
- If the user named a focus after the level, pass it to every skill.

## 2. Pick the level

| Level | Skills, in order | When |
| --- | --- | --- |
| **light** | `blast-radius` → `interrogate` | A small, well-specified fix, or a follow-up that changes behavior inside existing shapes |
| **full** | `blast-radius` → `interrogate` → `thermo-nuclear-review` | Anything that adds a type, module, endpoint or data shape; a cleanup or restructure; or when the user says "test everything" |

- The user's argument wins (`light` or `full`). With no argument, pick from the table and say why in one line. When unsure, pick **full**.
- Add `benchmark-checklist` at the end of either level when the change claims or risks a performance difference. Otherwise list it as skipped with the reason.
- A change that touches only docs, comments or tests still gets `blast-radius`; skip the rest and say so.

Announce before starting:

```
**Test:** <light|full> (<reason>)
`blast-radius` → `interrogate` → ... ── gate
**Skipping:** `<skill>` (<reason>), ... (or "nothing")
```

## 3. Run

Run each skill with the Skill tool (`work:<skill>`), on the same scope, one after another **without stopping between them**. They are one stage; the gate is at the end. Pass along what earlier skills found, so `interrogate` can aim at the risks `blast-radius` named.

Also run the project's own checks (tests, type check, lint, from `CLAUDE.md` or `package.json`) once and report the real output per [prove-it-works](../principles/prove-it-works.md). A failing check is a finding.

## 4. Gate

Report all findings in one list, most severe first, each with the skill that found it and its proof (a repro command or a `path:line`). Then follow [stage-gates](../principles/stage-gates.md): ask which findings to fix, and stop.

Accepted findings go to `tdd` (or `build`) as failing tests first, then cleanup. After the fixes, run this skill again: only the skills that found something, plus `blast-radius` on the new changes. Repeat until the user is satisfied.
