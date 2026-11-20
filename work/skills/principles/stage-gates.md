# Stage gates

The user approves the move from one stage to the next. Inside a stage, act without asking. Between stages, stop.

## At the end of every stage

1. Summarize what the stage produced in a few lines: the mental model, the chosen design, or the diff and its test results.
2. List what is still open or assumed.
3. Name the next stage and the skill you would run.
4. Stop and wait. Do not start the next stage until the user says so.

## Inside a stage

- Do the work. Look up facts yourself instead of asking for them.
- Ask only for decisions that belong to the user, such as scope, trade-offs and naming that leaks into public APIs.
- If the stage reveals that an earlier stage was wrong (the design does not survive the first test, say), stop and say so. Do not patch around it.
