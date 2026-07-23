# Gameplay Well Frame Ownership Closure Pass

Date: 2026-04-21

## Summary

This pass closes one more fixed-art question inside the gameplay composition model:

- what owns the central gameplay well frame, versus what the runtime clears and redraws inside it

Current best closure:

- the decorative well frame and border belong to the authored chunk-`1` gameplay base screen
- runtime gameplay helpers own only the `80x160` well interior
- empty cells inside that interior are restored to palette-zero background, while occupied cells are drawn from the chunk-`3` piece tiles

So the central playfield is now best modeled as:

- chunk-`1` fixed well frame
- runtime-cleared well interior populated by chunk-`3` tetromino tiles

not:

- one untouched preauthored well image carried straight through gameplay
- or a runtime-rebuilt border assembled by gameplay helpers

## Why This Pass Was Needed

[gameplay-screen-asset-composition-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-screen-asset-composition-pass-2026-04-20.md) already closed the large gameplay layer split:

- chunk `1` gameplay base
- chunk `3` dynamic gameplay resources
- chunk `6` alert overlays
- chunk `8` late `GAME OVER` overlay

And the later composition-tightening passes already closed the side panels:

- [gameplay-piece-stat-icon-ownership-closure-pass-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-piece-stat-icon-ownership-closure-pass-2026-04-21.md)
- [gameplay-hud-base-art-ownership-closure-pass-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-hud-base-art-ownership-closure-pass-2026-04-21.md)

But one central detail was still only implied:

- whether the well border itself stayed fixed chunk-`1` art
- or whether gameplay helpers owned more of the well framing than just the live blocks

The owned startup clears, bootstrap layout work, and piece helper closures now make that answer direct enough to record explicitly.

## Findings

### 1. Chunk `1` Already Owns The Fixed Well Frame

The gameplay composition pass already grounded chunk `1` as the authored gameplay base screen containing:

- the central playfield frame and decorative border
- the left HUD framing and labels
- the right statistics panel framing and silhouettes
- the bottom `LINES` band

So the default ownership for the visible stone-like or marbled frame around the well is already chunk `1`, unless the runtime helper family proves otherwise.

The owned helper family does not prove otherwise.

### 2. `0x05e0` Clears Only The Well Interior, Not The Surrounding Frame

The raw `0x05e0` startup slice now gives the exact first clear call:

```text
MOV ECX,0xa0
MOV EBX,0x50
MOV EDX,0x15
MOV EAX,0x6d
CALL 0x00006274
```

With the already-closed `0x6274` contract:

- `EAX` = `x`
- `EDX` = `y`
- `EBX` = `width`
- `ECX` = `height`

that is:

- `x = 109`
- `y = 21`
- `width = 80`
- `height = 160`

Those coordinates match the full playable `10x20` field at `8x8` pixels per cell.

So the startup helper explicitly wipes the well interior rectangle, not the authored frame around it.

### 3. Ordinary Gameplay Piece Helpers Also Stay Inside That Interior

The owned piece helpers are already specific:

- `0x10ec`
  - blits `8x8` chunk-`3` piece tiles into the working screen through `0x176df`
- `0x11e0`
  - zero-fills matching `8x8` footprints through `0x17700`

And the lower-level helpers are already closed as:

- `0x176df`
  - copy two dwords per row for `8` rows
- `0x17700`
  - zero-fill two dwords per row for `8` rows

So the ordinary live gameplay write family does:

- block-sized piece blits
- block-sized piece erases back to zero

inside the well

not:

- border reconstruction
- decorative frame redraw
- or any broader gameplay-side repaint of the authored well housing

### 4. The Empty Well Surface Is A Runtime-Zeroed Interior, Not A Preserved Chunk-`1` Interior

The state-`4` bootstrap work already showed that `0x05e0`:

- clears the well interior
- uploads that partially reset gameplay image to all three pages before later staged redraws finish

So the visible empty well during a new run is not best explained as untouched chunk-`1` interior pixels.

It is better explained as:

- the authored chunk-`1` frame still surrounding the well
- a runtime-cleared palette-zero interior inside that frame

Later ordinary gameplay keeps the same contract:

- `0x10ec` adds live piece tiles
- `0x11e0` restores vacated cells to zero inside the same playfield interior

### 5. The Gameplay Well Is Therefore Another Fixed-Plus-Dynamic Composite

The best current reading is:

- chunk `1` supplies the fixed well border and decorative surround
- runtime gameplay helpers own the `80x160` interior surface
- chunk `3` contributes live tetromino tiles inside that interior only

This matches the broader composition pattern already established elsewhere on the gameplay screen:

- fixed authored art in chunk `1`
- dynamic digits and tetromino tiles in chunk `3`
- fixed-region alert overlays in chunk `6`
- late `GAME OVER` overlay in chunk `8`

## Closure

The gameplay well is now best modeled as:

- fixed chunk-`1` frame and border
- runtime-owned `80x160` interior

That retires the weaker wording that left the playfield as one generic "base screen" region without explicitly separating the fixed frame from the runtime-cleared interior it surrounds.

## Port Implication

For a faithful port:

1. keep the well frame and decorative surround in the authored gameplay base layer
2. treat the `80x160` well interior as a runtime surface cleared to palette zero
3. redraw only live tetromino tiles inside that interior instead of repainting the surrounding frame on every movement step

## Artifact

This pass adds:

- [gameplay-well-frame-ownership-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-well-frame-ownership-closure.json)

## What I Now Treat As Resolved

- the gameplay well no longer needs to remain one broad chunk-`1` implication
- chunk `1` ownership there is now specific:
  - fixed frame yes
  - live empty or occupied interior no
- runtime gameplay ownership there is now specific:
  - clear and redraw the `80x160` interior yes
  - redraw the surrounding frame no

## Next Ordered Step

- pause this well-frame ownership branch unless a later renderer-contract pass wants more fixed gameplay subregions named explicitly
