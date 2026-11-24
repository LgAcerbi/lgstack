---
name: how
description: Explain how code works. Use for "how does X work", walkthroughs before changing something, and placement questions like "where should this live" or "which module owns this". Stage 1 (Plan). For the reasons behind a design, use `why`.
---

# How

Stage 1 (Plan). Build an accurate mental model of the code before anyone changes it.

If the scope is ambiguous, state how you read it and explore. The user can redirect.

## 1. Explore

Spawn one read-only subagent (`subagent_type: "Explore"`, or `general-purpose` if you need deeper reads) with the user's question and this brief:

> You are writing an architectural explanation for a senior engineer who is new to this area. Explore the codebase and explain it so they walk away with a solid mental model and could start working in it confidently. Cover: the main pieces and what each owns, how data and control flow between them at runtime, where the entry points are, and the files a change would most likely touch. Cite files as `path:line`. Read-only: change nothing.

A fresh subagent keeps the exploration noise out of the main conversation.

**Convention mode.** When the user (or the router) is about to add or change code in an existing project, also ask the subagent to answer every question in the convention checklist of the `project-structure` skill (read [its SKILL.md](../project-structure/SKILL.md)), each with one real `path:line` example. Present the answers as a **Conventions** section. If the project's `CLAUDE.md` has no **Conventions** section, offer to add it there. Write it only with the user's approval.

## 2. Present

Present the subagent's explanation. Light edits for clarity or to connect it to the conversation are fine. Do not rewrite its substance. If a flow has three or more moving parts, add a small mermaid diagram.

## 3. Gate

If the user is heading toward a change, follow [stage-gates](../principles/stage-gates.md). Name `why` if the motivation matters, or `grill` / `architect` (Stage 2) if they are ready to design.
