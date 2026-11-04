# Huntress CTF 2026, Day 21

Five regular challenges: a warm-up ("Under Pressure"), "Package Deal" (Android package metadata), GitHub Actions persistence in the script lane, "Debug Me If You Can" on Android, and a shared-library-boundaries binary bundle, plus three web challenges (reflected XSS, stored XSS, session handling). Each had a Single Solve, a Time Trial (a new variant per attempt) and a Mastery (several variants at once).

Ground rules unchanged: static analysis only, nothing from the evidence is ever executed. No flags or answer values appear here, only methods.

## The day in one table

| Challenge | First unattended run | What had to change |
|-----------|----------------------|--------------------|
| Warm-up Mastery | 0 of 103 sets accepted at first | compressed line fragments, then sentence alignment, then rail-fence |
| Package Deal (metadata) | APK package-name question open | Android binary-manifest reader |
| GitHub Actions Persistence (script analysis) | solved | nothing |
| Debug Me If You Can (mobile) | solved | nothing |
| Shared Library Boundaries (binary) | config fields open | read the config from the plugin member |
| Web: reflected XSS, stored XSS, session handling | not solved | see below |

## 1. Warm-up Mastery

Three rounds. First, every blob had at least one line that was a compressed payload behind a byte encoding (zlib, gzip, bz2 or lzma behind base64, hex, base32 or base85), added as a last-resort reading so it never changes a line that already decoded. That took the day from 0 to 8 of 103. Second, the numbered-line parser had swallowed a fragment's own leading space, and sentence alignment per blob needed a template of the message. Third, the rail-fence vote for the line carrying the slug's first letters was cancelled whenever one blob's fragment lacked a space or colon.

## 2. Metadata and binary

The zip parser never read `META-INF/MANIFEST.MF` and had no Android binary-manifest (AXML) reader. For the binary pack the loader is the first archive member, but the unlabelled config fields belong to the plugin library, so the config is now taken from another member when the first has none. Variant files named `sample-<label>` are recognised.

## 3. Web

The three web challenges were not solved. The lane fallback was hanging, so it was made bounded and self-adapting (a per-target budget, techniques ordered by what the landing page shows), and later runs leave the real page and forms in a capture file to design against.

## Lessons from Day 21

- Add new readings as last resorts so they cannot alter answers that already work.
- Hangs are bugs in the bot, not in the target: budget everything.
