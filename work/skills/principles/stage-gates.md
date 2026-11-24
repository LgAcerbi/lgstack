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

## Changes after a stage or route is done

A finished route is not a license to edit freely. When the user asks for more changes after Build or Test ("rename this", "also handle X", "change the status code"), any change to code runs through `build` and then `test` again:

1. Make the change test-first with `tdd` (a behavior change gets a failing test; a pure rename or move needs the existing tests green), then `deslop` and `no-comments` on the new changes.
2. Run `test` on the result. Pick the level by what the follow-up did, not by how small the request sounded: `light` for behavior inside existing shapes, `full` if it added a type, module, endpoint or data shape.
3. Stop at the Test gate as usual.

Skip Test only when the change touched no code (docs, comments, config the code never reads), and say that you skipped it and why. Never end a turn that changed code with "done" and no Test run.
