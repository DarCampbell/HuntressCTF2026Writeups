# Huntress CTF 2026, Day 20

Five regular challenges: a warm-up, "Universal Appeal" (metadata), a Jenkins pipeline intrusion in the script lane, "Loaded Later" on Android and a resources-and-overlays binary bundle, plus two web-exploitation challenges (local file inclusion and broken access control) that had been carried over unsolved. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up, metadata, script, mobile, binary | solved in all tiers | nothing recorded |
| Web: LFI | Single Solve and Time Trial solved, Mastery open | several rounds of web fixes |
| Web: Broken Access Control | Single Solve and Time Trial solved, Mastery open | route-inventory handling |

## 1. The web track

These were the first web challenges the bot attacked live, with no dedicated solver at first. The progression is the interesting part:

- A dedicated LFI solver and a broken-access solver were written from the challenge descriptions.
- Both only probed the root of the target; the apps are mounted under a hex path from the start reply. They now use the assigned route paths, `<form>` fields, path-style endpoints and followed links.
- A run appeared to hang after the download. A whole-target wall-clock budget, short connect/read timeouts and a bounded fan-out fixed that.
- An unsolved target now writes a page-capture file. Reading that capture from the real page showed the local-file hit was wrapped in HTML and being discarded; recovering it by a word-diff against the baseline page solved the LFI Single Solve and Time Trial on the live server.
- The broken-access app exposed a route inventory; the solver now parses it and expands the templated routes.

## 2. Mastery bundles can mix genres

The Masteries paired each challenge's own genre with a second target of a different genre. All dedicated web solvers now fall back to the all-technique lane engine when their own technique finds nothing. The same stance applies to other lanes.

## Lessons from Day 20

- Capture the real page when a web solver fails, then design against it instead of guessing.
- Bound every network step with a budget and short timeouts.
- A Mastery bundle can mix genres: try every technique per delivery.
