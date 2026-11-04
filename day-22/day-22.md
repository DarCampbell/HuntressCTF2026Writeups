# Huntress CTF 2026, Day 22

Five regular challenges: a warm-up, "Main Character Energy" (JAR metadata), an installer script chain, "Root of Trust" on Android and a managed-native-bridge binary bundle, plus two web challenges (CSRF and file upload). Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up Mastery | open | sentence alignment, rail-fence vote |
| Main Character Energy (metadata) | JAR main-class question open | manifest reader |
| Installer Script Chain (script analysis) | solved | nothing |
| Root of Trust (mobile) | solved | nothing |
| Managed Native Bridge (binary) | wrong primary assembly, format list | pick the primary assembly properly, add pe_cli |
| Web: CSRF and file upload | not solved | baseline solvers added |

## 1. Binary

In a managed/native bundle the dependency DLL sorted first and was taken as the primary assembly, so no config was found and the format was wrong. The solver now picks the primary assembly from the runtime config stem or the presence of HCFG, and the .NET format list includes `pe_cli`. Labelled sample variants (`sample-stripped.elf`, `sample-compact.wasm`) are recognised too.

## 2. Metadata and warm-up

Reading the JAR manifest answered the main-class question. For the warm-up Mastery the fixes shared with Day 21 applied: tab-separated parsing that keeps a fragment's leading space, and per-blob message alignment.

## 3. Web

Baseline CSRF and file-upload solvers were written from the challenge concepts, with a page dump when they miss. They had not solved the live targets at the time of writing.

## Lessons from Day 22

- A sort order is not a role: choose the primary file by evidence, not by position.
- Write the baseline, and let the page dump tell you what to refine.
