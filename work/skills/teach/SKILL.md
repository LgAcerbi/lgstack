---
name: teach
description: Explain a body of work plainly so the user actually understands it. Runs `how` and `why` and weaves the results into one clear explanation. Use for "teach me this", "help me really understand X", "explain this change or subsystem to me". Stage 1 (Plan).
---

# Teach

Stage 1 (Plan). Explain what a thing is, how it works and why it is built that way, at the user's pace. The goal is understanding. Change nothing.

## Steps

1. **Pick the takeaways.** Decide the few things the user should walk away understanding. Base this on why they are asking (about to change it, reviewing it, debugging it, new to it) and what they already know, both read from the conversation. Skip what they clearly already know.
2. **Let `how` and `why` dig.** Read enough code to get oriented, then run the `how` skill and the `why` skill, in parallel when both apply. A small change may only need one. Do not redo their work. Keep `why`'s hedges intact: they are findings, not style.
3. **Start with a plain definition.** Name the thing and say what it is in general terms, with its common name if it has one. Then tie it to this codebase ("here we use it to...") and build out: how it works, then why, then edge cases.
4. **Smallest complete answer first.** A sentence or two, then stop. Add layers when the user asks. Never a wall of text.
5. **Build pictures up.** For three or more moving parts, draw a short series of mermaid diagrams, each adding one part to the last, instead of one diagram with everything.
6. **Keep it a conversation.** Offer to go deeper or move on. No quizzes, no "pause and reflect", no labels like "key insight" or "TL;DR".

## Voice

Plain spoken English, like explaining to a colleague. State the concrete mechanism, not a metaphor. Short sentences. One name per concept, used consistently. No filler, no hype.

Reply with the explanation itself, never a report about what you did.
