# Huntress CTF 2026, Day 9

Five challenges: a warm-up built from escape sequences, PE header metadata, a large script-analysis pack about stager relationships, an Android app that trusts everyone, and a Nim-built binary bundle. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Single and Time Trial passed, Mastery had no answers | strip quotes around one line |
| Built Different (metadata) | one field of the Single Solve open | add a PE/COFF header reader |
| Stager Relationships (script analysis) | solved in all three tiers | nothing |
| Trust Everyone (mobile) | solved in all three tiers | nothing |
| Nim Generated Runtime (binary) | Single Solve one field short, then the Time Trial one field short | locate an encoded config, answer `[]` for a missing symbol |

I started the runner at 9:35 and had the first log a few minutes later. Two small rounds of fixes followed, and the whole day was solved by about 10:10.

## 1. Warm-up (escape sequences)

**Prompt:** "These characters would very much like to escape." The hint points at escape sequences.

**Approach:** same as Days 7 and 8. The Mastery is 43 blobs of numbered lines, each line a fragment of one sentence in a different encoding (base64, rot13, hex, binary, decimal, octal, percent-encoding, HTML entities). The fragments, decoded and joined in order, end in the answer slug.

**What went wrong:** the new ninth line is a JSON-style string: a run of `\uXXXX` escapes wrapped in double quotes. The unicode decoder handled the escapes, but the quote characters stayed in the joined text. The slug pattern is anchored by word boundaries and hyphens, so a stray quote right after the last digit broke the match and no blob produced an answer.

**Fix:** strip one pair of matching quotes around a line before decoding it. All 43 blobs then decode, and the answers for earlier days' blobs are byte-for-byte unchanged.

**Lesson:** after a decode "works", look at the characters at the edges of the result. One stray delimiter is enough to hide a correct answer.

## 2. Built Different (PE metadata)

**Prompt:** facts about files in an evidence folder, as on earlier metadata days (image dimensions, PDF version, ZIP member list, a plain text note) plus a Windows executable.

**Approach:** the earlier parsers carried over. For the executable the question asks for one COFF header field, the machine type, as a lowercase hex value with a `0x` prefix.

**What went wrong:** the metadata solver had a reader for images, PDFs, ZIPs and a few media formats, and none for PE files, so that field had no candidate.

**Fix:** a small PE reader. It follows the pointer at offset `0x3C` to the `PE\0\0` signature, then reads the COFF header (machine, section count, timestamp, symbol table pointer, optional header size, characteristics) and the standard optional-header fields (magic, entry point, image base, subsystem, DLL characteristics and so on). Each is offered under several name spellings and in both four-digit-padded and minimal hex, because the question only says "lowercase hexadecimal with a 0x prefix".

**Lesson:** when a new file type shows up, build the reader for the whole header, not for the one field the first question asks. Neighbouring fields are likely tomorrow's questions.

## 3. Stager Relationships (script analysis)

**Prompt:** a pack of scripts whose roles relate to each other as stagers, downloaders and launchers. The Time Trial had 695 questions and the Mastery 3475.

**Approach:** the pipeline from Days 4 to 8: decode each sample's plan statically, answer role, adapter, campaign, path and stage questions from it, and fill the ATT&CK technique questions from a table the runner learns from the server's accept/reject replies.

**Result:** solved in all three tiers on the first run, with no change to the code. About twenty technique and expansion-style fields needed an alternative spelling, which the runner found by trying the next candidate. The Mastery finished in 72 seconds, including 33 rate-limit pauses that the runner waited out.

**Lesson:** the fix-the-schema-from-the-data approach from Day 7 paid off: a brand-new pack just worked.

## 4. Trust Everyone (mobile)

**Approach:** unpack the APK without installing it and read the manifest, the DEX strings and the bundled puzzle asset, whose answer code carries a named codec.

**Result:** solved in all three tiers on the first run. The byte-wise shift handling added on Day 7 is still doing its job.

## 5. Nim Generated Runtime (binary)

**Prompt:** facts about a program compiled from Nim, inside the usual bundle of earlier-day binaries (native Linux executables, a .NET assembly, Java archives): hashes, container formats, ELF facts and the embedded configuration (the HCFG layout: marker, version, XOR key, length, CRC32, XOR-encoded JSON).

**What went wrong (two layers):**
- *Single Solve, one field open.* The question asked for the offset of the config in the file. In this binary the HCFG blob is not stored raw. It sits in the data section as a base64 string, and the literal `HCFG` that appears in the file is only a string from the program's own check. The solver already decoded the blob but did not remember where the string came from. Fix: record the position of the string that decodes to a valid blob (checked with the CRC32), and offer that, the offset inside the decoded blob and the end of the string as candidates, in that order.
- *Time Trial, one field open.* The Day 1 file in the bundle changes from attempt to attempt, and this variant was stripped: no symbol table at all. The question says to disassemble the `recover_code` function, "and if that symbol is absent, submit `[]`". My code returned nothing instead of an empty list. Fix: when the symbol is missing, answer `[]`.

**Lesson:** read the sentence in the question that covers the empty case. It is part of the specification, and a Time Trial variant will find it for you.

## Lessons from Day 9

- A decode that "works" can still leave a stray delimiter that hides the answer.
- Build a reader for the whole header of a new file type, not just the first asked field.
- A schema derived from the data and a runner that retries alternatives solved two challenges with no changes at all.
- When the question describes an empty case, handle it explicitly.
- Time Trial variants differ in ways the Single Solve never shows. Test with more than one sample.
- Unattended runs, with a pause between rounds, make a 30 minute fix cycle possible.
