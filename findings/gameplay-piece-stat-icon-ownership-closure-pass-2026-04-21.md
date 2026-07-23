# Gameplay Piece-Stat Icon Ownership Closure Pass

Date: 2026-04-21

## Summary

This pass closes one smaller fixed-art question inside the already-closed gameplay composition model:

- what owns the right-side piece-stat panel silhouettes

Current best closure:

- the fixed tetromino silhouettes on the right-side statistics panel belong to the authored chunk-`1` gameplay base screen
- runtime gameplay helpers only redraw the numeric per-piece counts on top of that fixed art

So the right panel is now best modeled as:

- static chunk-`1` panel frame plus silhouettes
- dynamic chunk-`3` decimal counts only

not:

- a panel whose tetromino silhouettes are re-blitted from chunk `3` during ordinary gameplay

## Why This Pass Was Needed

[gameplay-screen-asset-composition-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-screen-asset-composition-pass-2026-04-20.md) already closed the large asset split:

- chunk `1` gameplay base
- chunk `3` dynamic pieces, digits, and ramps
- chunk `6` alert faces
- chunk `8` late `GAME OVER` overlay

But one sub-detail inside that model was still only implied:

- whether the seven right-panel tetromino silhouettes were fixed authored base art
- or whether they were another dynamic chunk-`3` gameplay draw family

The owned helper map now lets that close cleanly.

## Findings

### 1. Chunk `1` Already Carries The Whole Fixed Right Panel

The gameplay composition pass already grounded chunk `1` as the authored gameplay base screen containing:

- playfield frame and border
- left-side panel frame and static labels
- bottom `LINES` band
- right-side statistics panel frame and static panel art

That right-side panel remains visibly stable across:

- new-game bootstrap
- ordinary gameplay
- late game-over

So the default ownership for fixed art there is already chunk `1`, unless a later runtime redraw family proves otherwise.

### 2. Chunk `3` Resolves Only To Piece Tiles, Decimal Digits, And Ramps

The owned chunk-`3` closure is already exact:

- first `448` decompressed bytes:
  - seven `8x8` gameplay piece tiles
- trailing `250` decompressed bytes:
  - ten `5x5` decimal digits
- raw `0x400` bytes:
  - four-stage transient color ramps

No extra chunk-`3` region remains available for:

- a separate fixed right-panel silhouette bank

So if the right panel is changing in normal gameplay, the owned chunk-`3` family only gives us two practical draw types:

- `8x8` piece tiles
- `5x5` decimal digits

### 3. The Owned Right-Panel Runtime Writes Are Counter Writes, Not Icon Writes

The owned bootstrap and gameplay helper passes now agree on the dynamic right-panel work:

- `0x05e0` stages seven zeroed per-piece counters through `0x2c58`
- those rows sit at:
  - `x = 248`
  - `y = 47, 58, 69, 80, 91, 102, 113`
- `0x1348` then increments and redraws one promoted-piece counter through `0x2c58`

And `0x2c58` itself is already closed as:

- a decimal counter renderer using the `5x5` digit glyphs from chunk `3`

That means the owned right-panel dynamic updates are specifically:

- digits only

not:

- silhouette redraws
- icon replacement
- or panel reconstruction from piece tiles

### 4. The Owned Piece-Tile Drawers Belong Elsewhere

The owned `0x10ec` / `0x11e0` helpers use the seven `8x8` chunk-`3` piece tiles for:

- live piece draw or erase in the well
- next-piece preview draw or erase

Those helpers are already grounded by the preview and playfield ownership passes.

The owned right-panel statistic updates, by contrast, route through:

- `0x2c58`

So the gameplay model now splits cleanly:

- piece tiles:
  - well and preview
- digits:
  - HUD and right-panel per-piece counts

There is no owned runtime helper family left that needs to carry the fixed right-panel silhouettes.

### 5. The Piece-Stat Panel Is Therefore A Fixed-Plus-Dynamic Composite

The best current reading is:

- chunk `1` provides the fixed panel frame and the seven tetromino silhouettes
- chunk `3` decimal glyphs provide the changing counts at the right-edge rows

That explains why the panel identity stays visually stable while:

- the counts change from `0`
- one promoted piece row becomes `1`
- later gameplay increments continue

without requiring any dynamic re-blit of the static silhouettes themselves.

## Closure

The right-side piece-stat panel is now best modeled as:

- fixed chunk-`1` silhouettes plus frame
- dynamic chunk-`3` digits only

This retires the weaker implied wording that left the silhouettes merely bundled into generic "static panel art" without saying whether runtime gameplay helpers ever owned them.

## Port Implication

For a faithful port:

1. keep the right-panel tetromino silhouettes in the authored gameplay base layer
2. redraw only the numeric counts on top of that base during bootstrap and live gameplay
3. do not treat the whole right panel as a runtime piece-tile composition surface

## Artifact

This pass adds:

- [gameplay-piece-stat-icon-ownership-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-piece-stat-icon-ownership-closure.json)

## What I Now Treat As Resolved

- the right-panel silhouettes no longer need to be carried as an implied chunk-`1` detail
- chunk `3` ownership on that panel is now specific:
  - digits yes
  - fixed silhouettes no
- the gameplay composition map is now sharper at the sub-panel level

## Next Ordered Step

- pause this piece-stat icon ownership branch unless a later renderer-contract pass wants more fixed gameplay subregions named explicitly
