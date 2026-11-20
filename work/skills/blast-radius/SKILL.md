---
name: blast-radius
description: Find what a change could break outside its own diff, and prove the one fact it is safe because of by running real code. Use for "blast radius of X", "what could this break", a small diff you don't trust, or a request that asserts something about existing code ("X already handles Y") before you design against it. Stage 4 (Test).
---

# Blast radius

Stage 4 (Test). Find what a change breaks somewhere else, before it ships. `how` says what code does and `why` says why it is shaped that way. Blast radius says what a change breaks beyond the diff.

Listing callers is not the job. Grep does that in a second. The job is the breakage grep won't show you.

Also run it before design when a request asserts something about existing code ("make endpoint X public", "X already validates Y"). The assertion is a hypothesis until proven.

## Don't trust your own writeup

A blast-radius writeup that sounds right is worthless, because it sounds right whether or not it is true. Find the one or two facts the change's safety depends on and prove them by running code.

For each fact, get it as far down this list as is cheap, and say where it stopped:

1. You said so. Worthless on its own.
2. You pointed at the line: a real `file:line`, or the library's own source.
3. You showed the bad case can't happen by walking the failure step by step.
4. You ran it: a script or test that calls the real code and fails loudly if you are wrong.
5. You reproduced it in the running app.

Level 4 is usually one small script that imports the real module and calls the exact function you are worried about.

## Steps

1. **Read the change.** The diff (against the base branch, including uncommitted changes), the symbols it adds, changes and deletes, and what it now does differently, including what the diff does not spell out.
2. **Find the one fact it is safe because of.** Most risky-looking changes are safe because of a single fact, like "ids are only ever created by `nextId`". If it holds, most risks clear at once. Spend your time here, not on a long list of maybes.
3. **Look where grep stops.** Read the source of libraries you call at their installed version (`node_modules`). Work out when things run: middleware order, async timing, module-level state shared between requests or tests. Follow what a symbol search misses: JSON shapes clients depend on, status codes, HTTP routes called by string, env vars, data written in one place and read in another.
4. **Be honest about each risk.** Give it a real likelihood and a real cost. Keep the risks you confirmed and list the ones you checked and cleared separately. Cite real `file:line`s. A search that finds nothing is still an answer. Never invent a caller or an API.
5. **Prove the one fact.** Write a script or test that runs the real code, run it, and paste what happened. Put throwaway scripts outside the repository.
6. **For a big or wide change,** run the question through `arena` so several models look independently, then merge their answers.

## Hand back

- **What it does:** what changed, including the non-obvious part.
- **The one fact it's safe because of:** stated, with the level you reached and the proof. If you couldn't prove it, write "unproven".
- **Risks:** for each, how it breaks, `file:line`, likelihood and cost, and how to check. Paste the proof for the ones that matter.
- **Cleared:** what you checked and why it is fine.
- **Before you merge:** the cheapest test or repro that would catch the real bug, including any script you wrote.

Report only. Do not fix what you find. Fixes go back to `tdd` after the stage gate.
