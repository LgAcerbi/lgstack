# CLAUDE.md template

Copy this into the new project's `CLAUDE.md` and fill it in. Delete any section that does not apply. Keep each line a rule or a fact. No prose.

```markdown
# <Project name>

<One sentence: what it is and who it is for.>

## Commands

- `<pm> dev`: run locally
- `<pm> test`: run tests (<runner>)
- `<pm> typecheck`: TypeScript, strict
- `<pm> lint`: <ESLint + Prettier | Biome>
- `<pm> check`: typecheck + lint + test. Run before calling any change done.

## Stack

- <Framework and version>, <package manager>
- Validation: <zod>. External data is `unknown` until parsed with a schema; infer types from schemas.
- Tests: <Vitest>. Test behavior through public interfaces; mock only at system boundaries.
- <Styling / data / auth, only if used>

## Layout

- `src/<...>`: <what lives here>
- `docs/design/`: approved design sketches from `/work:architect`

## Rules

- Use the `work` plugin flow: design (`/work:architect`) → build (`/work:tdd`) → clean (`/work:deslop`, `/work:no-comments`). Stop for approval between stages.
- Strict TypeScript: no `any`, no `as` before validation, discriminated unions for variants, exhaustive switches.
- Do not add a dependency without saying why the existing ones cannot do the job.
- <Project-specific rules: naming, error handling, logging>

## Out of scope (for now)

- <Things the user explicitly deferred at kickoff>
```
