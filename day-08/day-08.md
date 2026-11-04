# Huntress CTF 2026, Day 8

Five challenges: a warm-up with HTML character entities, file-attribute forensics, a cross-language pack of script samples, an Android cleartext-session app and a Swift-themed binary bundle. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | Result of the first unattended run | What had to change |
|-----------|-----------------------------------|--------------------|
| Warm-up | Single and Time Trial passed, Mastery failed | decode HTML entities, stop a slow fallback |
| What's in the Box (metadata) | Single failed, Time Trial blocked by a server error | text and file-attribute fields, then a per-pack folder lookup |
| Cross-Language IOCs (script analysis) | Single passed after a new plan format was read, Time Trial missed 5 of 615 | read a two-field codec label |
| Cleartext Session (mobile) | solved | nothing (Day 7's byte-wise shift handling carried over) |
| Swift Network Configuration (binary) | stalled on a 67 MB file | header-only reads, integer answer formats |

New this day: I ran it unattended. The runner waits for the day to open at 9:00, runs the whole day, logs everything and repeats what is still unsolved, so a fix can be picked up without anyone at the keyboard.

## 1. Warm-up (character entities)

**Prompt:** the browser "understands" the text. The hint points at HTML entities.

**Approach:** the Mastery is 38 blobs. Each blob is a list of numbered lines, and every line is a fragment of one sentence written in a different encoding: base64, rot13, hex, binary, decimal, octal, percent-encoding and, new today, HTML numeric entities. The fragments, decoded and joined in order, end in the answer slug.

**What went wrong:** the entity line was not recognised, so the join never ended in a slug. The code then fell back to a broad search over every decoder, which burned about 100 seconds per blob on a clock of seconds.

**Fix:** add entity decoding (and a few neighbours: unicode escapes, Morse), and skip the broad fallback for any blob that has the numbered-line layout, because there the right answer is a per-line decode, not a search.

**Lesson:** when the layout tells you what the answer looks like, trust it and fail fast instead of searching.

## 2. What's in the Box (file-attribute metadata)

**Prompt:** facts about files inside delivered archives: file name, size, MD5, SHA-1, SHA-256, the first sixteen bytes, whether a text file starts with a byte-order mark, plus the usual image, PDF and ZIP header questions from earlier days.

**Approach:** treat each question's id and `archive.zip -> file` hint as the pointer to the file, read the bytes and compute the fact. For ZIP questions, parse the end-of-central-directory record and the central directory by hand and list member names from it.

**What went wrong (two layers):**
- The solver had no notion of a plain text file or of "attributes of one file", so the Single Solve was missing the whole group.
- The Mastery has five packs, and I used one pack's note for all five. The pack folders were named after their archives (`...-evidence-02`), and my folder lookup only knew the plain `evidence-02` spelling. Name, size and BOM were identical in every pack, which hid the bug; only the digests differed.

**Fix:** a folder lookup that accepts any sub-folder whose name ends in the pack number, and a check that the digests of the five packs really differ.

**Lesson:** when only some fields are wrong, look at which fields are identical across variants. Those are the ones that cannot reveal a mix-up.

**Server note:** the Time Trial was blocked for a while by a server-side error (`training_rotation_invalid`) that nothing on my side could change. The runner keeps retrying, and it passed later.

## 3. Cross-Language IOCs (script analysis)

**Prompt:** a mixed pack of 600+ questions over scripts in many languages (batch, VBScript, JScript, PowerShell, PHP, Bash, Node.js, Python, hosts such as HTA and WSF).

**Approach:** each sample keeps its plan encoded in a `Material`/`Codec` pair. A day-8 addition is a comment that carries a whole analysis record, wrapped in layers (hex, base64, pieces joined by dots, hex of base64). The solver peels layers breadth-first until the text starts like a record, then answers role, language, adapter, campaign, host and stage questions from it. The ATT&CK technique questions use a table that the runner learns from the server's accept/reject replies and stores for later days.

**What went wrong:** the Time Trial left 5 of 615 open, all in one Bash sample whose label was `xor_hex|3e` — two fields where every earlier label had three. The key sat in the slot where the transform used to be.

**Fix:** normalise two-field labels of that shape into the usual representation/transform/key triple before decoding.

**Lesson:** a label format that changes by one field is still a new format. Normalise in one place instead of patching each decoder.

## 4. Cleartext Session (mobile)

**Approach:** unpack the APK without installing it and read the manifest, the DEX strings and the bundled puzzle asset. The asset carries an encoded code with a named codec.

**Result:** solved in all three tiers on the first run. The fix from Day 7 (treat a shift codec as a shift of whole byte values, not just letters) did exactly what it was written for.

## 5. Swift Network Configuration (binary)

**Prompt:** facts about a Swift-built program inside the usual bundle of earlier-day binaries: sha256, architecture, container format, ELF facts, and the embedded configuration (HCFG layout: marker, version, XOR key, length, CRC32, XOR-encoded JSON).

**What went wrong (two layers):**
- *Hang.* The Swift file is about 67 MB. My ELF symbol reader copied the rest of the file for every symbol, so reading the symbol table was quadratic and the whole binary solver stalled. The Single Solve never started answering, and the Time Trial and Mastery stay locked behind it. Fix: read the machine type straight from the header, read symbols by offset, cap how many are read.
- *Formats.* After the hang was gone, 72 of 80 fields passed. The rest failed because I sent text where the question asked for an integer (ELF class and machine as base-10 numbers), or had no value at all (entry point as lowercase hex, "has symtab", "has C++ typeinfo", "has C++ vtable", and the list of compression methods in two jar files). The question list carries an `answer_format` for each field, which the earlier code ignored.

**Fix:** compute the missing fields from the headers, and let the declared answer format reorder the candidates so an integer question tries integers first.

**Lesson:** read the answer format the server gives you. And when a file is large, check the cost of your parser before the clock is running.

## Lessons from Day 8

- Put a time limit and a size limit on every parser; a 67 MB file will arrive one day.
- When a layout tells you the shape of the answer, skip the broad search.
- Compare variants: fields that are identical across packs cannot prove a pack mix-up is absent.
- Use the answer format the server declares instead of guessing spellings.
- Normalise label formats in one place.
- Unattended runs work: wait for the day to open, log everything, retry what is open, and fix between rounds.
