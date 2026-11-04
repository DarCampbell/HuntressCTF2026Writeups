# Huntress CTF 2026, Day 12

Five challenges: a warm-up, "Managed Expectations" (.NET metadata), an AutoIt workflow pair in the script lane, SQLite assumptions on Android, and a Pascal archive stager binary bundle. This was the first day the fix loop ran live. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Time Trial and Mastery refused by the server | handle the server's no-fresh-variant reply |
| Managed Expectations (metadata) | one .NET field open | add a .NET metadata reader |
| AutoIt Workflow Pair (script analysis) | solved | nothing |
| SQLite Assumptions (mobile) | solved | nothing |
| Pascal Archive Stager (binary) | config offset open, Mastery no reply | more offset spellings, time-box the spelling probe |

## 1. Metadata and binary fixes

The metadata lane had no .NET reader, so a module id question had no candidate. The solver gained a reader for the .NET metadata (module id, assembly identity, table counts, streams). For the binary bundle the config offset question accepted only one of several possible meanings of "offset", so the solver now offers the payload start, the virtual address, the RVA and the section-relative value as alternatives.

A third problem was timing: after solving, a spelling probe re-parsed 95 MB bundles and could miss the 30 second clock. Candidate listing is now time-boxed in timed runs.

## 2. The server says no more fresh variants

Retrying a Time Trial or Mastery eventually returns HTTP 409 `retry_variants_unavailable`. The bot treated that as a failure. The rules say practice then continues on the last artifact with no timer. The bot now switches to practice mode and reuses the files on disk.

The first version still failed: practice replies came back `expired` and were counted as wrong answers. The saved raw replies showed each one carried `next_op: practice`, an operation not in the API description. The bot now calls it, uses the attempt it returns, saves unusual replies for diagnosis, and counts `expired` as no reply rather than wrong.

## 3. Result

The server showed all 15 modes solved at the end of the day, after four small fix rounds.

## Lessons from Day 12

- Save the raw server reply when something odd happens. The answer was in the replies all along.
- Read the error you get back: `next_op` told us the missing operation.
- Bound every optional step by the clock.
