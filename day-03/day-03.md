# Huntress CTF 2026, Day 3

Three challenges: batch/JScript/PowerShell script analysis, Android intent analysis and .NET/ELF/PE metadata reading. Each had a Single Solve, a Time Trial and a Mastery tier. All nine are solved.

## The shape of the problem

The Single Solve of each challenge is a normal analysis exercise. The Time Trial and Mastery tiers are what make the event interesting. You get a fresh variant of the evidence, a window of roughly 6 seconds (Time Trial) or 7 seconds (Mastery), and between about 24 and 318 questions. The only workable approach is to **solve the Single Solve first, understand the generator behind the samples, and turn that understanding into a static analyser** that can answer a brand-new variant faster than a human can read the first file.

Ground rules I kept for all of it:
- **Static analysis only.** Nothing from the evidence is ever executed. The samples are inert fixtures, but I treated them like malware anyway: decode, never run.
- Evidence archives are encrypted with the event's published, shared password, so extraction is part of the pipeline.
- Wrong answers are cheap in the untimed tier, but one wrong answer rejects a whole batch, so the pipeline submits in groups and isolates the bad ones (group testing).

## 1. Expansion Pack (script analysis)

**Prompt:** *cmd.exe expands percents, delays expansion, CALLs twice, and bends FOR loops into indirection. Reconstruct every variable exactly as the interpreter would see it, without executing a single line, then follow the staged chain to its final handoff.*

**Evidence:** nine behaviour "roles" (benign utility, stager, dropper, downloader, uploader, launcher, impact, staged chain, relay), each shipped as a Batch file and a JScript file, plus three PowerShell samples. The Mastery tier delivered two full sets at once (318 questions).

**What each file hides:** a pipe-separated plan (`steps`, adapter, campaign id, URLs, paths, next stage, a stage graph) stored under a stack of reversible encodings.

**Approach**
1. **Decode generically.** The encodings are layered and small in number: hex pairs, base64 (also of reversed text), decimal bytes, `%XX`, then a byte transform (XOR with a constant, rotate-left, add/sub a constant, chained XOR), then sometimes a reverse. Rather than hard-coding per file, the decoder tries every combination and accepts a result only if it looks like a plan (`v=1|steps=...`) or valid JSON.
2. **Classify by behaviour, not by name.** The ordered `steps` list (for example `acquire_remote, write_file`) identifies the role. Two roles share identical steps, so file naming is only a tiebreaker.
3. **Read the shell constructs.** Questions about the *expansion form* (percent, delayed `!var!`, CALL-double, FOR-indirect) and the *control-flow form* (error-level guard, `if defined` guard, call to a subroutine, goto dispatch, direct) are answered by pattern-matching the way the script rebuilds its variable. Several spellings had to be learned from the server's feedback in the untimed tier and then reused in the timed ones.
4. **MITRE ATT&CK mapping.** Roles with a stable meaning always map to the same technique (downloader to ingress tool transfer, impact to data encrypted for impact, and so on). Roles that hand off to another interpreter map to *the language of the file in front of you* (batch: T1059.003, JScript: T1059.007, VBScript: T1059.005).

**Lessons**
- The first run of the timed tiers failed on a *spelling* problem, not an analysis problem. Learning the accepted vocabulary during the untimed run paid off many times over.
- Make the unsure answers cheap to test. Anything the analyser is not certain of is probed in parallel inside the window while the confident answers go out as one batch.
- Measure the clock. At one point 40 percent of the window went on a slow brute-force decode, which led to a cheap prefilter (check the first and last few bytes of every candidate transform before decoding everything).

## 2. Intent Injection (mobile)

**Prompt:** *Trace attacker-controlled extras through an internal navigation boundary.*

**Evidence:** Android APKs (the Mastery tier delivers three at once, one question each). The questions are about how data from an externally reachable component flows into an internal screen.

**Approach**
- Unpack the APK and read the manifest and the activity code statically, to see which components are exported and where `Intent` extras are consumed.
- In this fixture family the final answer code is also shipped as an encoded asset inside the APK, so the pipeline extracts it directly. The intended route is the data-flow tracing from the prompt. I used the shortcut once I confirmed it matched the traced behaviour on the Single Solve.

**Lessons**
- Mastery means several independent evidence folders at once. The same solver has to be run once per folder and the answers re-keyed (`set1.`, `set2.`, `set3.`). That bug cost one attempt.

## 3. Table Manners (binary and .NET metadata)

**Prompt:** *A .NET binary is a database wearing a trench coat. Read the CLI metadata tables (methods, strings, signatures) and find where the configuration is stashed.*

**Evidence:** an evidence set of three binaries per variant (ELF, then a PE or ELF, then a .NET assembly), with 24 questions: hashes, ELF class / machine / type / endianness, section and symbol names, a disassembly check, and configuration values (campaign, domain, mutex, URL, a CRC-32 and an XOR key) plus the .NET metadata version.

**Approach**
- Parse the container formats directly (ELF headers and sections, PE headers, the CLI metadata root and its heaps) instead of depending on external tools, so it works inside a few hundred milliseconds.
- Recover the configuration constants from where the compiler places them (embedded strings, a key and its checksum), then verify them against each other.
- Several answers have more than one valid-looking spelling (`ELF64` vs `elf64`, `0x%02x` vs `%02x`, `PE/.NET`). The pipeline records the spelling the server accepts per question and reuses it.

**Lessons**
- A later variant had a PE where an ELF used to be. A generic, format-detecting parser beat the "known layout" version immediately.
- Time Trial 1 failed on a 30-second network timeout. Short timeouts, plus never starting the clock until everything is ready, fixed it.

## Tooling

A single Python script drives the event's API: it lists challenges, starts an attempt, downloads and unpacks the evidence, runs a per-challenge solver module, submits answers in groups, and writes a result file. One command (`auto`) runs a challenge's Single Solve, Time Trial and Mastery in turn. Solver modules are small files that plug into it. I built this with an AI assistant (Claude) pair-programming alongside me, and the reasoning about the file formats and the generator was a back-and-forth the whole way.

## Takeaways
1. When the tier is timed, the solution is the analyser, not the analysis. Solve the untimed version until you can explain the generator.
2. Learn the vocabulary of the checker early (accepted spellings, stable mappings) and persist it.
3. Parallelise uncertainty, batch certainty, and respect the rate limit (a burst of requests can lock you out of starting the next attempt).
4. Static analysis is enough. None of this required executing a sample.
