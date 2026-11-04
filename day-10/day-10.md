# Huntress CTF 2026, Day 10

Five challenges: a warm-up, a metadata pack ("Where Do We Begin"), a WMI persistence reconstruction in the script lane, a network-config-split Android app, and a Zig cross-platform services binary bundle. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | solved in all three tiers | nothing |
| Where Do We Begin (metadata) | solved | nothing |
| WMI Persistence Reconstruction (script analysis) | solved | nothing |
| Network Config Split (mobile) | solved | nothing |
| Zig Cross-Platform Services (binary) | solved | nothing |

## 1. The day

The runner went through the day twice (an afternoon run and an evening re-run) and the server showed every Single Solve, Time Trial and Mastery as solved at the end. No code change for this day is recorded in the fix log, so this was the first day where the pipeline built on Days 3 to 9 simply carried a whole new set of challenges.

The parts that did the work were the same ones as before: the layered-encoding decoder for the warm-up, the file-format parsers in the metadata lane, the decode-the-plan-then-answer approach for scripts, APK unpacking for the mobile app and the HCFG config reader for the binary bundle.

## Lessons from Day 10

- A pipeline that derives answers from the data generalises: a full day passed with no fixes.
- The two-run day shows the value of a runner that skips finished modes and only redoes what is open.
