# Huntress CTF 2026, Day 6

Five challenges: the last layered-encoding warm-up, PDF metadata forensics, an Android content-provider puzzle, a Java archive with a bytecode loader and a Bash version of the behaviour-classification pack. Each had a Single Solve, a Time Trial (about 6 to 7 seconds) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed, the shared event password unlocks the archives. No flags or answer values appear here, only methods.

## What changed from Day 5

Day 6 was mostly a test of whether the Day 3 to 5 framework generalises. It did, with three additions:
- **Bash** as a fifth script language for the behaviour pack.
- **PDF** as a new metadata format and **Java (JAR)** as a new binary format.
- A few server-side hazards that only show up when a day is run end to end: read timeouts on submit, a temporary HTTP 503 on start and a variant that hands out a different file type than the earlier ones.

As on Day 5, the whole day runs from a single `run_day6.bat`.

## 1. Warm-up (layered encodings)

**Approach:** the same breadth-first decoder as Day 5. This time the variants used octal numerals where Day 5 used decimal, so the solver gained an octal step and a generic "try every numeral base" pass, and the numbered-fragment reassembly from the Mastery tier was kept.

**Lesson:** a solver that tries a family of encodings in order of cost beats one that matches a single pattern.

## 2. Portable Deception (PDF metadata)

**Prompt:** read facts about a set of PDFs from their bytes.

**Approach:** parse the file as text without rendering it: header version, the Info dictionary (title, author, producer, dates), page count, object count, the trailer size, `startxref`, the number of `%%EOF` markers, encryption and MediaBox. Nothing is opened in a viewer and no embedded content is run, which matters for a challenge named after PDF tricks.

**What the server taught me:** the header version is answered as a bare number, not with `%PDF-`. For other fields I listed several spellings (dates, booleans, MediaBox as list or space separated) and let the checker's feedback choose one, then reused it in the Time Trial and Mastery.

## 3. Provider Paths (mobile)

**Prompt:** an Android app with a content provider whose paths hide a code.

**Approach:** unpack the APK and read the manifest and the bundled puzzle asset. The asset carries the same codec fields as the Day 4 and Day 5 mobile challenges, so the existing decoder was extended for this day's variant rather than rewritten.

**What went wrong:** my first version of the Day 6 solver imported helper code from a sibling solver file. That works in my test setup but fails when the runner loads each solver file on its own, so the challenge scored zero answers. The fix was to make each solver file self-contained.

## 4. Java Bytecode Loader (binary)

**Prompt:** identify facts about a Java archive that loads bytecode, in an evidence set that bundles the earlier ELF, PE and .NET binaries.

**Approach:** unzip the JAR in memory and read the class constants. The embedded configuration (campaign, domain, mutex, URL, CRC and XOR key) sits as a base64 constant inside a class, with the same HCFG layout as in the native binaries: magic marker, version, XOR key, length, CRC32, then XOR-encoded JSON. The CRC is checked before the blob is trusted. The Time Trial asked 48 questions and the Mastery 96.

**What went wrong**
- Two answers in the second set were rejected on one Mastery attempt: the format and architecture of one of the files. The solver assumed every file in that slot was a Linux executable, but some variants ship a different kind of executable there. The solver now reads the header, detects the container and architecture, and lists the matching spellings.
- A Mastery attempt timed out on submit after about 12 seconds with no reply. A retry on a new variant passed. A timeout can mean the server has already counted the answers, so the runner now treats a "CORRECT" reply on the next start as a success.

## 5. Bash Behavior Graphs (script analysis)

**Prompt:** the nine behaviour roles again, now as Bash scripts with a "behaviour graph".

**Approach:** Bash files keep their plan in an encoded `Material_*` and `Codec_*` pair just like the VBScript files of Day 4, with one extra representation (percent-encoded hex). The decoded plan lists steps, stage count, adapter and cron path. The solver classifies the role from the ordered steps, answers the stage-by-stage graph questions from the plan and maps roles to MITRE ATT&CK techniques. The stable roles map to a fixed technique, and the roles that follow the carrier language use the Bash technique, which the server confirmed or corrected and the solver remembers.

**Lesson:** reading the plan from the file beats guessing from code style, because the plan is the author's own statement of what the file does.

## Lessons from Day 6

- Self-contained solvers: do not import from sibling files.
- Never assume the file type of a slot. Read the header and branch.
- Retry on temporary server errors, and treat a timeout as "unknown", not "failed".
- The pipeline from Day 3 handled five new formats with small plugins. The investment in a framework paid off.
- One command per day keeps the timed tiers independent of how fast you type.
