# Huntress CTF 2026, Day 16

Five challenges: a warm-up, "Return to Sender" (e-mail metadata), obfuscation equivalence in the script lane, the JNI boundary on Android, and a WebAssembly optimization-equivalence binary bundle. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Mastery open (78 sets) | three decoder fixes |
| Return to Sender (metadata) | an e-mail field open | new .eml parser |
| Obfuscation Equivalence (script analysis) | solved | nothing |
| JNI Boundary (mobile) | solved | nothing |
| WebAssembly Optimization Equivalence (binary) | 158 of 160 accepted | wasm architecture spellings |

## 1. Warm-up Mastery

Three separate problems: one line was Morse spelling a hex string that is itself text, and only the hex string was offered; a byte decoder turned a reversed-text line into non-ASCII junk that outranked the real text readings; and the Caesar vote preferred a shift for a symbol-heavy ROT47 line. Fixes: follow the hex chain after Morse, restrict fragment readings to ASCII, and penalise shift votes on fragments with few letters.

## 2. Return to Sender

New parser for RFC 5322 e-mail: every header (raw and bare spellings), address parts, dates, hops, MIME parts, attachments and body.

## 3. WebAssembly

Wasm modules fell through to the CPU architecture list, none of which the server accepts for a wasm file. The solver now offers `wasm32` first and then the other plausible spellings; the server took it.

## Lessons from Day 16

- A decoder that returns junk can beat a correct one. Constrain outputs to plausible text.
- Fall-through defaults are where new file types go wrong.
