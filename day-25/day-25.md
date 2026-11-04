# Huntress CTF 2026, Day 25

Five regular challenges: a warm-up, "Hive Mind" (metadata), "Phoenix Stage Chain" (script analysis), "Integrity Signals" on Android and a synthetic-vulnerability root-cause binary bundle, plus three web challenges. **This is a status note, not a finished write-up.**

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Single Solve not solved | needs work |
| Hive Mind (metadata) | Time Trial 24 of 30 | six fields open |
| Phoenix Stage Chain (script analysis) | stopped by the 20-minute watchdog | needs a look |
| Integrity Signals (mobile) | Time Trial 0 of 1 | needs work |
| Synthetic Vulnerability Root Cause (binary) | Time Trial 300 of 300 and Mastery solved | nothing |
| Web (3 challenges) | one Time Trial solved; its Mastery 0 of 4 | needs work |

## 1. What worked

The binary pack (Time Trial 300 of 300, then the Mastery). On the web side, one challenge's Time Trial was accepted.

## 2. What did not

The script pack ran until the watchdog stopped it at 20 minutes, which the Day 23 watchdog exists for. The metadata Time Trial was close (24 of 30) with six filename-style fields rejected, which suggests a spelling issue.

## 3. Note

As on Day 24, no automated fix session is recorded after 23 October.

## Lessons from Day 25

- The watchdog worked as designed: it skipped a stuck challenge and let the rest of the day run.
