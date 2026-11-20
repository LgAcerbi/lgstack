---
name: grill
description: Grill the user relentlessly about a plan, feature or design until there is a shared understanding with nothing silently assumed. Use for "grill me", "stress-test this", "interrogate this plan", or before designing anything non-trivial. Stage 2 (Design).
---

# Grill

Stage 2 (Design). Interview the user until you both understand the plan the same way. Map it as a **design tree**: every decision branches into the decisions that depend on it.

## Rounds

Work the tree in rounds. The **frontier** is every decision whose prerequisites are already settled, meaning the questions you can ask now without guessing at answers you have not heard. Ask the whole frontier in one round. Number each question and give your recommended answer. Then wait.

Format each round like this:

```
❓ **Q1** - **<question title>**: <question body, with options if there are any>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body>

➡️ <your recommended answer>
```

Word each question so that "yes" accepts your recommendation.

Each round of answers reshapes the tree. Settled decisions push the frontier outward and unblock the questions that depended on them. Recompute the frontier and ask the next round. A question that depends on another question still open in this round belongs to a later round.

## Facts versus decisions

Finding **facts** is your job, never the user's. When a question needs a fact from the codebase or environment, dispatch a subagent with fresh context to find it instead of asking. Do not block on it: only the questions downstream of that lookup wait. Ask the rest of the frontier now.

**Decisions** belong to the user. Put each one to them and wait.

## Done

The session is done when the frontier is empty: every branch visited, nothing left silently assumed. Then write a short summary of every decision made, as a list. Follow [stage-gates](../principles/stage-gates.md): do not act on the plan until the user confirms the summary. The usual next skill is `architect`.
