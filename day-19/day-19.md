# Huntress CTF 2026, Day 19

Five challenges: a warm-up, "Little Endian on the Prairie" (ELF header metadata), a managed-assembly bridge in the script lane, a reflection maze on Android, and a static-unpacking binary bundle. The runner list also carried the first three web-exploitation challenges from the web track. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | open | accept UTF-16 text |
| Little Endian on the Prairie (metadata) | ELF header field open | ELF header parser |
| Managed Assembly Bridge (script analysis) | solved | nothing |
| Reflection Maze (mobile) | solved | nothing |
| Static Unpacking (binary) | config and overlay fields open | PK19 packed-overlay decoder |

## 1. Static Unpacking

The ELF carries a packed overlay with a per-byte rotate-right unpack and a SHA-256 check, and the HCFG record sits inside it. No decoder and no overlay field existed. The new `overlay_info` finds the PKnn or RSnn magic, verifies the digest, and feeds the normal config reader; overlay offset and size are offered as candidates.

## 2. Metadata and warm-up

An ELF header parser (machine and the other header fields, in plain, sized and hex spellings). The warm-up hex blob decoded to UTF-16 with a byte-order mark, which the printable-text check rejected because it only accepted UTF-8.

## 3. Web

The first web challenges (backup exposure, IDOR, SQL injection) were included in the run from this day, using the existing web lane.

## Lessons from Day 19

- Verify before trusting: the overlay's SHA-256 turned a guess into a certainty.
- Printable-text checks should accept every plausible encoding.
