# Chunk 7 Record Map Pass

Date: 2026-04-13

## Summary

This pass turns the chunk-7 frontend animation path into a usable structural model.

The current best interpretation is:

- chunk `7` decompresses into `8` frontend animation banks
- one bank is selected into the active `0x7000`-byte buffer at `0x2d223`
- the active bank contains `1024` records of `0x1c` bytes each
- each record is transformed by `0x39c4` into screen-space coordinates plus a shade value
- the menu and transition loops erase the previously drawn points, recompute the record bank, draw the new points, and flush only the dirty cells to VGA

This is now much stronger than the earlier "bitmap panel" hypothesis.

## The Startup Allocations Now Make Sense

The startup loader allocates:

- `0x38000` bytes for the full decompressed chunk-7 bank set
- `0x7000` bytes for the active bank at `0x2d223`
- `0x3000` bytes for an auxiliary bank-transition buffer at `0x2d24f`

That `0x3000` allocation is now explained:

- `1024` records
- `3` dwords per record
- `12` bytes each
- `1024 * 12 = 0x3000`

So the extra buffer is not a mystery scratch area.

It is the per-record delta table used to morph one frontend bank into the next.

## Active Record Layout

The strongest current field map for one `0x1c` chunk-7 record is:

- `+0x00`: source coordinate A
- `+0x04`: source coordinate B
- `+0x08`: source coordinate C
- `+0x0c`: projected screen `x`
- `+0x10`: projected screen `y`
- `+0x14`: projected color or shade byte
- `+0x18`: draw-success flag for the current frame

The names of the first three fields are still intentionally generic.

They behave like a point or pseudo-3D source vector, but that should stay provisional until the surrounding helpers at `0x3c70` and `0x3cdc` are understood better.

## `0x39c4` Projects The Bank Into Screen Space

The fallback assembly in [FUN_000039c4.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-update-followup/ATET.EXE.flat-relocated.bin.000039c4.FUN_000039c4.c) shows:

- a full walk over the active `0x7000` bank in `0x1c`-byte steps
- reads from `[record+0x00]`, `[record+0x04]`, and `[record+0x08]`
- heavy use of the sine table at `0x2c6cf`
- evolving global offsets or phase terms at:
  - `0x2d22b`
  - `0x2d227`
  - `0x2d213`
  - `0x2d23f`
  - `0x2d243`
  - `0x2d23b`
  - `0x2d237`
  - `0x2d247`

For each record, it writes:

- `[record+0x0c] = projected_x + 160`
- `[record+0x10] = projected_y + 120`
- `[record+0x14] = 0xef - clamped_shade`

That `+160/+120` centering matches a `320x240` screen-space projection.

The clamp range for the derived shade is `0..15`, which then becomes palette values in the `0xe0..0xef` range.

That makes the record bank look like a projected pointfield or starfield-style decorative system rather than sprite objects or literal images.

## `0x3c70` Builds The Next-Bank Delta Table

Raw disassembly of the relocated flat binary at `0x3c70` shows a tight `0x400`-iteration loop.

For each record index, it:

- reads source fields `+0x00/+0x04/+0x08` from the current bank selected by `0x2d233`
- reads the same three fields from the next bank selected by `0x2d24b`
- subtracts current from next
- stores the three signed differences into the `0x3000` buffer at `0x2d24f`

So `0x3c70` is best understood as:

- build `1024` delta vectors
- one signed 3-component delta per chunk-7 record

This also explains the top of `0x39c4`:

- when the animation timer reaches `0x258`
- it chooses a different random bank
- then calls `0x3c70` once to prepare the morph

## `0x3cdc` Applies One Morph Step Into The Active Bank

Raw disassembly at `0x3cdc` shows another `0x400`-iteration loop.

It takes `EAX` as an interpolation progress value and, for each record:

- reads the base source coordinates from the current bank in the full chunk-7 bank set at `0x2d22f`
- reads the matching `x/y/z` delta triple from `0x2d24f`
- multiplies each delta component by the progress value
- divides by `128`
- adds the scaled delta back onto the base source coordinates
- writes the interpolated source coordinates into the active bank at `0x2d223`

So `0x3cdc` does not project points to screen space.

It prepares an intermediate active bank whose source coordinates smoothly transition from one bank to another.

## The Frontend Animation Cycle Is Now Mostly Resolved

With `0x3c70` and `0x3cdc` understood, the chunk-7 cycle now reads like this:

1. one chunk-7 bank is active
2. for about `0x258` frames, `0x39c4` keeps projecting and animating that bank
3. at frame `0x258`, a different random bank is chosen
4. `0x3c70` builds the per-record delta table from current bank to next bank
5. for the next `0x80` frames, `0x3cdc(progress)` interpolates the source coordinates into the active bank
6. `0x39c4` projects the interpolated active bank each frame
7. at frame `0x2d8`, the next bank becomes the current bank and the timer resets

That means the frontend decoration is not just a single animated pointfield.

It is a bank-to-bank morphing pointfield system with a steady-state phase and a timed transition phase.

## `0x175c5` Draws One Point If The Target Pixel Is Empty

[FUN_000175c5.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-record-followup/ATET.EXE.flat-relocated.bin.000175c5.FUN_000175c5.c) takes:

- `EDI = y`
- `ESI = x`
- `CL = color`

It:

- rejects out-of-bounds coordinates outside `0 <= y < 240` and `0 <= x < 320`
- computes the linear offset into the working screen at `0x2c727`
- only writes the pixel if the destination is currently zero
- marks the corresponding dirty cell in the coarse map at `0x1ad97`
- returns `1` on success and `0` on failure

The caller stores that return value into `[record+0x18]`.

So `+0x18` is best understood as "this point was actually drawn last frame."

## `0x17613` Clears The Previously Drawn Point

[FUN_00017613.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-record-followup/ATET.EXE.flat-relocated.bin.00017613.FUN_00017613.c) uses the same coordinate convention as `0x175c5`.

It:

- validates the same `320x240` bounds
- writes zero to the target pixel in the working screen
- marks the same dirty cell map

The fade loops call it only when `[record+0x18] == 1`.

That means the frontend animation path is explicitly "erase last point, project, draw next point" rather than redrawing the whole screen blindly.

## `0x17719` Flushes Dirty `8x4` Cells To Planar VGA

[FUN_00017719.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-record-followup/ATET.EXE.flat-relocated.bin.00017719.FUN_00017719.c) confirms the dirty-map geometry:

- `40` columns by `60` rows
- one dirty cell per `8x4` pixel region on the `320x240` screen

The function:

- scans the dirty cell map at `0x1ad97`
- queues every non-zero dirty cell
- decrements the cell counter after queuing it
- copies each queued `8x4` cell from the working linear screen into VGA memory at `0x2c6ab`
- performs four plane writes through sequencer port `0x3c4`

The decrement behavior matters.

Because `0x175c5` and `0x17613` write the value `3` into the dirty map, a changed cell is refreshed across multiple flushes instead of a single immediate copy.

## Fade Loops Now Read Cleanly

Both [FUN_000062e0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000062e0.FUN_000062e0.c) and [FUN_000063b8.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000063b8.FUN_000063b8.c) follow the same core pattern:

1. walk all `1024` records
2. if `[record+0x18] == 1`, clear the previously drawn point through `0x17613`
3. call `0x39c4` to update all projected positions and shades
4. walk all records again
5. draw each new point through `0x175c5`
6. store the draw-success result back into `[record+0x18]`
7. flush the dirty `8x4` cells through `0x17719`
8. present the frame through `0x24d0`
9. repeat while the palette brightness ramps

The difference between the two functions is the direction of the brightness ramp:

- `0x62e0` fades toward darker output
- `0x63b8` fades toward brighter output

## Frontend Menu Handlers Use The Same Animation Core

The same helper chain shows up in the menu-side handlers:

- main menu `0x40c0`
- options `0x4584`
- keyboard setup `0x49bc`
- sound setup `0x5a94`

So the chunk-7 animation bank is a shared frontend decoration system, not a one-off transition effect.

## Decompilation Impact

This is enough structure to plan a faithful source-port subsystem later:

- keep a `1024`-record frontend point bank
- preserve the per-record screen `x`, screen `y`, shade, and drawn flag fields
- preserve the erase-then-project-then-draw loop
- preserve the coarse dirty-cell flush model if visual parity matters

For a first faithful Windows port, the coarse `8x4` dirty-cell batching does not need to survive literally.

What does need to survive is:

- the projected point motion
- the depth or shade-derived palette behavior
- the fade timing and overall frontend look

## Remaining Unknowns

The main open items are:

- the exact semantic meaning of source fields `+0x00/+0x04/+0x08`
- whether the bank records represent pure points or something more structured than a point cloud
- whether the point bank ever encodes anything more than projected points

## Recommended Next Move

The best next move is to sample one or two bank pairs directly from the chunk-7 raw data and confirm the morph visually or statistically, now that the interpolation path is clear.
