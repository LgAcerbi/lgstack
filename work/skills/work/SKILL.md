---
name: work
description: The front door of the work workflow, called as /work <request>. Takes any development request, inspects the project, picks the right path (kickoff for a new project, or plan/design/build/test for an existing one) and runs it stage by stage with approval gates. Use for "/work <request>", or when the user asks to build, implement, fix or understand something and has not named a specific work skill.
argument-hint: <what you want done>
---

# Work

Route the user's request to the right path through the `work` skills, then run that path one stage at a time.

The request: $ARGUMENTS

If the request is empty, ask what they want done and stop.

## 1. Inspect the project

Look before you classify. The folder decides the branch, not the wording alone.

- Is there a `package.json`, `tsconfig.json` or source files? Is it a monorepo (`workspaces`, `pnpm-workspace.yaml`, `apps/`, `packages/`)?
- Is it a git repository with history?
- Is there a `CLAUDE.md`, and a `docs/design/` folder from earlier `architect` runs?

Classify the folder as **empty** (nothing, or only docs and dotfiles), **existing project**, or **monorepo**.

## 2. Classify the request

| Folder | Request | Path |
| --- | --- | --- |
| Empty | Build something new | `kickoff` → `architect` (first feature) → `tdd` → Test (full) |
| Existing | Question: how / why / explain | `how`, `why` or `teach`. No gates. |
| Existing | New feature or larger change | `how` (only if the area is unfamiliar) → `grill` (only if requirements are fuzzy) → `architect` (`arena` if large or contested) → `tdd` → Test (full) |
| Existing | Small, well-specified change or bug fix | `tdd` directly → Test (light). For a bug, start from a failing test that reproduces it. |
| Existing | Clean up existing code | `deslop`, then `no-comments` → Test (light) |
| Existing | Review or test your own changes that already exist | Test (full) on the current diff |
| Existing | Review someone else's MR or PR (a number, URL or their branch) | `review`. It ends at a gate with draft comments; nothing is posted without approval. |
| Existing | Restructure or migrate the layout (explicit request only) | `project-structure` Migrate mode: `grill` → `architect` → `tdd` → Test (full), one feature at a time |
| Monorepo | New app or package | `kickoff` scoped to a new folder in the workspace, following the repo's existing tooling, then `architect` → `tdd` → Test (full) |
| Monorepo | Anything else | Same as existing, scoped to the relevant package |

Judge "small" honestly. A change that adds a new type, module, endpoint or data shape is a feature, not a small change.

**Conventions in existing projects.** For a new feature or larger change in an existing project, check whether the project's `CLAUDE.md` has a **Conventions** section. If it doesn't, the route always starts with `how` in convention mode (the checklist in `project-structure`), even when the area is familiar. Never restructure existing code unless the user explicitly asked for it; that request is its own route (Migrate mode in `project-structure`).

**Test stage (Stage 4)** runs after Build, on the diff. The `test` skill defines it: **light** is `blast-radius` → `interrogate`; **full** adds `thermo-nuclear-review`; `benchmark-checklist` joins only when performance is at stake. Run it as one step, `work:test light` or `work:test full`.

**Stage skills.** Each stage has a skill that runs the whole stage: `plan`, `design`, `build`, `test`. Use them in the route when a stage runs in full; name the individual skills when only some of a stage runs.

## 3. Resolve conflicts once

Ask the user one question, with your recommended answer, when the signals disagree. For example:

- The request says "new project" but the folder has code: new package in this repo, new sibling folder, or did they mean a feature here?
- The request says "add to the API" but the folder is empty.
- The request could be a small fix or a redesign.

Do not ask about anything you can find out by reading the project.

## 4. Announce the route

Before doing any work, print the route in exactly this format, every time, including for small fixes:

```
**Route:** <folder type>, <request type>
`<skill>` → `<skill>` → ...
**Skipping:** `<skill>` (<reason>), ... (or "nothing")
**Starting with:** `<skill>`
```

For example:

```
**Route:** existing project, bug fix
`tdd` → `deslop` → `no-comments` ── gate ── `blast-radius` → `interrogate` ── gate
**Skipping:** `how` (the cause is in one function), `architect` (no new shape), `thermo-nuclear-review` (small fix), `benchmark-checklist` (no performance claim)
**Starting with:** `tdd`
```

List every skill that will run, including the cleanup skills at the end of a build and the Test skills, and mark each stage gate.

The user can redirect here. If they do, re-route and announce again.

## 5. Run the route

Run each skill in the route with the Skill tool (`work:<skill>`), passing it the request and what earlier stages produced.

- Follow [stage-gates](../principles/stage-gates.md) between stages: summarize, list what is open, name the next skill on the route, and stop. When the user approves, continue with that next skill. You do not need to call `/work` again.
- Run `deslop` and `no-comments` at the end of every build, as `tdd` says.
- If a stage shows the route was wrong (the "small fix" needs a design, the area turns out to be unfamiliar), stop, say so, and propose the corrected route.
- After the last stage, say the route is complete and what is not covered yet: opening a PR, and deploy.
- **The route does not end the workflow.** If the user then asks for more changes, follow "Changes after a stage or route is done" in [stage-gates](../principles/stage-gates.md): `build` on the change, then `test` again, with a short announce such as `Follow-up: build → test light ── gate`. A change that touched code always ends at a Test gate.
