# Piece And Board Pass

Date: 2026-04-13

## Summary

This pass resolved the core helpers for:

- active-piece collision testing
- active-piece draw / erase behavior
- line-clear row handling
- stack-height warning logic

Together with the previous gameplay-helper pass, this gives us a much more actionable gameplay model for the future source port.

## `0x12ac` Tests Whether A Piece Can Occupy A Position

`0x12ac` is the piece-placement collision test.

Its inputs are effectively:

- `EAX` -> board X
- `EDX` -> board Y
- `EBX` -> rotation
- `ECX` -> piece ID

What it does:

- masks rotation to `0..3`
- walks the four block offsets for the selected piece and rotation
- rejects positions where any block would be:
  - left of column `0`
  - right of column `9`
  - below row `19`
- treats negative Y as allowed spawn space above the visible field
- checks occupied cells only when the candidate block is inside the visible board
- returns `1` on collision / invalid placement and `0` on success

This is the clearest executable confirmation so far that:

- the logical board is `10x20`
- the piece system uses four orientations
- spawn logic allows part of the piece to exist above the visible playfield

## `0x10ec` Draws A Piece Using Chunk-3 Tile Graphics

`0x10ec` is the best current candidate for the piece draw helper.

Evidence:

- it walks four block offsets from the piece/rotation tables
- it computes per-block screen positions in `8x8` steps
- it blits `8x8` tiles sourced from the chunk-3 auxiliary graphics buffer at `0x2c60b`
- it is called after movement resolution and after next-piece selection

Current best reading:

- draw a tetromino into the working screen using opaque or foreground tile copies

## `0x11e0` Restores The Background Under A Piece Footprint

`0x11e0` is paired closely with `0x10ec`.

Evidence:

- it walks the same piece/rotation offset tables
- it is called before movement and before the next-piece preview is replaced
- it uses a different lower-level copy primitive than `0x10ec`
- it performs explicit clipping checks before drawing

Current best reading:

- restore or erase the previous piece footprint using background-aware tile copies

That explains the call pattern:

- `0x11e0` before movement to remove the old piece image
- `0x10ec` after movement to draw the new piece image

## Piece Geometry Tables

The piece helpers use geometry tables rooted at:

- `0x184e3`
- `0x184e7`

Those tables provide X and Y offsets for the four blocks in a tetromino across four rotations.

This is the table family used by:

- piece drawing
- piece erasing
- collision testing

## `0x1414` Repaints Cleared Rows Using Level-Themed Logic

`0x1414` is called once per cleared row during the line-clear path.

What it does:

- dispatches through a ten-entry table based on `current_level % 10`
- calls one of ten level-specific helpers
- marks one board-row-sized region dirty afterward

Current best reading:

- run the current level/theme's row-clear effect before the multi-frame collapse step

Update:

- the follow-up pass in [line-clear-theme-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-theme-pass-2026-04-13.md) resolved the ten leaf helpers as visual debris or particle treatments and identified `0x1d04` plus `0x1e34` as the actual board-collapse pipeline

## `0x1de8` Finds The Topmost Occupied Board Row

`0x1de8` scans the logical board buffer `0x2c23b` from the top.

It returns the first row index that contains any occupied cell, or `20` if the board is empty.

This helper is useful because it feeds directly into the warning logic below.

## `0x21c4` Updates Rising-Stack Warning Effects

`0x21c4` is a board-pressure warning helper.

Evidence:

- it calls `0x1de8` every frame
- it classifies the returned row into danger bands
- it uses cooldown timers at:
  - `0x2c753`
  - `0x2c757`
  - `0x2c76b`
- it triggers sound/effect IDs `3`, `4`, or `5` depending on the danger band

Current best reading:

- update stack-height warning audio/visual cues as the pile gets closer to the top of the board

This is a nice preservation detail for the future port:

- the original game is not just checking for game-over
- it also has escalating pressure feedback as the stack rises

## Decompilation Impact

This pass improves the source-port plan in practical ways:

- collision rules are explicit enough to recreate directly
- piece draw and piece erase are now understood as separate helpers
- the line-clear path is tied to per-level row repaint logic
- rising-stack warning behavior is identified as a real gameplay feedback system, not incidental noise

## Recommended Next Move

The strongest next step after this pass is the one now captured in [line-clear-theme-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-theme-pass-2026-04-13.md):

- connect the resolved line-clear effect bank to captured gameplay footage
- map the shared alert helper at `0x2008`
- tie the level/theme IDs back to the chunk-7 background panels and any reuse rules
