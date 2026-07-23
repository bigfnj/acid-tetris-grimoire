# Font And Menu Text Pass

Date: 2026-04-13

## Summary

This pass resolved the frontend font system much more cleanly than before.

The important changes were:

- [decode_verified_graphics.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/decode_verified_graphics.py)
- [render_frontend_menu_text.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/render_frontend_menu_text.py)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)

New outputs:

- [verified font outputs](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/fonts)
- [frontend text renders](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-text)

## Chunk 5 Resolution

Chunk `5` is now strongly resolved as a frontend font resource, not just a vague "font-like image."

Verified structure:

- first dword: metadata size = `66`
- next `66` bytes: width table
- next dword: decompressed glyph data size = `12672`
- next dword: compressed glyph payload size = `6748`
- decompressed glyph data length: `12672 = 66 * 16 * 12`

That is an exact fit for:

- `66` glyphs
- each glyph stored as `16x12`
- sequentially, not as a conventional sprite sheet grid

This also matches the executable-side text renderer:

- `0x178b0` indexes glyphs with a `192`-byte stride
- `192 = 16 * 12`

So chunk `5` is now best modeled as:

- width table: `66 x 1 byte`
- glyph table: `66 x 16 x 12`

## Character Mapping

The static remap table at `0x189b8` lines up cleanly with the chunk-5 glyph count.

Useful resolved ranges:

- `A-Z` -> glyph codes `1..26`
- `a-z` -> glyph codes `27..52`
- `0-9` -> glyph codes `53..62`
- apostrophe `'` -> glyph code `63`
- colon `:` -> glyph code `64`
- backtick `` ` `` -> glyph code `65`
- underscore `_` -> glyph code `66`

The renderer treats unmapped characters as blanks and advances by `6` pixels.

## Executable-Side Text Path

The frontend text path is now much clearer:

- `0x60cc` draws a frontend string using the remap table plus chunk-5 width metadata
- `0x61c8` does not draw text; it clears the rectangle needed for a string band
- `0x6274` clears a rectangle to palette index `0`
- `0x178b0` draws one chunk-5 glyph into a `16`-pixel-tall destination band

Important renderer behavior:

- visible glyphs advance by `width + 2`
- spaces advance by `6`
- centered strings subtract half the measured width from the supplied center X
- `0x178b0` scales/reveals the `12` source rows into `16` destination rows using the caller-supplied reveal amount

That means the fifth stack argument passed into `0x60cc` is not just an arbitrary style token. It is part of the text reveal/vertical-resample behavior.

## New Owned Outputs

The verified graphics pass now exports:

- [font widths JSON](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/fonts/atet-dat.chunk-0005.off-00014789.len-00001AAA.font-widths.json)
- [glyph atlas gray](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/fonts/atet-dat.chunk-0005.off-00014789.len-00001AAA.font-atlas-176x72-gray.png)
- [glyph atlas menu palette](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/fonts/atet-dat.chunk-0005.off-00014789.len-00001AAA.font-atlas-176x72-menu-palette.png)

The research pass now renders reconstructed menu text layers:

- [main menu reveal 48](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-text/main-menu.reveal-48.png)
- [main menu reveal 64](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-text/main-menu.reveal-64.png)
- [options menu reveal 48](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-text/options-menu.reveal-48.png)
- [options menu reveal 64](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-text/options-menu.reveal-64.png)

These are intentionally research-owned renders, not treated as original assets.

## Motion Correlation Result

The new menu text layer explains the frontend captures better than the old hand-wavy interpretation, but not enough to close the problem by itself.

Against the existing title/options motion masks:

- main menu text render overlap remains very weak
- options menu text render overlap is somewhat better than main menu
- but both are still far too small to explain the whole capture-side motion footprint

Current best text-only overlap from the sample renders:

- options menu reveal `48`
- motion overlap: `23`
- motion Jaccard: `0.001941`

That is still weak.

## What This Means

This pass resolves the font subsystem well, but it also tells us something important:

- the frontend capture mismatch is not just "we forgot the menu text layer"

Menu text matters, and it definitely contributed noise to the earlier correlation work, but the current text-only reconstruction still does not match the capture-side spatial change strongly enough to explain the remaining gap.

## Recommended Next Move

The best next move is to keep following the menu-side presentation path, especially:

- the exact behavior of `0x3fd8` during steady-state menu highlighting
- how `0x60cc` reveal values vary in the main menu and options menu loops
- whether the aligned capture set is mixing title and options states in a way that invalidates simple whole-frame comparison

At this point, the font/text system is in good enough shape to stop being a mystery and start being a controlled input to the next frontend-analysis pass.
