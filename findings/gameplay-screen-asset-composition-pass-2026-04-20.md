# Gameplay Screen Asset Composition Pass

Date: 2026-04-20

## Summary

This pass closes one old but still-referenced ambiguity from the early capture review:

- what actually composes the shipped gameplay screen

Current best closure:

- the gameplay screen is **not** one large still-unknown graphics source
- it is a layered composition built from:
  - `chunk 1` gameplay base screen
  - `chunk 3` dynamic piece / digit / ramp resources
  - `chunk 6` alert-face overlays
  - `chunk 8` late `GAME OVER` overlay

The practical result is that the old wording from [capture-review-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/capture-review-2026-04-13.md), which still treated the gameplay art source as one large unknown chunk family, can now be retired.

## Why This Pass Was Needed

The project already had the important ingredients, but they were still spread across several findings:

- [startup-init-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/startup-init-pass-2026-04-13.md)
- [startup-and-chunk3-resolution-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/startup-and-chunk3-resolution-pass-2026-04-14.md)
- [board-alert-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/board-alert-pass-2026-04-13.md)
- [piece-and-board-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/piece-and-board-pass-2026-04-13.md)
- [gameplay-helper-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-helper-pass-2026-04-13.md)
- [topout-and-gameover-presentation-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/topout-and-gameover-presentation-pass-2026-04-14.md)
- [26d000-state4-bootstrap-contributor-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-contributor-pass-2026-04-20.md)
- [26d000-state4-bootstrap-layout-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-layout-pass-2026-04-20.md)

What was missing was one composition-level closure that says:

- which asset chunk owns each major visible gameplay layer
- which chunks do **not** belong to gameplay presentation
- how those layers combine in ordinary play, new-game bootstrap, and game-over

## Findings

### 1. `chunk 1` Is The Gameplay Base Screen

The startup path loads `chunk 1` as a verified `320x240` screen bundle, decodes it into the working screen, and mirrors it into the visible page set before gameplay begins.

Owned extracted output:

- [gameplay-screen-320x240.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/screens/atet-dat.chunk-0001.off-00003391.len-0000AD86.gameplay-screen-320x240.png)

This is the current best source for the stable gameplay background layer:

- playfield frame and border
- left-side panel frame and static labels
- right-side statistics panel frame and static tetromino icons
- bottom `LINES` band
- general blue marbled / jagged-edged presentation family seen in captures

The strongest practical reason to treat those as chunk-`1` base art is:

- later recovered gameplay bootstrap helpers redraw digits, preview pieces, live pieces, and alert state
- they do **not** redraw the fixed panel frames or their static decorative art

So the fixed gameplay layout is best treated as preauthored screen art from `chunk 1`, not as something reconstructed procedurally from later helpers.

### 2. `chunk 3` Supplies The Dynamic Gameplay Piece And Number Resources

The already-closed chunk-`3` layout is now specific enough to place directly into the gameplay composition model.

Owned extracted outputs:

- [piece-atlas-56x8-game-palette.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/auxiliary/atet-dat.chunk-0003.off-000125CB.len-00000641.piece-atlas-56x8-game-palette.png)
- [digit-atlas-25x10-game-palette.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/auxiliary/atet-dat.chunk-0003.off-000125CB.len-00000641.digit-atlas-25x10-game-palette.png)
- [lookup-table-0x400.ramps.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/auxiliary/atet-dat.chunk-0003.off-000125CB.len-00000641.lookup-table-0x400.ramps.json)

Resolved roles:

- first `448` decompressed bytes:
  - seven `8x8` gameplay piece tiles
  - used by active-piece draw and preview-piece draw
- trailing `250` decompressed bytes:
  - ten `5x5` decimal glyphs
  - used by HUD and piece-stat counters through `0x2c58`
- raw leading `0x400` bytes:
  - four-stage transient color ramps
  - used by gameplay debris / tracked transient color staging

That means `chunk 3` contributes the dynamic gameplay-side art that sits on top of the chunk-`1` base:

- live well pieces
- next-piece preview piece
- score / high-score / level / lines digits
- right-panel per-piece count digits
- transient particle / debris colors

### 3. `chunk 6` Is The Lower-Left Alert Tile Bank, Not A General Gameplay Background Layer

Owned extracted outputs:

- [pattern-200x175-game-palette.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/patterns/atet-dat.chunk-0006.off-00016233.len-00001CA7.pattern-200x175-game-palette.png)
- [tile02-50x50-game-palette.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/patterns/atet-dat.chunk-0006.off-00016233.len-00001CA7.tile02-50x50-game-palette.png)
- [tile06-50x50-game-palette.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/patterns/atet-dat.chunk-0006.off-00016233.len-00001CA7.tile06-50x50-game-palette.png)

Executable-side closure already shows:

- `0x1793d` / `0x17983` consume `0x9c4`-byte frames from `0x2c69b`
- `0x9c4` is exactly `50x50`
- `0x206c` stages and restores those icons over the fixed alert region at:
  - `x = 20`
  - `y = 140`

So chunk `6` is part of gameplay presentation, but only as the alert subsystem:

- line-clear and reward faces
- stack warning faces
- sleepy / wake-up faces
- top-out face

The `200x175` preservation render is best treated as a convenient sheet view of the alert bank, not as the live gameplay backdrop.

### 4. `chunk 8` Is The Late `GAME OVER` Overlay Only

Owned extracted outputs:

- [game-over-128x36-game-palette.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/overlays/atet-dat.chunk-0008.off-0002B95C.len-000005C5.game-over-128x36-game-palette.png)

Executable-side closure already shows:

- `chunk 8` decodes into the `128x36` overlay buffer
- the overlay is not part of the steady gameplay base image
- it is drawn only after the staged top-out dissolve completes
- it is later explicitly restored away before the state-`9` handoff

So chunk `8` is a late temporary gameplay overlay, not a persistent gameplay screen layer.

### 5. `chunk 4` And `chunk 7` Do Not Belong To Gameplay Screen Composition

This is the negative half of the closure, and it matters because older notes sometimes blurred gameplay art with frontend art.

Already-owned closure:

- `chunk 4`
  - frontend title/logo base screen
- `chunk 7`
  - structured frontend floating-object bank

So the gameplay screen should **not** be modeled as:

- chunk-`7` panel selection
- title/logo-family reuse
- or a mixed gameplay/frontend decorative bank

The gameplay screen is its own composition family built on chunk `1` plus the gameplay-local overlays above.

### 6. The Actual Visible Gameplay Image Is A Layer Stack, Not One Asset

Current best layer model for ordinary live gameplay:

1. chunk `1` gameplay base screen
2. chunk `3` live piece / preview / HUD digits / piece-stat digits
3. chunk `6` alert tile when active
4. chunk `3` transient ramps driving debris and tracked transient colors

Current best layer model for `New Game` bootstrap:

1. restored gameplay snapshot based on the chunk-`1` family
2. `0x05e0` clears the well interior and preview area
3. staged chunk-`3` digits and preview/bootstrap state
4. clean alert-region restore
5. first shared gameplay step adds the live piece

Current best layer model for top-out:

1. steady gameplay composition
2. top-out alert through chunk `6`
3. staged dissolve using gameplay-side particle/transient paths
4. late chunk-`8` `GAME OVER` overlay
5. overlay restore
6. handoff into frontend/high-score family

## Closure

The old broad wording:

- "large unknown graphics chunk likely tied to gameplay art"

is now no longer the best project-level reading.

The current best reading is:

- fixed gameplay base art is chunk `1`
- dynamic piece / digit / debris support is chunk `3`
- lower-left alert art is chunk `6`
- late `GAME OVER` overlay is chunk `8`
- chunks `4` and `7` belong to frontend presentation, not gameplay-screen composition

## Port Implication

For the faithful port, the gameplay renderer should preserve this asset split:

- base gameplay screen as a static authored layer
- dynamic piece and counter art as separate overlays
- alert faces as a saved-under fixed-region overlay system
- `GAME OVER` as a late temporary overlay, not a static replacement screen

That is a much better target than treating gameplay as one monolithic background bitmap or one generic runtime redraw.

## Artifact

This pass adds:

- [gameplay-screen-asset-composition.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-screen-asset-composition.json)

## What I Now Treat As Resolved

- the gameplay screen is no longer an unresolved large-art-source bucket
- chunk ownership for the major gameplay visual layers is now specific enough to preserve directly
- the remaining open questions are finer-grained than "where does the gameplay screen art come from?"

## Next Ordered Step

- pause the gameplay-screen asset-composition branch unless one of these becomes newly useful:
  - a need to attribute one remaining fixed gameplay detail inside chunk `1` more precisely
  - a port-side renderer milestone that wants a direct gameplay layer contract
  - a later capture or static pass that needs one sublayer compared against this now-closed composition map
