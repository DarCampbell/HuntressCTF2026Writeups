# Huntress CTF 2026, Day 28

Seven challenges: a warm-up, "Where It All Began" (metadata), a Terraform cloud timeline in the script lane, "Protocol Reconstruction" on Android, a family-clustering binary pack and two web challenges. **This is a status note, not a finished write-up.**

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Mastery stalled for 20 minutes | fixed on Day 29, see below |
| Where It All Began (metadata) | Single Solve not solved | needs work |
| Terraform Cloud Timeline (script analysis) | Single Solve solved; Time Trial 441 of 1781 | needs work |
| Protocol Reconstruction (mobile) | Time Trial 0 of 1 | needs work |
| Family Clustering (binary) | Single Solve not solved | needs work |
| Web (2 challenges) | locked behind the prerequisite | needs work |

## 1. The stall

The warm-up Mastery hit the 20-minute watchdog right after extracting its files. The cause, found on Day 29: the Mastery has about 140 sets in one flat folder, and the warm-up solver recomputed its slow per-line vote over every blob once per set (roughly 140 times). Caching the vote per folder removes the repeat, as does a short cool-down between tries instead of waiting out a long refill. The fix is in place but unconfirmed against a live Mastery at the time of writing.

## 2. Everything else

The same families of gaps as Days 23 to 25: Single Solves for metadata and binary open, and a script Time Trial with most questions unanswered.

## Lessons from Day 28

- A hang that looks like a network problem can be a quadratic loop in your own code. Profile it before blaming the server.
