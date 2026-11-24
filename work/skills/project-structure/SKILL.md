---
name: project-structure
description: Where code lives and which layer owns it. Scaffolds the preferred layout in new projects, discovers and follows the existing conventions in existing projects, and restructures only when explicitly asked. Use when creating files or modules, deciding "where should this live", designing a feature, scaffolding a project, or reviewing whether code is in the right layer.
---

# Project structure

Decide where code goes. In a new project, use the preferred layout. In an existing project, follow the project. Never restructure on your own.

## Precedence

1. The project's `CLAUDE.md` (its **Conventions** section, if present)
2. The conventions already visible in the project's code
3. The defaults in this skill, which fill gaps only (for example, the project has no folder yet for a new kind of thing)

Consistency beats preference. One feature in a different style is worse than a whole project in a style you don't love.

## Pick the mode

| Situation | Mode |
| --- | --- |
| Empty folder, or `kickoff` is running | **Scaffold** |
| Existing project, any feature, fix or cleanup | **Follow** |
| The user explicitly asks to restructure ("move the API to layers", "migrate to services") | **Migrate** |

### Scaffold

Create the default layout for the project type and record it in the project's `CLAUDE.md` under **Conventions**.

- REST API (Express): [rest-api.md](rest-api.md)
- Other project types are not covered yet. Ask the user for the layout during `kickoff` and record their answers.

### Follow

1. **Know the conventions.** If the project's `CLAUDE.md` has a **Conventions** section, read it and trust it. Otherwise run `how` with the checklist below and ask it for a **Conventions** section, citing a real example for each answer.
2. **Offer to save them.** If `CLAUDE.md` has no **Conventions** section, offer to add what `how` found, so later runs read it instead of rediscovering it. Write it only with the user's approval.
3. **Place new code the project's way:** same grouping, same layers, same naming, same wiring, same test location. Copy the shape of the closest existing feature.
4. **Report differences, don't fix them.** When the project's conventions differ from these defaults, say so in one or two lines in the stage report ("this project calls repositories from routes; the default goes through a service; I followed the project"). Suggest a migration as a separate task only if the difference causes real problems.
5. **Never move, rename or re-layer existing code** as part of a feature, fix or cleanup.

### Migrate

Treat it as its own feature: `grill` on the scope (which modules, in what order) → `architect` for the target layout and the migration steps → `tdd` with the existing tests kept green at every step → Test (full). Migrate one feature at a time, never in one giant diff.

## Convention checklist

The questions `how` answers in Follow mode, and the decisions recorded in `CLAUDE.md`:

| Topic | Question |
| --- | --- |
| Grouping | By layer (`services/`, `controllers/`) or by feature (`users/`, `orders/`), or features with layers inside? |
| Layers | Which layers exist, and who may call whom? Can routes skip the service and call data access directly? |
| Style | Classes or functions for services, controllers and data access? |
| Wiring | How are dependencies created and passed: constructor injection, factories, plain imports, a DI container? Where? |
| Validation | Where is request input parsed and with what (zod, middleware, manual checks)? |
| Errors | How do expected failures travel (thrown errors, result values) and where become HTTP statuses? |
| Types | Where do domain types live? Are API shapes separate from domain types? |
| Naming | File naming pattern (`user.service.ts`, `userService.ts`, `users/service.ts`)? |
| Tests | Next to the code or in a separate folder? Unit, integration or both? How is state reset? |
| Config | Where are environment variables read? |

## Layering rules

These hold in any layer-based project, and are the defaults in Scaffold mode:

1. **Dependencies go one way:** routes → controllers → services → repositories → models. Never backwards.
2. **Features talk through services.** A service may call another feature's service, never another feature's repository. Each repository belongs to exactly one service.
3. **No circular imports** between services. If two services need each other, extract the shared logic into a third service, or revisit the design.
4. **Consistent names per feature across layers** (`user.routes.ts`, `user.controller.ts`, `user.service.ts`), so a feature is easy to find in every folder.
5. **Revisit point.** When the project grows clearly separate domains (billing, auth, catalog), or layer folders get crowded, suggest moving to feature folders with layers inside. Suggest it as a separate Migrate task. Never do it unasked.

In an existing project, apply only the rules the project already follows.

When the project uses ESLint, `eslint-plugin-boundaries` or `dependency-cruiser` can enforce rules 1 to 3. Suggest it as a separate task, don't add it in the middle of a feature.
