---
name: interrogate
description: Adversarial review. Several subagents, ideally on different models, try to break a change or design, and only findings proven by a real run survive. Use for "interrogate", "adversarial review", "attack this diff", "stress test this code", "find blind spots" or "tear this apart". Stage 4 (Test). Reports findings, never fixes them.
---

# Interrogate

Stage 4 (Test). Adversaries try to break a change. A finding survives only when a run proves it. Report the findings. Do not apply fixes.

Track the steps in the task list: Frame, Fan out, Verify, Report.

## 1. Frame

- **Scope:** what the user names. Otherwise the diff from the merge base with the base branch through the working tree, including uncommitted changes.
- **Intent:** restate in a few sentences what the change is supposed to do, from the user's request, the approved design sketch (`docs/design/`), the commits and the tests. Ask the user only if the intent is genuinely unclear.

## 2. Fan out

Spawn 3 adversaries in one message, in the background, as `general-purpose` subagents. Mix models for real diversity (for example `opus`, `sonnet` and the session model). Give each the scope, the intent, and this brief:

> Break this change. Prove each finding with a run against the real code: a failing test, a repro script, or command output that shows the wrong behavior. Do not edit any file in the repository. Put every repro script in your own temporary directory outside the repo and import the real modules from there. For a design or plan, probe the current code or run a small prototype in that directory. Report each finding with a title, the exact command, the output it printed, and the output the intent requires. Leave out anything you did not run, including style, naming, structure, and risks that need some future change to matter. No findings is a valid result.

Give each adversary a different temporary directory so they cannot interfere with each other. If one produces nothing, continue with the rest and note the dropout.

## 3. Verify

Rerun each finding's command yourself. Keep a finding when the output shows the failure and the failure breaks the intent. Drop the rest. Merge findings that share a root cause.

Adversaries find different failures, so keep every verified finding, not just one adversary's list.

## 4. Report

- **Intent:** your restatement.
- **Findings:** for each verified finding, the command, its output, how it breaks the intent, and which adversaries found it. Say "no verified findings" when none survive.
- **Dropped:** for each adversary, how many findings were dropped and why.
- **Cleanup:** confirm the repository has no new or changed files from the adversaries (`git status`).

Report only. Verified findings go back to `tdd` as failing tests after the stage gate.
