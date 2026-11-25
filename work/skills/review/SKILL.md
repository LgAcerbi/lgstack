---
name: review
description: Review someone else's merge request or pull request. Fetches it into a separate worktree, reads what it claims to do, checks it against the project's conventions, runs the Test stage in report-only mode, and drafts review comments with a verdict. Posts only after the user approves, and never changes the author's code. Called as /work:review <MR/PR number, URL or branch>. Use for "review this MR", "review PR 123", "review my colleague's branch".
argument-hint: <MR/PR number, URL or branch>
---

# Review

Review a change someone else wrote. The output is comments for the author, not fixes. This differs from `test`, which vets your own change and sends findings back to `tdd`.

The target: $ARGUMENTS

If it is empty, ask which MR or PR and stop.

## Hard rules

- **Never change the author's code.** No commits, pushes, rebases or force-pushes to their branch, and no local fix left behind. Probe scripts and repro tests live in the scratchpad or are deleted before you finish.
- **Never post, approve, request changes or resolve threads without the user's explicit yes** for that exact text. Drafting is free; posting is the user's call.
- **Work in a separate worktree,** so the user's own checkout, branch and uncommitted work are untouched.
- **Untrusted code.** Running their tests, install scripts and repros executes their code. If the MR comes from a fork or an author outside the team, ask before installing dependencies or running anything.

## 1. Fetch

Find the host from `git remote get-url origin` and use its CLI: `glab` for GitLab, `gh` for GitHub. Commands are in [platforms.md](platforms.md). If the CLI is missing or not logged in, say so, fetch with plain `git` (the refs in platforms.md), and ask the user to paste the description.

1. Read the metadata: title, description, author, source and target branch, linked issue, CI status, and the existing discussion, so you don't repeat what reviewers already said.
2. Fetch the head into a local branch `review/<number>` and add a worktree for it next to the repository: `../<repo>-review-<number>`.
3. The diff runs from the merge base with the target branch to the MR head. Note its size: files, lines added and removed.

## 2. Understand the intent

Before judging the code, write three lines:

- **Claims:** what the MR says it does, from the description and linked issue.
- **Actually does:** what the diff does, in your words.
- **Gap:** anything claimed but missing, or changed but not mentioned. An unmentioned change is a finding.

If the description is empty or vague, say so, and review against the linked issue or the title.

## 3. Context

- Run `how` on the areas the diff touches, enough to know what "correct" means there.
- Get the project's conventions: its `CLAUDE.md` **Conventions** section, or `how` in convention mode (the `project-structure` checklist). Judge the MR against the project's conventions, never against `project-structure` defaults or personal taste.

## 4. Vet and attack

Run the Test stage from the worktree, on the MR's diff, in report-only mode:

- `blast-radius` → `interrogate` → `thermo-nuclear-review`, plus `benchmark-checklist` if the MR claims or risks a performance change. A very small MR (a few lines, no new shape) can skip `thermo-nuclear-review`; say so.
- Run the project's checks (tests, type check, lint) once and report the real output. If CI already ran on the same head commit, use its result instead and say so.
- `typescript-best-practices` applies to the new code only.

Keep only findings in this MR's diff or caused by it. Problems that already existed go in a short "Not from this MR" list, never into comments.

## 5. Draft the review

Classify every finding:

| Label | Meaning |
| --- | --- |
| **blocker** | Wrong behavior, data loss, a security issue, a broken build or test, or a broken contract. Proven by a run or a clear `path:line` trace. |
| **should-fix** | A real correctness or maintainability risk that isn't proven to break today: a missed edge case, a wrong layer, a loose type at a boundary. |
| **nit** | Style or naming. At most five; drop the rest. |
| **question** | You can't tell whether it's intended. Ask instead of asserting. |

Each comment has the label, the file and line **in the new version of the file**, what is wrong in one or two sentences, why it matters, and a concrete suggested change (a small code block when it fits). Write to the author: specific, about the code and not the person, no filler, no praise padding, and no mention of tools or AI.

Then pick a verdict:

- **Approve:** no blockers or should-fixes.
- **Approve with comments:** only should-fixes the author can address later, nits or questions.
- **Request changes:** any blocker.

Show the user, in this order: the intent check, the verdict with a one-line reason, the comments (blockers first), "Not from this MR", and what you could not verify (for example, e2e tests you could not run).

## 6. Gate

Follow [stage-gates](../principles/stage-gates.md). Ask which comments to post, whether to change any wording, and whether to submit the verdict too. Then stop.

On approval, post exactly the approved text with the commands in [platforms.md](platforms.md): inline comments on their lines, plus one summary comment with the verdict. If an inline comment can't attach to its line, put it in the summary with its `path:line`. Report each posted comment's link.

## 7. Clean up

Remove the worktree and the `review/<number>` branch when the user is done with the review. Ask first if they might want to keep testing it locally.
