# Gameplay Helper Pass

Date: 2026-04-13

## Summary

This pass followed the gameplay helpers behind the new-game path and the live gameplay loop. The most important payoff is that the second high-score column is no longer ambiguous.

It is the total number of `LINES` cleared in the run.

That conclusion comes directly from the gameplay loop:

- `0x05e0` resets `0x2c72f` to `0` at the start of a new game
- the gameplay loop at `0x09c8` adds the number of cleared lines to `0x2c72f`
- the high-score insertion path at `0x50b0` copies `0x2c72f` into the saved record
- the gameplay HUD already labels that live value as `LINES`

## `0x09c8` Is The Main Gameplay Update Loop

`0x09c8` is the live gameplay update loop or frame-step handler.

High-confidence behaviors visible in this function:

- reads the active key state table through the configured bindings from `SETUP.DAT`
- applies left, right, rotate-left, rotate-right, and down behavior through per-action repeat state
- uses short repeat-delay counters for each input direction/action
- moves and redraws the active tetromino
- checks board collisions through `0x12ac`
- detects cleared lines by scanning the logical `10x20` board buffer rooted at `0x2c23b`
- triggers line-clear side effects, score updates, line-count updates, and level progression
- advances to the next piece through `0x1348`
- detects failure/collision after spawn and triggers the game-over path

This function also copies three small constant tables onto the stack at startup:

- `0x0980` -> `2, 0, 9, 7`
- `0x0990` -> `9, 7, 7, 10`
- `0x09a0` -> `100, 300, 600, 1200`

The third table is now clearly the line-clear score table.

Update:

- the repeat-delay model is now resolved in [gameplay-input-timing-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-input-timing-pass-2026-04-14.md)
- left, right, rotate-left, and rotate-right each use their own `8`-frame timer
- Down uses a separate soft-drop override plus the hold-lockout field at `0x2c783`

## Scoring And Level Progression

The scoring logic in `0x09c8` is now much clearer.

When one or more lines are cleared:

- `0x184df` holds the number of cleared lines in the current event
- `0x2c72f += cleared_lines`
- score increment is:
  `(current_level + 1) * score_table[cleared_lines]`
- `score_table` is:
  - `1 line` -> `100`
  - `2 lines` -> `300`
  - `3 lines` -> `600`
  - `4 lines` -> `1200`

Level progression is line-based:

- the loop computes `total_lines % 10`
- if `total_lines % 10 + cleared_lines >= 10`, the level increases
- when the level increases, the timing value at `0x2c717` increases by `0x200`

This is the direct executable proof that:

- the high-score table’s second numeric column is `LINES`
- the gameplay `LINES` HUD and the saved high-score secondary field are the same run statistic

## `0x1330` Clears The Logical Board

`0x1330` is a simple but important helper.

It zeroes `0xc8` bytes at `0x2c23b`.

That is exactly `200` cells, which matches a `10x20` Tetris playfield.

Practical reading:

- `0x2c23b` is the logical board occupancy/state buffer
- `0x1330` is the clear-board helper used when a new run starts

## `0x1348` Promotes Next Piece To Current Piece

`0x1348` now reads like the piece-spawn / next-piece bootstrap helper.

It performs this sequence:

1. move the previously selected next-piece ID from `0x2c6ef` into the live current-piece slot at `0x2c71f`
2. render that piece into the preview/gameplay-side artwork path
3. generate a new next-piece ID modulo `7`
4. store that new next-piece ID back into `0x2c6ef`
5. render the newly chosen next piece
6. increment the per-piece statistics counter at `0x2c6f3[piece_id]`
7. redraw that piece-stat counter using `0x2c58`
8. reset active piece state:
   - board X at `0x2c737` -> `4`
   - board Y at `0x2c71b` -> `0`
   - rotation index at `0x2c733` -> `0`
   - gravity accumulator at `0x2c6d3` -> `0`

This fits the captured HUD exactly:

- live piece statistics on the right side
- next-piece preview on the left side

## `0x2c58` Draws Decimal Counters

`0x2c58` is the numeric HUD renderer.

Its behavior:

- uses the powers-of-ten table at `0x2c30`
- extracts decimal digits from an integer value
- draws digit glyphs from the decompressed chunk-3 auxiliary graphics at `0x2c60b`
- marks the affected screen region dirty through `0x2d30`

Practical role:

- draw zero-padded decimal counters for HUD elements like:
  - score
  - high score
  - level
  - lines
  - per-piece counts

This is why it appears both in new-game initialization and during line-clear / stat updates.

## `0x6274` Clears Rectangular Gameplay Areas

`0x6274` is a rectangular clear helper for the linear `320x240` working buffer.

Arguments are effectively:

- `EAX` -> X
- `EDX` -> Y
- `EBX` -> width
- `ECX` -> height

Behavior:

- computes `y * 320 + x`
- clears `width` bytes per row for `height` rows
- marks the affected region dirty through `0x2d30`

Known startup uses in `0x05e0`:

- clear the main playfield interior
- clear the next-piece preview area

## Decompilation Impact

This pass closes several practical source-port questions:

- the saved high-score secondary field is `total lines`
- the classic line-clear scoring table is confirmed from the executable
- level progression is confirmed to be line-based, with one level per ten lines
- the logical board buffer is identified as a `10x20` grid
- the next-piece and per-piece-stat update path is now structurally understood
- the HUD number renderer is identified and tied back to chunk-3 digit graphics

## Recommended Next Move

The next best targets are:

- `0x10ec`
- `0x11e0`
- `0x12ac`
- `0x01414`
- `0x021c4`

That group should answer:

- how current-piece and next-piece art are drawn
- how collision tests are represented on the logical board
- how completed rows are collapsed
- how the line-clear animation and row-removal path are staged

Update:

- those follow-up questions are now covered by [piece-and-board-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/piece-and-board-pass-2026-04-13.md) and [line-clear-theme-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-theme-pass-2026-04-13.md)
- held-input timing and the soft-drop lockout are now covered by [gameplay-input-timing-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-input-timing-pass-2026-04-14.md)
