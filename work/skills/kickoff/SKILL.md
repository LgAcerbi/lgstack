---
name: kickoff
description: Start a brand-new TypeScript/web project. Settles the product's first slice and the stack, scaffolds the project with strict defaults, writes its CLAUDE.md, and proves the setup with a walking-skeleton test. Use for "start a new project", "kickoff", "bootstrap", "set up a new app", or in an empty folder. Stage 0 (Kickoff).
---

# Kickoff

Stage 0 (Kickoff). Turn an idea and an empty folder into a project that the rest of `work` can build on. Every choice made here ends up in the project's CLAUDE.md, so later skills follow it instead of guessing.

If the folder already has code, stop and say so. Use `how` to learn the existing project instead.

Track the steps in the task list. There are two stage gates: after step 2 and after step 5.

## 1. Product

Run the `grill` skill on the product, limited to what kickoff needs:

- What it is and who it is for, in one or two sentences
- The **first slice**: the smallest piece of real functionality a user could try
- Hard constraints: where it runs or deploys, data that must persist, auth, offline use, deadlines
- What is explicitly out of scope for now

Do not design features beyond the first slice. That is `architect`'s job later.

## 2. Stack

Propose a stack as one table, with a recommendation and a one-line reason for each row, shaped by the answers from step 1. Cover:

| Concern | Typical options |
| --- | --- |
| App type and framework | Next.js, Vite + React, Remix, Astro, Node/Hono API, CLI |
| Package manager | pnpm, npm, bun |
| Test runner | Vitest (default), Jest; Playwright for end-to-end |
| Schema / validation | zod (default), valibot |
| Lint and format | ESLint + Prettier, or Biome |
| Styling (if UI) | Tailwind, CSS modules |
| Data (if needed) | SQLite / Postgres with Drizzle or Prisma |

Prefer fewer tools. Every tool must earn its place in the first slice (see [subtract-before-you-add](../principles/subtract-before-you-add.md)). Leave out anything the first slice does not need, such as a database for a static page.

**Gate:** follow [stage-gates](../principles/stage-gates.md). Present the product summary and the stack table, and stop until the user approves.

## 3. Scaffold

1. Use the framework's official scaffolder where one exists (`pnpm create vite`, `create-next-app` and so on), non-interactively with flags. Never pipe remote scripts into a shell.
2. Make TypeScript strict in `tsconfig.json`: `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitOverride`, `noFallthroughCasesInSwitch`. Relax a flag only if the framework demands it, and record why.
3. Add the chosen test runner, schema library and lint/format setup. Delete scaffolder demo code and assets the first slice does not use.
4. Add `package.json` scripts with stable names: `dev`, `build`, `test`, `typecheck`, `lint`, and `check`, which runs `typecheck`, `lint` and `test` together.
5. Run `git init` if the folder is not a repository, with a `.gitignore` that matches the stack. Do not commit unless the user asks.

## 4. CLAUDE.md

Write the project's `CLAUDE.md` from [claude-md-template.md](claude-md-template.md). Fill in every section from the decisions above, and keep it short: rules and commands, not prose. Remove sections that do not apply.

## 5. Walking skeleton

Prove the setup end to end with the thinnest possible version of the first slice. Use the `tdd` loop: one failing test that crosses every layer the slice will use (for example UI → handler → validation → storage), then just enough code to make it pass.

Then run `check` and `build` and report the real output, per [prove-it-works](../principles/prove-it-works.md). If anything fails, fix the setup now. Do not leave it for later.

**Gate:** follow [stage-gates](../principles/stage-gates.md). Summarize the stack, the files created, the scripts and the passing skeleton test. The next step is `architect` for the first real feature, then `tdd`.
