# Huntress CTF 2026, Day 17

Five challenges: a warm-up, "Hear Me Out" (audio metadata), a download graph in the script lane, name collapse on Android, and an encoded-configuration-recovery binary bundle. This was the hardest fix day so far and the first to hit the attempt cap. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Single Solve and Mastery open | a/b nibble text decoder |
| Hear Me Out (metadata) | audio fields had no parser | media parser for WAV/FLAC/OGG/MP3/MP4/AIFF/MKV |
| Download Graph (script analysis) | solved | nothing |
| Name Collapse (mobile) | unanswered | indexed codec (data string plus order permutation) |
| Encoded Configuration Recovery (binary) | 202 of 204 accepted | needed a human; fixed on Day 18 |

## 1. Warm-up

The blob was a Baconian-style a/b text: five-symbol groups read as nibbles, in either polarity. A decoder was added and then wired into the per-line reader, because the Mastery carried the same text as one numbered line.

## 2. Metadata and mobile

The audio pack asked ffprobe-style fields (codec names from the WAV format tag, sample rates and so on), so a media parser went in with ffprobe spellings first. The Android puzzle used an indexed codec: a single data string reordered by a permutation, handled in both directions.

## 3. The binary pack and the attempt cap

The pack bundled the earlier days' binaries, with 204 questions. Several fixes landed (analysis fields from older days, section names with and without the null section), and 202 were accepted. Two `hcfg_storage` questions stayed open. I guessed about 53 free-text spellings across three attempts and all were refused, so the cap was reached and the session wrote "needs human".

The root cause, found on the last session: these are enum questions. The question text lists the accepted tokens and an ordered decision rule (base64 carrier, then a literal HCFG in a section named exactly `.hcfg`, then a WebAssembly data segment, then another mapped section referenced by code, otherwise raw). The solver ignored the accepted-values list and guessed. The fix itself shipped on Day 18.

## Lessons from Day 17

- If the question gives a list of accepted values and a rule, read them. Guessing 53 spellings was the wrong tool.
- An attempt cap that forces a stop and a clear note is worth having: it made the real cause visible.
