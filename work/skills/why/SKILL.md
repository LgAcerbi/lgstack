---
name: why
description: Find out why code was built the way it is. Use for "why does X work this way", "why did we pick Y", design rationale, regressions and postmortems. Answers from the record, with citations. Stage 1 (Plan). For runtime behavior, use `how`.
---

# Why

Stage 1 (Plan). Answer from the record, not from the code.

1. Find the target's history: `git log -L` or `git log --follow -- <path>` for the lines or file, then `git blame` on the specific lines in question.
2. Read the commits that introduced the target and each meaningful later change. When a commit references a PR or issue and `gh` is available, read that discussion too.
3. If the trail is thin, look for ADRs, docs, changelogs and comments near the code.

## Answer

- Cite what the authors wrote (commit hash, PR number, doc path).
- Mark your own inferences clearly as inferences ("Likely because...").
- Say what the record does not answer. "No commit explains this" is a finding, not a failure.
- Keep it short: the decision, the reason given, the trade-off accepted, and whether that reason still holds.
