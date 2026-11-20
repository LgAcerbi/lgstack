# work

A staged, approval-gated workflow for AI-assisted TypeScript/web development in Claude Code.

> If you want to go fast, go deep first.

`work` splits a change into stages. Inside a stage the agent works on its own. Between stages it stops, summarizes, and waits for your approval.

## Just ask

```
/work build me a new REST API in TypeScript
/work implement a new endpoint for listing orders
/work how does auth work here?
```

`/work` inspects the folder, decides between a new project and an existing one, announces the route it picked (for example `architect → tdd`), and runs it one stage at a time, stopping for your approval between stages. If the bare `/work` is taken by another command on your machine, use `/work:work`.

## Stages

| Stage | Skill | What it does |
| --- | --- | --- |
| **0. Kickoff**: new projects only | `/work:kickoff` | Settles the first slice and stack, scaffolds strict TS, writes CLAUDE.md, proves it with a walking-skeleton test |
| **1. Plan**: understand the code | `/work:how` | Explains how code works, with file citations |
| | `/work:why` | Explains why it was built that way, from git history and PRs |
| | `/work:teach` | Runs `how` and `why` and teaches the result plainly |
| **2. Design**: settle the shape | `/work:grill` | Interviews you in rounds until nothing is assumed |
| | `/work:architect` | Sketches two shapes (types, signatures, ownership) and picks one |
| | `/work:arena` | Several subagents design in parallel; a judge helps pick and graft |
| **3. Build**: test-first, then clean | `/work:tdd` | Red-green in vertical slices at agreed seams |
| | `/work:typescript-best-practices` | Strict TS rules, applied automatically to `.ts`/`.tsx` |
| | `/work:deslop` | Strips AI slop from the diff without changing behavior |
| | `/work:no-comments` | Deletes needless comments and fixes the code they explained |
| **4. Test**: vet and attack the diff | `/work:blast-radius` | Finds what the change breaks outside the diff and proves the fact it's safe because of |
| | `/work:benchmark-checklist` | Vets any performance number before you trust it |
| | `/work:interrogate` | Adversary subagents try to break the change; only failures proven by a run survive |
| | `/work:thermo-nuclear-review` | A harsh maintainability review that hunts for dramatically simpler structures |

Test skills only report. You pick which findings to fix, and each fix goes back through `tdd`.

`/work:principles` lists the rules every skill follows, such as stage gates, subtract before you add, boundary discipline and prove it works.

### Typical flow

New project:

```
/work:kickoff  → product + stack                ── you approve
               → scaffold, CLAUDE.md, skeleton test ── you approve
/work:architect → first feature, then /work:tdd as below
```

Existing project:

```
/work:how      → understand the area            ── you approve
/work:grill    → settle requirements
/work:architect → approved sketch in docs/design ── you approve
/work:tdd      → red-green, then deslop + no-comments ── you approve
/work:blast-radius → /work:interrogate → /work:thermo-nuclear-review ── you pick fixes
```

Small fixes get a lighter Test stage: `blast-radius` → `interrogate`.

## Install

```
/plugin marketplace add LgAcerbi/lgstack
/plugin install work@lgstack
```

To try it locally before publishing:

```
claude --plugin-dir ./work
```

## Not covered yet

PRs and deploy.

## Credits

Adapted from [pstack](https://github.com/cursor/plugins/tree/main/pstack) (Lauren Tan, via Michael Denyer's [pstack-claude](https://github.com/michael-denyer/pstack-claude)) and [Matt Pocock's skills](https://github.com/mattpocock/skills). See [NOTICE.md](NOTICE.md). MIT licensed.
