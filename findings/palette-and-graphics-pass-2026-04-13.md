# Palette And Graphics Pass

Date: 2026-04-13

## Summary

This pass added two new analysis steps:

- palette candidate export from palette-like `ATET.DAT` chunks
- first-pass visualization of chunk `7` as possible 320x240 indexed image data

The palette export produced useful results.

The chunk `7` visualization produced an important negative result:

- chunk `7` does not appear to decode into a recognizable image as plain linear indexed pixels
- chunk `7` also does not appear to decode into a recognizable image using simple Mode X planar assumptions under the offsets tested

That is still valuable because it narrows the search space.

## Scripts Added

- `Decompilation.Effort/scripts/extract_palettes.py`
- `Decompilation.Effort/scripts/visualize_chunk7.py`

## Outputs Generated

### Palette Outputs

Manifest:

- `Decompilation.Effort/extracted/converted/palettes/manifest.json`

Exported palette candidates:

- `chunk0000-pal00`
- `chunk0000-pal01`
- `chunk0000-pal02`
- `chunk0001-pal00`
- `chunk0002-pal00`
- `chunk0004-pal00`

Each palette candidate was exported as:

- JSON
- GIMP palette (`.gpl`)
- PNG swatch image

### Graphics Visualization Outputs

Manifest:

- `Decompilation.Effort/extracted/converted/graphics/chunk-0007-renders/manifest.json`

Generated render count:

- `126`

These renders tested:

- linear 320x240 indexed layout
- Mode X plane-major 320x240 reinterpretation
- Mode X row-planar 320x240 reinterpretation

Across multiple candidate offsets near the start of chunk `7` and near the likely `320*240 = 76800` byte payload boundary near the end of the chunk.

## Palette Findings

### Chunk 0

The chunk `0` palette exports look sparse and strongly menu-like.

Observed characteristics:

- lots of black / empty entries
- blue ramp
- warm orange / tan ramp
- magenta / purple ramp

This looks compatible with the title/menu/credits screens that use:

- black background
- deep blue logo outline
- purple beveled text
- warm red/brown title accents

Interpretation:

- chunk `0` is a strong candidate for menu/credits palette data or a palette bank used by those screens

### Chunk 1

The chunk `1` palette export looks much richer and more gameplay-like.

Observed characteristics:

- broad blue range
- darker cyan / teal tones
- purple, magenta, yellow, green, and red accents
- enough variation to plausibly support backdrop art plus colored tetromino sprites

Interpretation:

- chunk `1` is the strongest current candidate for a gameplay palette bank

This lines up well with the gameplay captures, which show:

- blue marbled background
- brown/tan borders
- brightly colored tetromino sprites
- magenta HUD text

### Chunk 2

The chunk `2` export is effectively a grayscale ramp.

Interpretation:

- this may be a utility palette
- it may support fade effects, transitions, or lookup work
- it is not the most likely primary gameplay palette

### Chunk 4

The chunk `4` export shows:

- bright reds
- strong purple/lavender ramp
- dark blue tones
- green range

Interpretation:

- this may be a second UI/screen-specific palette bank
- it could support alternate menu/credits/high-score pages or overlay effects

## Chunk 7 Visualization Findings

### Strong Signal

Chunk `7` remains the strongest candidate for major graphics-related content because:

- it is large
- its size is very close to `320 * 240 = 76800` plus a small leading region
- it still feels structurally different from the obvious table-like chunks
- its first dword is exactly `length - 4`, which strongly suggests an explicit block wrapper or size-prefixed format

### Negative Result

However, the first-pass renders show:

- linear interpretation looks like structured noise
- Mode X plane-major interpretation also looks like structured noise
- Mode X row-planar interpretation also looks like structured noise

This suggests chunk `7` is probably one of:

- compressed graphics data
- tile/sprite-packed data rather than a single full-screen image
- graphics plus lookup/offset structures mixed together
- data requiring additional decode tables from other early chunks
- a size-prefixed encoded block with repeated control/record values in the leading region

### Important Practical Conclusion

Chunk `7` should no longer be treated as “probably just a raw framebuffer dump with a small header.”

That hypothesis has now been tested in a reasonable first pass and did not produce recognizable art.

## Decompilation Impact

These results shift the likely roles of the early chunks into a more refined picture:

- chunk `0`: menu/credits palette bank candidate
- chunk `1`: gameplay palette bank candidate
- chunk `2`: grayscale/fade/utility palette candidate
- chunk `3`: offset or coordinate table candidate
- chunk `4`: alternate palette/UI palette candidate
- chunk `7`: encoded or packed graphics-related data, but likely not directly display-ready

## Recommended Next Steps

1. Add a chunk-7 structure analysis pass that looks for internal tables, sub-block boundaries, and compression-like patterns.
2. Correlate chunk `7` use in Ghidra with the palette-bearing chunks `0`, `1`, and `4`.
3. Prefer reverse engineering the chunk loader / decode path in `ATET.EXE` before adding more brute-force image reinterpretations.
4. If runtime tracing becomes convenient, capture memory after graphics are decoded rather than trying to guess all static encodings from file data alone.

## Bottom Line

This pass was successful.

It did not fully decode gameplay graphics yet, but it did produce two important advances:

- we now have concrete exported palette candidates that visually correlate with captured screens
- we have ruled out the simplest “raw 320x240 image” interpretations for chunk `7`

That is meaningful progress and gives the next reverse-engineering step a much clearer target.
