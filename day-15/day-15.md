# Huntress CTF 2026, Day 15

Five challenges: a warm-up, "Word to the Wise" (Office document metadata), credential-store clues in the script lane, native-library secrets on Android, and an assembly-and-symbol-loss binary bundle. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | solved | nothing |
| Word to the Wise (metadata) | one Time Trial question open | OOXML relationships, digest-order question |
| Credential Store Clues (script analysis) | solved | nothing |
| Native Library Secrets (mobile) | solved | nothing |
| Assembly and Symbol Loss (binary) | labelled variants unanswered | read the sample labels (stripped / symbolized) |

## 1. Binary bundle

The bundle now contained the same program in stripped and symbolized forms, with the variant named in the file. Questions about symbols have different correct answers for each, so the solver now reads the label from the file name and answers each variant accordingly.

## 2. Metadata

An OOXML relationship parser was added. The Time Trial then asked for a synthesis question: the filenames sorted by the digest of the full file, as a JSON array. The solver now reads the algorithm and the ascending or descending wording from the question text, uses the file list in the question, emits compact JSON and keeps the opposite order as an alternative.

## Lessons from Day 15

- Read the wording of the question: algorithm, direction and format were all stated.
- Offer the opposite order as an alternative when the wording is ambiguous.
