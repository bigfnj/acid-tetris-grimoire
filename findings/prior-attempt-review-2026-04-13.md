# Prior Attempt Review

Date: 2026-04-13
Archive reviewed: `Decompilation.Effort/prior/AcidTetris_v0.1.zip`

## Scope

This review examined the prior reverse-engineering attempt for informational value only.

Important rule followed during this review:

- no prior scripts were imported into the active workflow
- no prior extracted assets were adopted as authoritative outputs
- any useful claim was treated as a hypothesis until checked against the current project data

## Bottom Line

Yes, the prior project contains useful information.

The most valuable parts were:

- format hypotheses for several early `ATET.DAT` chunks
- a custom 4-mode RLE description
- an extracted strings dump from the executable
- some annotated function-name hypotheses and address mapping

However, the prior project also contains contradictions and inferred reimplementation details that should **not** be treated as ground truth without verification.

## What Was Independently Verified

The following prior claims were checked against the current raw chunks from `Original.Game` and are now useful to this effort.

### Verified: 4-mode RLE compression exists

The prior project described a 4-mode RLE scheme:

- 2-byte header: `color1`, `color2`
- control byte: `MMCCCCCC`
- mode 0: literal copy
- mode 1: fill with `color2`
- mode 2: fill with next byte
- mode 3: fill with `color1`

This was independently tested against our current raw archive chunks and it successfully decoded multiple entries.

### Verified: chunk 7 is RLE-compressed, not raw image data

Our current chunk:

- `chunk 7` = `80514` bytes
- first dword = `80510`
- first dword equals `length - 4`

Using the prior RLE description, chunk `7` cleanly decompresses to:

- `229376` bytes

Important consequence:

- `229376` is exactly `1024 * 224`
- this strongly explains why our earlier brute-force `320x240` visualizations failed

This is one of the most useful takeaways from the prior attempt.

### Verified: chunk 8 is RLE-compressed `GAME OVER`

Our current chunk `8`:

- first dword = `1473`
- decompresses cleanly to `4608` bytes

`4608` is exactly:

- `128 * 36`

A current independent render confirms this chunk is the `GAME OVER` overlay.

### Verified: chunk 6 is RLE-compressed background pattern data

Our current chunk `6`:

- starts with `35000` then `7327`
- decompresses cleanly to `35000` bytes

`35000` is exactly:

- `200 * 175`

A current independent render shows this is the repeating yellow smiley background tile/pattern data.

### Verified: chunk 5 contains raw metadata plus compressed image data

Our current chunk `5` parses cleanly as:

- `u32 raw_size = 66`
- `66` raw metadata bytes
- `u32 decomp_size = 12672`
- `u32 comp_size = 6748`
- RLE payload

The decompressed payload size is:

- `12672`

This is exactly:

- `192 * 66`

The rendered output looks like font/sprite-sheet style glyph data, which strongly supports the prior attempt’s “font sprite sheet” interpretation.

### Verified: chunks 0, 1, and 4 are palette-plus-screen bundles

The prior attempt implied that several early chunks were:

- `768` bytes of VGA palette
- then a compressed image payload descriptor

This was independently verified.

Using:

- palette = first `768` bytes
- `u32 comp_size` at offset `768`
- RLE payload after that

we successfully decoded:

- `chunk 0` -> `320x240` DDD splash / studio logo screen
- `chunk 1` -> `320x240` gameplay HUD / empty playfield screen
- `chunk 4` -> `320x240` ACiD Tetris title/logo screen

This is a major advance because it resolves several of the early “unknown” chunks.

## New Working Map Of Early Chunks

Based on current independent verification, the early archive chunk map is now much stronger:

- `chunk 0`
  `320x240` palette + compressed DDD splash screen
- `chunk 1`
  `320x240` palette + compressed gameplay screen
- `chunk 2`
  very likely remap/fade tables
  Size alignment with prior claim is strong, but still not fully decoded
- `chunk 3`
  likely piece/sprite-related data
  still not fully decoded
- `chunk 4`
  `320x240` palette + compressed ACiD Tetris title/logo screen
- `chunk 5`
  raw metadata + compressed `192x66` font/glyph image
- `chunk 6`
  compressed `200x175` tiled background pattern
- `chunk 7`
  compressed `1024x224` graphics/background data
  not a direct `320x240` image
- `chunk 8`
  compressed `128x36` `GAME OVER` overlay

## Prior Information That Is Useful But Still Needs Verification

These items look promising, but they should remain hypotheses until we verify them directly from the current project.

### Decompiled function/address naming

The annotated decompilation claims useful labels like:

- `main_init` at `0x023B`
- `rle_decompress` at `0x2880`
- `vga_write_planes` at `0x2938`
- `menu_main_loop` at `0x3830`
- `menu_game_loop` at `0x5A94`

These labels are plausible and several line up with behavior we already suspect, but they are still prior-attempt annotations, not authoritative symbols.

### Strings dump

The prior `strings.txt` is valuable because it contains useful text we had not fully cataloged yet, including:

- `atet.dat`
- `setup.dat`
- music menu strings for all six music tracks
- `Music Volume:%d`
- `Sound FX Volume:%d`
- `Keyboard Setup`
- `Mixing Rate:%d`
- `Stereo`
- `%s16 Bit`
- `Play Game`
- `Exit to Dos`
- sound-driver and MikMod-related error strings

This strongly suggests the setup mode and menu system expose more audio configuration than we have captured in screenshots so far.

### Gameplay constants and data-structure hypotheses

The prior attempt also claims:

- scoring multipliers at `DAT_0000098C`
- drop speed formula `level * 0x200 + 0x200`
- `8192`-entry PRNG table
- `2048`-entry sine table
- `4096` max particles
- line-clear animation styles tied to level mod `10`

These are good leads, but they have not all been re-verified yet in the current effort.

## Prior Information That Looks Unreliable Or Conflicted

Some parts of the prior attempt conflict with current evidence and should be treated cautiously.

### Conflicting DAT entry tables

The prior archive contains inconsistent claims about entry numbering and contents.

Examples:

- the README’s DAT table and the annotated decompiled header do not agree with each other
- one annotated section claims `Entry 0` contains WAVs, which conflicts with both the actual archive layout and the prior README

Interpretation:

- the prior work clearly evolved over time
- some comments were not kept in sync
- we should trust only the parts we independently verified

### Prior gameplay reimplementation defaults

The prior `src/game.c` sets default controls like:

- clockwise rotate = `Up`
- counter-clockwise rotate = `Space`

This conflicts with our captured keyboard-setup screen, which shows:

- `Rotate Left: A`
- `Rotate Right: S`

So the prior gameplay source should be treated as an inferred reimplementation, not as authoritative evidence of original default behavior.

## Artifacts Created During This Review

To verify the prior claims without using prior outputs directly, independent current renders were created under:

- `Decompilation.Effort/research/formats/atet-dat/prior-verification/`

These were generated from the current raw archive chunks in this project, not copied from the prior zip.

Key independent verification renders include:

- `chunk0_320x240.png`
- `chunk1_320x240.png`
- `chunk4_320x240.png`
- `chunk6_200x175_pal1.png`
- `chunk7_1024x224_pal1.png`
- `chunk8_128x36_pal4.png`
- `chunk5_192x66_gray.png`

## Recommended Follow-up

The prior attempt gave us enough verified information that the next logical step is:

1. promote the verified chunk formats into our current extraction scripts
2. add proper decoders for chunks `0`, `1`, `4`, `5`, `6`, `7`, and `8`
3. keep chunk `2` and `3` as the main remaining unknown-format focus
4. use the prior decompilation labels only as hypotheses while we verify them in Ghidra

## Practical Conclusion

The prior project was worth reviewing.

Its most valuable contribution to the current effort is not its reimplementation code or extracted assets. It is the format knowledge that we were able to independently confirm:

- the early graphics chunk structures
- the RLE scheme
- the `1024x224` decompressed size of chunk `7`
- the `128x36` `GAME OVER` overlay
- the `200x175` tiled smiley background
- the `192x66` font-like chunk structure

That materially reduces uncertainty for the active decompilation effort.
