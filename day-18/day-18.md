# Huntress CTF 2026, Day 18

Five challenges: a warm-up, "Frame of Reference" (video metadata), "Process Story" (Google Apps Script), encrypted strings on Android, and a control-flow-reconstruction binary bundle. The enum lesson from Day 17 was applied here. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | open | rail-fence (zigzag) transposition decoder |
| Frame of Reference (metadata) | video dimensions open | MP4 video-track parser |
| Process Story (script analysis) | 45 Apps Script questions open | find the plan in joined string arrays |
| Encrypted Strings (mobile) | solved | nothing |
| Control Flow Reconstruction (binary) | two enum questions open (also Day 17's) | enum-driven answers |

## 1. Warm-up

A three-rail zigzag transposition. The new decoder tries 2 to 6 rails and keeps the best English reading.

## 2. Script analysis

Google Apps Script files keep the plan in string arrays that are joined at run time. The marker line only says `SPLIT`, so the payload was never found. The plan finder now falls back to joined string arrays or long literals when there is no inline payload.

## 3. Binary

`hcfg_storage` is now answered from the question's own `accepted_values`: the solver classifies each sample with the ordered rule (base64 carrier, exact `.hcfg` section, wasm data segment, other mapped section with code evidence, otherwise raw) and offers only listed tokens. This closed both Day 17 and Day 18 items.

## 4. Result

After a few re-runs through the day the server showed all 15 modes solved.

## Lessons from Day 18

- Applying a lesson from the previous day immediately closed two days at once.
- When a plan is split across containers, join the pieces before looking for a marker.
