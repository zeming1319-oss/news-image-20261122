# SPEC — news.png for "Sum up 22 November's world news in one image"

## Goal
`main.py` (Python 3, standard library only preferred) generates **news.png** deterministically.

## Hard requirements (grader sandbox — all MUST hold)
- Run as `python3 -P -s main.py` (isolated mode, no user site-packages). Must exit **0** on any environment.
- **No network access.** No external assets, no font downloads.
- Repository is mounted **read-only** at `/bot`; an empty writable dir is at `$MAKEFUN_OUTPUT_DIR`.
  Output path: `os.path.join(os.environ.get("MAKEFUN_OUTPUT_DIR") or os.getcwd(), "news.png")`.
- 60 s wall time, 512 MB RAM, 1 CPU. Keep the program fast (< 5 s).
- **Deterministic**: two runs must produce byte-identical news.png. No wall-clock time, no unseeded randomness,
  no locale dependence. Sort/ordering must be explicit, not set-iteration order.
- **Fonts**: NEVER hardcode system font paths. If Pillow is importable, load a font by trying a relative
  fallback chain under `os.path.join(os.path.dirname(os.path.abspath(__file__)), "fonts")` (any .ttf found there),
  then common font dirs guarded by `os.path.exists`, and finally `ImageFont.load_default()`.
  If Pillow is NOT importable, fall back to a pure-stdlib PNG writer using `zlib` + `struct`
  (raw RGBA/RGB scanlines, filter byte 0, IHDR/IDAT/IEND) drawing the same layout with a built-in 5x7
  bitmap font scaled up. Every code path must exit 0 and write news.png.
- Wrap the whole program in try/except so that even on unexpected errors it still exits 0
  (last-resort fallback: minimal valid PNG). news.png must always exist after the run.

## Image spec
- **Size: 1280 x 800 pixels**, PNG (RGB).
- Layout: dark navy background (#0d1b2a); a header band with title
  **"WORLD NEWS — 22 NOVEMBER 2026"** (title ≈ 44–52 px tall when Pillow path used) and subtitle
  "Nine stories that shaped the day" (≈ 20 px); below, a 3-column grid (3 rows), each cell a rounded
  rectangle card (lighter panel #1b263b) with a 2-digit index, a category tag word, and a headline.
- Headlines: keep each **≤ 60 characters**, wrapped to at most 3 lines per card, card text ≈ 15–17 px.
- Category tags (uppercase, short): CONFLICT, CLIMATE, MIDEAST, US POLITICS, US POLITICS, DIPLOMACY,
  UKRAINE, SPORT, US POLITICS — one per story, in order below.

## The 9 headlines (exact text, in this order)
1. [CONFLICT] Russian warship sinks in Benghazi after drone strike; Zelenskyy hails attack
2. [CLIMATE] UN climate talks collapse in Turkey as EU rejects draft omitting fossil fuels
3. [MIDEAST] Iran says air-defence missile hit a U.S. F-16 over the Gulf; Washington denies
4. [MIDEAST] Trump, back from China trip, says he will push a permanent Middle East deal
5. [US POLITICS] House Democrats win 258 seats, their best midterm result in decades
6. [DIPLOMACY] Nepal's PM Balendra Shah arrives in Delhi for a two-day state visit
7. [UKRAINE] U.S. intelligence chief: Putin is "not America's friend in Ukraine"
8. [SPORT] FIFA's Infantino wins African backing with pledge to triple funding
9. [US POLITICS] Lawmakers leave for Thanksgiving recess with year-end agenda unresolved

(You may slightly shorten a headline to fit wrapping, but keep meaning, names and numbers intact.)

## Colors
Background #0d1b2a, cards #1b263b, header accent #e0e1dd, text #f8f9fa, tag colors muted
(#e63946 CONFLICT, #2a9d8f CLIMATE, #e9c46a MIDEAST, #a8dadc US POLITICS, #cdb4db DIPLOMACY, #b5e48c UKRAINE, #90e0ef SPORT).

## Verify before finishing
1. `python3 main.py` runs twice → identical `sha256sum news.png` both times.
2. `python3 -P -s main.py` exits 0.
3. PNG header parses: 1280x800.
4. `news.png` size is a few hundred KB at most.
