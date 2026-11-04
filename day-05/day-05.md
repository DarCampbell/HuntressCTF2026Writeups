# Huntress CTF 2026, Day 5

Five challenges: a layered-encoding warm-up, PNG chunk forensics, a Python malware-behaviour pack, a Rust binary and an Android backup puzzle. Each had a Single Solve, a Time Trial (about 6 to 7 seconds) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed, the shared event password unlocks the archives. No flags or answer values appear here, only methods.

## What changed from Day 4

The Day 4 pipeline was a decoder plus a classifier per language. Day 5 added **Python** as a fourth script language and **PNG** and **Rust** as new file formats, so the work was mostly about extending the framework rather than writing five new tools:
- Each challenge is a small *solver plugin* with a challenge id and a `solve()` function. The runner downloads, extracts, calls the plugin and submits in one batch.
- Anything the plugin is unsure about becomes a *candidate list*, and the runner tests the candidates in parallel inside the time window. Whatever the server accepts is remembered for the next variant.
- One command per day (`run_day5.bat`) runs Single Solve, Time Trial and Mastery for all five challenges.

## 1. By the Numbers (warm-up)

**Prompt:** a short blob that hides a message in a stack of encodings.

**Approach:** a breadth-first decoder. At each layer it tries binary, octal, decimal, hex, base64, base32, ROT13, reversal and URL decoding, and keeps any result that looks like text. It stops when a line matches the answer shape. Time Trials changed the encoding per attempt, and Mastery split the message into numbered fragments across several sets, so the decoder also reassembles fragments by their number.

**Lesson:** do not hard-code the encoding you saw first. The Day 5 and Day 6 variants differed only in which numeral base they used, so one generic decoder served both days.

## 2. Chunk Norris (PNG metadata)

**Prompt:** read facts about a set of PNG panels from their bytes.

**Approach:** walk the PNG chunk stream by hand: signature, then length, type, data and CRC for each chunk. Verify each CRC. Pull dimensions, bit depth, colour type, interlace and any text or trailing data from the chunks. Questions are named like `...-png-ihdr-dimensions-pixels`, which is a field of a named panel.

**Lesson:** the value is only half of the answer; the **spelling** matters. I offered several spellings (`WxH`, `W x H`, `W,H`) for the first uncertain question, let the checker's feedback pick one and reused it for every later panel. Images are never rendered or opened in a viewer, only parsed.

## 3. Python Carrier Lab (script analysis)

**Prompt:** the Day 4 behaviour roles again, but the carrier language is Python, alongside the Batch, JScript, VBScript and PowerShell samples.

**Approach**
- Python samples keep their operation plan in an encoded constant, the same `representation | transform | key` idea as VBScript, so the decoder from Day 4 was reused. The Python parser evaluates only literals in the AST. It never imports or runs the file.
- The role comes from the ordered steps in the decoded plan (for example "acquire remote" plus "write file" is a downloader, "decode embedded" plus "write file" is a dropper, "profile host" plus "dispatch process" is a staged chain).
- Syntax and obfuscation labels (the question asks how the file is written, not what it does) are read directly from the decoded plan where it states them.
- MITRE ATT&CK: the stable roles map to a fixed technique. For the roles that follow the carrier language, the Python mapping was a first guess that the checker confirmed or corrected, and the accepted value is stored for later variants.

**Result:** the Time Trial was 357 questions and the Mastery 714. Both were accepted in a few seconds, with about 5 percent of the answers coming from parallel guessing.

**Lesson:** when a rule is inferred from a few samples, treat it as a hypothesis, test it in the window and keep what the server accepts.

## 4. Rust Runtime Shapes (binary)

**Prompt:** identify runtime facts of a Rust binary from the file itself.

**Approach:** parse the executable header for the format, architecture and entry details, then scan for the embedded configuration blob. The blob has a magic marker, a version, an XOR key, a length and a CRC32, followed by XOR-encoded JSON. In a multi-megabyte binary the marker also appears by chance in code, so every hit is validated against the CRC before it is trusted. The Time Trial asked 40 questions and the Mastery 80, over files of 2 to 4 MB.

**What went wrong:** on one Mastery attempt two answers in the second set were rejected (format and architecture of one file). A retry on a fresh variant passed in full, so the unusual file shape is still worth a closer look.

## 5. Backup Spill (mobile)

**Prompt:** an Android backup with a hidden code.

**Approach:** read the manifest and the bundled puzzle asset statically. This day's asset used a `split` codec: the code is cut into parts and an `order` list says how to put them back together. The solver tries both readings of the order list (position takes part, or part goes to position) and the generic decoders (base64, hex, XOR, reversal) for the other days, and accepts the one that matches the code format.

**What went wrong:** the first Time Trial attempt hit a read timeout on submit and the next start returned an HTTP 503. The runner now waits and retries on a temporary server error.

## Lessons from Day 5

- Build one solver per *format*, not per challenge.
- Labels in the evidence beat inference. Use inference only to find the label.
- Verify magic numbers with a checksum.
- Treat the server as part of the system: it rate limits, it times out, and a timed-out request may already have been accepted.
- Make every day a single command so the timed tiers do not depend on typing speed.
