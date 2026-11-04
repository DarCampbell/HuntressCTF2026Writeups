# Huntress CTF 2026, Day 23

Five regular challenges: a warm-up, "Page Me Maybe" (metadata), a game-resource loader in the script lane, "Hook, Line and Signal" on Android and an anti-analysis-recognition binary bundle, plus three web challenges (template injection, replay, mass assignment). **This is a status note, not a finished write-up.**

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Single Solve not solved | needs work |
| Page Me Maybe (metadata) | Single Solve not solved | needs work |
| Game Resource Loader (script analysis) | Single Solve solved; Time Trial 400 of 1495 | needs work |
| Hook, Line and Signal (mobile) | Time Trial 0 of 1 | needs work |
| Anti-Analysis Recognition (binary) | solved in all three tiers | nothing |
| Web (3 challenges) | not solved | needs work |

## 1. What worked

The binary pack (276 of 276 in the Time Trial, 1932 of 1932 in the Mastery) and the script Single Solve.

## 2. What did not

The warm-up and metadata Single Solves were not solved, so their Time Trials stayed locked behind the server's prerequisite (HTTP 409). The script Time Trial was far from complete: most of its 1495 questions were unanswered, which points at a family of per-sample fields with no candidate. A runner stall also led to a per-challenge watchdog that kills a stuck challenge after 20 minutes so one challenge can no longer freeze a day.

## Lessons from Day 23

- A per-challenge watchdog turns a hang into a skipped challenge.
- Locked modes mean the Single Solve is the first thing to fix.
