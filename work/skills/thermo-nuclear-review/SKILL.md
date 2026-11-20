---
name: thermo-nuclear-review
description: An extremely strict maintainability review of the current changes, focused on structure, abstraction quality, spaghetti growth, oversized files and "code judo" simplifications. Use for "thermo-nuclear review", "harsh review", "deep code quality audit", or after interrogate in the Test stage. Stage 4 (Test). Reports, never edits.
---

# Thermo-nuclear review

Stage 4 (Test). `interrogate` asks "does it break?". This asks "does it make the codebase worse, and is there a much simpler way?". Do not approve code just because it works.

## Steps

1. **Collect.** Get the diff against the base branch (merge base through the working tree, including uncommitted changes) and read the full contents of every changed file. For a large set, use an `Explore` subagent to gather them.
2. **Review.** Spawn the `thermo-nuclear-reviewer` agent (`subagent_type: "work:thermo-nuclear-reviewer"`) with fresh context. Give it the diff under `### Diff` and the file contents under `### Changed files`, plus the intent of the change and the path to this skill's [rubric](rubric.md). A fresh reviewer has no stake in the code it is judging.
3. **Check its claims.** Open every `file:line` it cites and confirm the evidence. Drop findings that misread the code. Keep the tone. Don't soften real structural problems into suggestions.
4. **Report.** Lead with the verdict (**approve** or **request changes**), then the findings in the rubric's priority order. Each finding is a short paragraph: the file and evidence, the problem, and the remedy. For a large review, also write the full report to `docs/reviews/<slug>.md` and link it.

Report only. Accepted findings go back to `tdd` or `architect` after the stage gate.
