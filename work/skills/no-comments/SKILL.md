---
name: no-comments
description: Delete needless comments in the diff and turn the surprising code they explain into code that needs no comment. Spawns the comment-hater reviewer, then applies what holds up. Use for "no comments", "remove comments", or as the last Build step. Stage 3 (Build).
---

# No comments

Stage 3 (Build). A comment that explains our own surprising code is a sign the code should change. Delete the comment and fix the shape.

## Scope

Use the files or diff the user gives you. Otherwise use the current diff against the base branch (default `main`), including uncommitted changes.

## Skip when there is nothing to judge

If the diff adds, changes or sits directly next to no comments, `eslint-disable`, `@ts-ignore` or `@ts-expect-error` lines, skip the steps below. Say "no-comments: skipped, the diff touches no comments" and show how you checked (for example, the `git diff` lines you searched). Otherwise, run every step.

## Steps

1. **Review.** Spawn the `comment-hater` agent (`subagent_type: "work:comment-hater"`) and pass it the scope. Do not restate its rules. It deletes comments and reports. It never edits application code.
2. **Check its work.** Read its report and diff. Reject any edit to application code, anything outside the scope, and any deletion of a comment that fits one of its keep rules. Restore what you reject. For a comment that claims "important" or "do not remove", trace the symbol or run `why` before accepting the kill.
3. **Fix the shape.** For each `MUST KILL` flag, make the code obvious without prose: rename, extract a function, add a type, or use the real API. If a fix needs a real design change, run `architect` once for all of them and stop at the sketch for approval.
4. **Encode constraints.** When a deleted comment stated a real constraint ("do not change this wording", "talk to X first"), offer the cheapest enforcement instead: a type, a test or a lint rule. Wait for the user's approval before adding it.
5. **Verify.** Run tests, type check and lint. Revert any change that breaks them.
6. **Report** the number of deletions, restored comments, `HOLD` items, `MUST KILL` fixes, constraint encodings offered or added, and anything left open.
