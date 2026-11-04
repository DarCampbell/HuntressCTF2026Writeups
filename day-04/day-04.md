# Huntress CTF 2026, Day 4

Five challenges: a Batch/JScript/VBScript/PowerShell script pack, an Android PendingIntent puzzle, JPEG metadata forensics, a Go binary and a layered-encoding warm-up. As on Day 3, each had a Single Solve, a Time Trial and a Mastery tier.

> Status when written: every Single Solve and Time Trial is solved. The last two Mastery runs (Dim and Dimmer, Two of a Kind) were still being finalised. [Review: update this line once both show a flag.]

## What changed from Day 3

Day 3 taught me the format of the problem: the timed tiers are a speed test of a *static analyser*, not of a person. Day 4 made the analyser harder in three ways:
- **More languages for the same behaviour.** The Day 3 samples were Batch and JScript. Day 4 added **VBScript** and asked how the *interpreter* sees the construct, not only what the plan says.
- **More ways to say the same thing.** The question set grew to 231 (Single and Time Trial) and 462 (Mastery, two full sets at once).
- **The server became part of the problem.** A burst of answers can trip the rate limit, and a request can time out *after* the server accepted it.

Ground rules were unchanged: static analysis only, nothing from the evidence is ever executed, the shared event password unlocks the archives.

## 1. Dim and Dimmer (script analysis, shell contrast)

**Prompt:** compare how the same behaviours are expressed in Batch, JScript and VBScript, plus three PowerShell samples.

**Evidence:** nine behaviour roles (benign utility, stager, dropper, downloader, uploader, launcher, impact, staged chain, relay), each as `.cmd/.bat`, `.js` and `.vbs`, plus resource, collection and impact PowerShell samples.

**How the VBScript files hide their plan**
- A `Material_*` constant holds an encoded operation plan (`v=1|steps=...|adapter=...|campaign_id=...`).
- A `Codec_*` constant names the encoding as `representation|transform|key`. Representations seen: hex (compact, dash, colon, comma), decimal (pipe, dash, comma, space), reversed variants and base64 (literal or in chunks). Transforms: identity, add or subtract modulo 256, rolling XOR, repeating-key XOR, single-byte XOR, reverse, reverse-then-XOR.
- The label is the **authoritative answer** to the "which representation / transform" questions. The decoder only needs to *find* the plan, so it uses the label first and falls back to a search over combinations for unseen variants.

**Reading the control flow, statically**
- For VBScript, the question is which construct wraps the call to the decode function inside the recover function (`Select Case`, `Do`, `While`, `For`, `On Error`, or a direct call). I scan the function line by line with a small stack and report the innermost construct that is still open at the decode call.
- For Batch, the same idea: error-level guard, `if defined` guard, call to a subroutine, goto dispatch, or direct. The *expansion form* (percent, delayed, CALL-double, FOR-indirect) is read from how the plan variable is built.
- JScript got three style questions per file: how the inspect function is declared, how the descriptor is invoked (direct, call, apply) and how properties are accessed (dot, bracket, mixed).

**Classify by behaviour.** The ordered `steps` identify the role. Two roles (staged chain and relay) have identical steps, and the server's vocabulary for the relay file turned out to be `staged_chain` in every language. The MITRE ATT&CK technique follows the same pattern as Day 3: stable roles always map to the same technique, while launcher, chain and relay follow *the language of the file in front of you* (T1059.003, T1059.005, T1059.007).

**What went wrong, and what fixed it**
- **Expansion form is not a clean function of the visible code.** The same code pattern was accepted as different words in different variants. I stopped trusting a single guess: the expansion questions are probed in parallel (four candidate words each) inside the window, and the accepted spelling is remembered.
- **Time budget.** On my machine one variant took 1.2 to 1.7 s to solve, out of a window of about 6 s. Fixes: cache file reads in memory, prefilter candidate transforms by checking the first and last bytes before decoding everything (chained XOR via a cumulative XOR trick), and write diagnostic dumps *after* the answers are in. Solve time dropped to roughly 0.13 s.
- **Rate limiting.** After a Time Trial, the next start or batch could come back HTTP 429. The script now retries with the server's delay and waits between attempts in proportion to how many requests the last attempt sent.
- **Stale code.** At one point my own run used a half-updated copy of the script. Check that the file you run is the file you edited.

## 2. Pending Intent Relay (mobile)

**Prompt:** a PendingIntent that is handed across a boundary and relayed.

**Approach:** unpack the APK and read the manifest and the DEX statically. The case code ships as an encoded asset, `huntress_puzzle.json`, with a codec name and optional key. The decoder tries base64, hex, single-byte XOR, reversal and ROT13 on it and accepts the result that matches the answer format. The Time Trial and Mastery tiers reuse the same solver once per evidence folder.

**Lesson:** the Single Solve of Day 3 (Intent Injection) and Day 4 shared the same asset format. Generalise the solver once, instead of writing one per challenge.

## 3. Picture This (metadata forensics)

**Prompt:** read JPEG facts from the bytes.

**Approach:** parse the JPEG marker stream by hand (SOI, APPn segments, SOF, SOS), pulling out the frame dimensions, precision and component count from SOF, and JFIF density and EXIF orientation from the APP segments. Questions look like `...-jpeg-001-jpeg-sof-dimensions-pixels`, i.e. a field of a named photo.

**Lesson:** the answer *format* matters as much as the value. I listed several spellings of the dimensions (`WxH`, `W x H`, `W,H`, ...) and let the checker's feedback pick one, then reused it for every photo of the timed tiers.

## 4. Go Runtime Fingerprints (reverse engineering)

**Prompt:** identify a Go binary's runtime fingerprints.

**Evidence:** an evidence set that bundles the Day 1 to Day 3 binaries again (ELF, PE, .NET) and adds a Go ELF of about 1.8 MB. The questions repeat the earlier ones and add architecture, format and the embedded configuration.

**Approach:** reuse the Day 3 container parsers and add the Go ELF. The configuration is an `HCFG` record (a marker, a key, a length, a CRC-32 and an XOR-encoded JSON body). Scanning for the marker finds several false hits inside machine code, so each candidate is validated by decoding it and checking the CRC before it is accepted.

**Lesson:** validate before you trust a magic number. A four-byte marker is not unique inside a multi-megabyte binary.

## 5. Two of a Kind (warm-up)

**Prompt:** decode the blob until the last line tells you the answer slug.

**Approach:** a breadth-first search over a small set of decoders (binary, hex, base64, base32, decimal, URL, ROT13, reverse), stopping at the first text that contains a slug. The Mastery tier changed the shape: each blob was a list of *numbered lines*, every line in its own encoding, to be decoded and joined in order.

**What went wrong:** one Mastery batch timed out on the network *after* the server had already accepted it. The retry then answered "CORRECT" for the previous attempt, which my script did not recognise. It now treats that reply as a solve and saves the result.

## Tooling

The same single Python driver as Day 3 (`list`, `run`, `auto`), with new solver modules for each challenge. Additions this day: VBScript decoding, a per-language knowledge file of accepted spellings, parallel probing of uncertain answers, group testing for batches with one bad answer, rate-limit-aware cooldowns, and recognition of a server-side "already correct" reply. I built it with an AI assistant (Claude) as a pair programmer.

## Takeaways
1. When a label in the evidence names the encoding, believe it. Search is for the cases where it is missing.
2. Don't trust a rule you inferred from three samples. Probe the uncertain answer set in parallel and persist what the checker accepts.
3. Budget the window: measure it, then optimise the slowest step.
4. Treat the network as unreliable. A timeout does not mean the server did not accept your answer.
