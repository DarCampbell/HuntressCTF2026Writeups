# Huntress CTF 2026, Day 7

Five challenges: a percent-encoded warm-up, ZIP metadata forensics, a pack of Node.js operation graphs, an Android WebView bridge and a Kotlin binary. Each had a Single Solve, a Time Trial (about 7 to 12 seconds) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed, the shared event password unlocks the archives. No flags or answer values appear here, only methods.

## What changed from Day 6

Day 6 had shown the framework generalises, so before Day 7 I reworked the runner so that a day I had never seen could be attempted from a lane-level fallback solver, with no day-specific code. That fallback got a real test on the first morning, and it was only partly good enough:

| Challenge | First run | After the fix |
|-----------|-----------|---------------|
| Warm-up | Single and Time Trial passed, Mastery failed | solved |
| Seven-Zip, No Relation | no answer for the Single Solve | solved |
| Node by Node | most of the pack answered, 19 fields open | solved |
| WebView Bridge | no answer | solved |
| Kotlin Staging Logic | one Mastery answer rejected, retry passed | solved |

Every failure was a gap in the generic code, and each one came down to something I had assumed rather than checked. They are described below.

## 1. Warm-up (percent-encoded address bar)

**Prompt:** "My address bar is having a moment." The hint points at URL (percent) encoding.

**Approach:** the Single Solve and Time Trial reuse the layered decoder from Days 4 to 6. The Mastery is 33 separate blobs. Each blob is seven numbered lines, and every line is a fragment of one sentence written in a different encoding: base64, rot13, hex, binary, decimal, octal and, as the last line, percent-encoding. The fragments decoded and joined in order form a sentence that ends with the answer slug.

**What went wrong:** three separate bugs stacked.
- The lane solver used the first blob for every question, so all 33 answers were the same wrong value.
- Octal numerals such as `040 115 141` are also valid decimal numbers, and the decimal decoder ran first. It produced printable garbage, so the join never contained a slug.
- With no slug found, the code fell back to a breadth-first search over every decoder. On a 400-byte blob that ran for 2 to 20 minutes on a 9 second clock.

**Fix:** match each question to its own blob by artifact name or set number (and refuse to guess when none matches). Detect octal first by its zero-padded three-digit tokens. Read every line its most likely way, check that the joined text reads like the expected sentence, and only then search alternatives, a few at a time. The fallback search got a hard time budget.

**Lesson:** a search with no time limit is a bug when the clock belongs to someone else.

## 2. Seven-Zip, No Relation (ZIP metadata)

**Prompt:** "Count what the archives claim, distinguish empty space from evidence, and find the census that does not add up." The first question asked for the total entry count recorded in the end-of-central-directory record.

**Approach:** parse the ZIP by hand instead of through a library. From the end of the file: find the end record, read the disk numbers, entries on this disk, total entries, central directory size and offset and comment length (and the ZIP64 variant when present). Then walk the central directory and the local headers separately and compare. The gap between what an archive claims and what is actually there is the point of the challenge, and a library that tolerates or repairs a broken archive hides exactly that gap.

**What went wrong:** my metadata solver had a field list built from Days 4 to 6 and no entry for this field, so it returned nothing and the Time Trial and Mastery stayed locked behind the unsolved Single Solve. The new parser exposes the end-record fields, walked counts, unreferenced bytes, trailing data and per-entry facts under several field-name spellings.

**Lesson:** use the library to confirm, never as the only reader, when the question is about disagreement between a file and its own header.

## 3. Node by Node (script analysis)

**Prompt:** eight Node.js operation graphs that never execute. Spot the indirect expansions and token fragmentation, then sort each graph into its role: benign, staging, dropping, downloading, uploading, launching, scheduled or multi-stage. The Single Solve had 469 questions.

**Approach:** like the VBScript and Bash packs, each file keeps its plan in an encoded `Material_*` and `Codec_*` pair that says `representation|transform|key`. The solver decodes the plan statically and answers the role, adapter, campaign, stage count, steps, URL and paths from it. Day 7 added two Bash files for "advanced syntax" (indirect expansion and token fragmentation) with feature and primitive questions.

**What went wrong:** three of the eight Node files were not decoded, because they used representations I had not seen: a JSON array of hex strings and a run of escaped byte values. The Bash questions used field names (`feature`, `primitive`) that my question parser did not recognise, and the Node files used `.mjs`, an extension the older code did not handle.

**Fix:** add the two representations, and build the list of known field names from the keys of the decoded plans themselves instead of from a hand-written list. A new pack with new field names is then recognised as soon as its files decode. Side-by-side runs against the Day 4 to 6 packs showed no changes to the old answers.

**Lesson:** derive the schema from the data. A hard-coded field list is a bet that the next day looks like the last.

## 4. WebView Bridge (mobile)

**Prompt:** cross a challenge-owned WebView bridge through an untrusted origin.

**Approach:** unpack the APK and read three things without installing it: the manifest, the DEX strings (bridge class, the loopback evidence URL and the day's topic) and the bundled puzzle asset. As on earlier days the asset carries an encoded answer code with a named codec.

**What went wrong:** the codec was named "caesar" with a numeric key, and my decoder treated that as the classic letters-only rotation. The code is made of digits and a dash, so nothing matched. This version moves every byte value by the key, not only letters.

**Fix:** for any codec whose name suggests a shift, also try shifting whole byte values by the key in both directions, and try every single-byte shift on the first layer when no key is given. That also covers the next day's variant, whatever shift it picks.

**Lesson:** a name is a hint, not a specification. Test what a codec does to the bytes.

## 5. Kotlin Staging Logic (binary)

**Prompt:** facts about a Kotlin-built program, inside the usual bundle of earlier-day binaries (native Linux executables, a .NET assembly, Java archives).

**Approach:** the same pipeline as Day 6. Read the container from the header, take the architecture, and decode the embedded configuration, which keeps the HCFG layout (marker, version, XOR key, length, CRC32, XOR-encoded JSON). The CRC is checked before the blob is trusted. The Mastery asked 112 questions across two evidence sets.

**What went wrong:** the Mastery needed two attempts. One answer, the container format of a Go-runtime file, was rejected in one variant. The solver had learned the spelling "elf" from the Single Solve and then reused it for a slot that, in that variant, held a different kind of executable. A remembered spelling was overriding what the header said.

**Fix:** a remembered spelling is only offered when it belongs to the container type actually found in the file. Otherwise the candidates for that type come first.

**Lesson:** memory of what the server accepted is only valid for the same kind of input.

## Lessons from Day 7

- Never assume the contents of a slot: read the header and branch (again).
- Derive field names and schemas from the evidence instead of listing them in advance.
- Put a time budget on every fallback search that runs while a clock is ticking.
- Treat names like "caesar" as hints and verify the transform on the bytes.
- A learned answer format must be tied to the type of input it was learned on.
- A generic fallback solver is worth building: on the first run it answered most of the day (441 of 469 Node fields, 111 of 112 binary Mastery answers) with no day-specific code, and it narrowed the rest to a few small, fixable gaps.
- One command per day (`run_day7.bat`) still holds up.
