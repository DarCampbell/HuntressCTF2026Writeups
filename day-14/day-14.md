# Huntress CTF 2026, Day 14

Five challenges: a warm-up, "Shortcut Home" (Windows .lnk metadata), MSBuild trusted-project abuse in the script lane, predictable crypto on Android, and an Objective-C persistence descriptor binary bundle. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Mastery open | add Caesar shift readings |
| Shortcut Home (metadata) | no candidate | new .lnk parser |
| MSBuild Trusted Project Abuse (script analysis) | solved | nothing |
| Predictable Crypto (mobile) | solved | nothing |
| Objective-C Persistence Descriptor (binary) | solved | nothing |

## 1. Warm-up Mastery

The last lines of each blob used a Caesar shift of 7, which the per-line readings (ROT13, reverse, ROT47, atbash) did not cover. I added shifts 1 to 25 (except 13) to the line vote, and a shift only wins with a clear margin so ordinary text keeps its first reading.

## 2. Shortcut Home

A Windows shortcut file has a structured header, an ID list and a LinkInfo block. The new parser extracts target path, working directory, arguments, name, icon, show command and drive information.

## Lessons from Day 14

- Gate a new reading behind a clear margin so it can't disturb answers that already worked.
- New file type, new parser: same pattern as Days 11 and 13.
