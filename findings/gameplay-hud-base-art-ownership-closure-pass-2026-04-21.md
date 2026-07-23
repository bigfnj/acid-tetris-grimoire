# Gameplay HUD Base-Art Ownership Closure Pass

Date: 2026-04-21

## Summary

This pass closes one more fixed-art question inside the already-sharpened gameplay composition model:

- what owns the left HUD labels, preview housing, and bottom `LINES` band

Current best closure:

- the fixed gameplay HUD text and framing belong to the authored chunk-`1` gameplay base screen
- runtime gameplay helpers only redraw the numeric digits and the preview piece inside those fixed regions

So the gameplay HUD is now best modeled as:

- chunk-`1` fixed labels and panel framing
- chunk-`3` digits and preview piece only

not:

- a runtime-reconstructed text or frame layer assembled from the gameplay helpers

## Why This Pass Was Needed

[gameplay-screen-asset-composition-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-screen-asset-composition-pass-2026-04-20.md) already closed the large split:

- chunk `1` gameplay base
- chunk `3` dynamic gameplay resources
- chunk `6` alert overlays
- chunk `8` late `GAME OVER` overlay

And the later gameplay HUD passes already closed the dynamic ownership side:

- [gameplay-hud-counter-label-closure-pass-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-hud-counter-label-closure-pass-2026-04-21.md)
- [high-score-live-redraw-closure-pass-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/high-score-live-redraw-closure-pass-2026-04-21.md)

But one implied sub-detail was still worth making explicit:

- whether the left HUD labels and preview frame were fixed chunk-`1` art
- or whether the gameplay helpers were expected to reconstruct more of that panel at runtime

The owned startup, bootstrap, and helper map now makes that answer clean.

## Findings

### 1. Chunk `1` Already Owns The Fixed Left Gameplay HUD

The gameplay composition pass already grounded chunk `1` as the authored gameplay base screen containing:

- the left-side panel frame and static labels
- the preview housing
- the bottom `LINES` band
- the broader themed border work around the gameplay layout

That means the default ownership for:

- `NEXT`
- `HIGH-SCORE`
- `SCORE`
- `LEVEL`
- the preview box frame
- the bottom `LINES` label band

is already chunk `1`, unless a later runtime helper family proves otherwise.

### 2. Chunk `3` Only Gives Us Digits, Piece Tiles, And Ramps

The owned chunk-`3` closure is already exact:

- seven `8x8` piece tiles
- ten `5x5` decimal digits
- shared four-stage transient ramps

No extra chunk-`3` region remains available for:

- a gameplay HUD label font
- a preview frame bank
- or a separate bottom-band text strip

So the owned dynamic gameplay-art family can only explain:

- numeric counters
- live or preview tetromino tiles
- transient color behavior

not:

- the fixed HUD text and framing itself

### 3. Owned Runtime HUD Writes Are Digit Writes Only

The owned `0x2c58` redraw set is now specific:

- bootstrap `0x05e0` draws:
  - `HIGH-SCORE`
  - `SCORE`
  - `LEVEL`
  - `LINES`
  - piece-stat counts
- live `0x09c8` redraws:
  - `LEVEL` conditionally
  - `LINES`
  - `SCORE`
- `0x1348` redraws:
  - one piece-stat count

And `0x2c58` itself is already closed as:

- a decimal digit renderer using the `5x5` chunk-`3` glyphs

So the owned gameplay HUD update family redraws:

- digits only

not:

- `HIGH-SCORE` text
- `SCORE` text
- `LEVEL` text
- `LINES` text
- or the surrounding fixed panel frames

### 4. The Preview Path Rewrites Only The Interior Piece State

The owned preview-side helpers are also now specific:

- `0x05e0` clears the next-piece preview area through `0x6274`
- `0x1348` immediately erases the old preview piece and draws the new preview piece
- `0x10ec` uses the seven `8x8` chunk-`3` piece tiles

That means the runtime preview path owns:

- the blanked interior region
- the preview tetromino itself

but not:

- the `NEXT` label
- the preview frame or housing

Those fixed parts remain better explained as authored chunk-`1` base art around the preview interior.

### 5. The Gameplay HUD Is Therefore Another Fixed-Plus-Dynamic Composite

The best current reading is:

- chunk `1` provides the fixed gameplay HUD text and framing
- chunk `3` provides:
  - HUD digits
  - the preview piece

This matches the broader pattern now established across the gameplay screen:

- fixed authored art in chunk `1`
- dynamic counts and piece content in chunk `3`
- fixed alert overlays in chunk `6`
- temporary `GAME OVER` overlay in chunk `8`

## Closure

The left gameplay HUD is now best modeled as:

- fixed chunk-`1` labels and framing
- dynamic chunk-`3` digits and preview piece only

That retires the weaker implied wording that left the left panel and preview housing inside generic "static panel art" without explicitly separating them from the owned runtime redraw family.

## Port Implication

For a faithful port:

1. keep the left HUD labels, preview frame, and bottom `LINES` band in the authored gameplay base layer
2. redraw only the digits and preview tetromino on top of those fixed regions
3. do not treat the gameplay HUD text or preview housing as a runtime text-render or frame-rebuild surface

## Artifact

This pass adds:

- [gameplay-hud-base-art-ownership-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-hud-base-art-ownership-closure.json)

## What I Now Treat As Resolved

- the left gameplay HUD text and preview housing no longer need to remain implied chunk-`1` details
- chunk `3` ownership there is now specific:
  - digits and preview piece yes
  - fixed labels and frames no
- the gameplay composition map is now sharper across both the left and right side panels

## Next Ordered Step

- pause this HUD base-art ownership branch unless a later renderer-contract pass wants more fixed gameplay subregions named explicitly
