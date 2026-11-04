# Huntress CTF 2026, Day 11

Five challenges: a warm-up, "What's My Name Again" (Windows executable metadata), native API semantics in the script lane, an Android app with local secrets, and a D-language collection planner binary bundle. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | solved | nothing |
| What's My Name Again (metadata) | three tiers open after the first run | Windows version-resource reader |
| Native API Semantics (script analysis) | solved | nothing |
| Local Secrets (mobile) | solved | nothing |
| D Collection Planner (binary) | solved | nothing |

## 1. What's My Name Again (metadata)

The questions asked for strings that live in a Windows executable's version resource (company name, file description, original filename, product version and similar).

**What went wrong:** the metadata solver had no reader for the `RT_VERSION` resource, and a rule meant for file-attribute questions wrongly claimed question ids that merely ended in `-filename`, so those questions were answered from the wrong source.

**Fix:** a reader for the version resource strings and the fixed numeric block, and a tighter test for which ids are really file attributes. A before/after comparison over saved evidence showed only the intended Day 11 question changing; the day was fully solved on the next run.

## 2. The other four

Solved with no changes. The same fix session also carried a Day 12 follow-up (see that write-up).

## Lessons from Day 11

- Pull strings from the structure where they live (here the version resource), not from a generic scan.
- Run a before/after comparison over saved evidence so a new rule can't quietly change older answers.
