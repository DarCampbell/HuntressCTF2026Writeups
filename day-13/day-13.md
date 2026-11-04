# Huntress CTF 2026, Day 13

Five challenges: a warm-up, "Product Placement" (Windows Installer metadata), an archive delivery chain in the script lane, keystore misuse on Android, and a Crystal DNS configuration binary bundle. Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up | Mastery open (60 of 63 sets) | rank text readings, vote per line |
| Product Placement (metadata) | Single Solve had no candidate | MSI (OLE compound file) parser |
| Archive Delivery Chain (script analysis) | solved | nothing |
| Keystore Misuse (mobile) | solved | nothing |
| Crystal DNS Configuration (binary) | solved | nothing |

## 1. Warm-up Mastery

The Mastery blobs are numbered lines, each a fragment of one sentence in its own encoding. The last lines used text ciphers: the final line atbash, the one before it reversed. The bot tried ROT13 first for every text line, so most sets came out wrong.

**Fix:** score each text reading (rot13, reverse, rot47, atbash, plain) with an English bigram model and, for sets of three or more blobs, vote per line number across all blobs, since one recipe encodes line N the same way in every blob.

## 2. Product Placement

The metadata lane had no parser for `.msi` files. I added a pure-Python OLE reader that reads the Property table and the SummaryInformation stream and offers product, upgrade and package code GUIDs, name, version and manufacturer. The time trial was locked behind the Single Solve (HTTP 409) until this passed.

## Lessons from Day 13

- When one recipe generates many samples, vote across samples instead of judging each alone.
- A locked Time Trial is a signal to fix the Single Solve first.
