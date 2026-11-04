# Huntress CTF 2026, Day 24

Five regular challenges: a warm-up, "Event Horizon" (metadata), a JSP web-shell source pack in the script lane, "Emulator Alibi" on Android and a patch-diffing binary bundle, plus two web challenges. **This is a status note, not a finished write-up.**

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Single Solve not solved | needs work |
| Event Horizon (metadata) | Single Solve not solved | needs work |
| JSP Web-Shell Source (script analysis) | Single Solve solved; Time Trial 408 of 1560 | needs work |
| Emulator Alibi (mobile) | Time Trial 0 of 1 | needs work |
| Patch Diffing (binary) | Time Trial solved; Mastery 2302 of 2304 | two fields open |
| Web (2 challenges) | not solved | needs work |

## 1. What worked

The binary pack was nearly clean: the Time Trial passed all 288 answers in 10 seconds, and the Mastery had 2302 of 2304 accepted in about 80 seconds, with two `d01_analysis_marker` fields open.

## 2. What did not

The warm-up and metadata Single Solves again blocked their Time Trials. The script pack's Time Trial answered about a quarter of its 1560 questions, so a whole family of fields had no candidate.

## 3. Note

No automated fix session is recorded in the fix log after 23 October, which is why the open items stayed open.

## Lessons from Day 24

- A nearly-clean Mastery (2302 of 2304) is a good place to look for the remaining spelling.
- Without a fix session, the gaps stay.
