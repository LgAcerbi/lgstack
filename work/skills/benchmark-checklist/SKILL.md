---
name: benchmark-checklist
description: Vet a performance number before reporting or acting on it (limiter, tuning, physical limits, errors, repeatability, relevance, and whether the work actually happened). Use when you run a benchmark, or claim a speedup or regression you measured. Stage 4 (Test). Skip it when the change makes no performance claim.
---

# Benchmark checklist

Stage 4 (Test). Use this whenever you produce a performance number: a before-and-after, a regression claim, or a choice between libraries or configs. Answer each question with evidence from a run, not a guess about the code.

For a quick ballpark the user asked for, one run is enough. Still check questions 4 and 7, and say it is one run. A choice between options is never a ballpark.

## Before running anything

- Write down the claim you expect to make, in the words you would ship ("`GET /users` is 30% faster at p50 with 10k users"). The questions test that sentence.
- Read the measurement script. Note what it times, what it counts and what it ignores.
- Check whether the machine is busy (Task Manager or `Get-Process | Sort-Object CPU -Descending | Select-Object -First 5` on Windows, `uptime` elsewhere). If you can't quiet it, interleave the two sides so both see the same noise, and say so.

## The questions

1. **Why not double?** Name the limiter: what stops it from going twice as fast. Profile in a run you don't report, since profilers slow the work down (`node --cpu-prof`, `clinic`, CPU per process). Check the load generator too (`autocannon`, `k6`): if it saturates first, you measured the load generator. If a change didn't move the number, the limiter explains why.
2. **Was it tuned?** Run every side the way production runs it: production build and `NODE_ENV`, same versions, same data, caches as warm or cold as production sees them. If one side runs on defaults, you compared configurations, not implementations.
3. **Did it break limits?** Do the arithmetic. Requests per second times work per request against your cores, bytes per second against bandwidth, time saved against the time the changed piece took. Removing a piece that takes 10% of the run can make it at most about 11% faster. A result past a limit measured something else: a cache, a no-op or a bug.
4. **Did it error?** Count failures and non-2xx responses, and check outputs are correct, not just present. Errors are often fast and timeouts slow. If the script doesn't count errors, add the count.
5. **Does it reproduce?** Run each side at least 5 times, alternating (A, B, A, B...) so warmup and drift favor neither. Report the median and range. A gap smaller than the run-to-run spread is no measurable difference.
6. **Does it matter?** Next to any micro result, measure the end-to-end path a user waits on, with realistic data sizes and concurrency, and report the micro result as a share of the whole.
7. **Did it even happen?** Confirm the work ran inside the timed region: the request reached the handler, the data was read, the result was used. Promises nobody awaits and results nobody uses produce numbers for work that never happened.

## Report

- Lead with the verdict: faster, slower, no measurable difference, or inconclusive.
- Give the number with its unit, run count, range and limiter. For example: "p50 41 ms → 33 ms, median of 7 runs per side, range 32–35 ms after, bound by JSON serialization."
- Call it inconclusive when you can't name the limiter, a side ran untuned, or you couldn't check questions 4 and 7. Name the gap.
