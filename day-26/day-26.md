# Huntress CTF 2026, Day 26

Seven challenges: a warm-up, "Before You Go" (metadata), "BEAM Message Flow" (script analysis), "Split Decision" on Android, a crash-triage binary pack and two web challenges. **This is a status note, not a finished write-up.** Only part of this day's run was logged.

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Single Solve not solved | needs work |
| Before You Go (metadata) | Time Trial 26 of 31 | five zip-related fields rejected |
| BEAM Message Flow (script analysis) | run logged but not finished | unknown |
| Split Decision, Crash Triage, Web (4 challenges) | no result in the log | unknown |

## 1. What the log shows

The metadata Time Trial reached 26 of 31, and the rejected fields were zip central-directory member lists, which points at ordering or formatting of a JSON array. The script run was mid-way, learning technique ids from server replies, when the log ends.

## 2. Note

Day 27 has no runner log at all, so it is not written up.

## Lessons from Day 26

- Check ordering and formatting of list-valued answers, as on Day 15.
