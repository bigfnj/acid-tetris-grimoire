# Line Clear Theme Pass

Date: 2026-04-13

## Summary

This pass resolved the actual structure of the line-clear pipeline.

The gameplay loop does not treat a cleared row as a single instant operation. It stages the result in three layers:

- a level-themed row effect keyed by `current_level % 10`
- an eight-step pixel-drop animation for the board image
- a final logical-board compaction pass on the `10x20` cell buffer

That is a strong preservation result because it tells us exactly how to separate:

- rules/state updates
- row-clear presentation
- board-collapse timing

for the future Windows 11 source port.

## `0x1414` Is The Level-Themed Line-Clear Dispatcher

`0x1414` takes a cleared row index in `EAX`.

It:

- divides the current live level at `0x2c6df` by `10`
- uses the remainder to dispatch through a ten-entry table at `0x13ec`
- calls one of the helpers at:
  - `0x14b4`
  - `0x158c`
  - `0x1694`
  - `0x1770`
  - `0x1830`
  - `0x192c`
  - `0x19dc`
  - `0x1a9c`
  - `0x1b54`
  - `0x1c40`
- marks the affected board row dirty afterward through `0x2d30`

Best current reading:

- `0x1414` is the per-level line-clear presentation hook
- the level decides which particle or debris treatment gets used when a row disappears

## The Ten Leaf Helpers Are All Visual Row Effects

All ten helpers operate on the same physical board row:

- board origin `x = 0x6d`
- board origin `y = 0x15`
- width `0x50` bytes = `80` pixels = `10` cells
- height `8` scanlines per board row

Shared behavior visible across the helpers:

- sample the existing row pixels from the live `320x240` working screen at `0x2c727`
- iterate across the `80` pixels in each of the row's `8` scanlines
- use the sampled palette index as the color/material seed for the effect
- queue transient objects through `0x2f78` or `0x3034`
- clear the source scanline from the board image with `0x0e6b0`

That means the row effect is not just a generic flash. It is derived from the row's actual pixels, so the disappearing blocks visually break apart using the colors already on screen.

## `0x2f78` And `0x3034` Are Two Different Effect Queues

`0x2f78` allocates an effect object and derives velocity from:

- angle
- magnitude
- the sine table at `0x2c6cf`
- a lifetime/divisor parameter

Best reading:

- queue a particle using polar motion

`0x3034` allocates an effect object and derives velocity from:

- source position
- destination position
- lifetime/divisor parameter

Best reading:

- queue a particle using linear point-to-point motion

So the line-clear helper bank is really choosing among different ways to throw row pixels around:

- random sprays
- mirrored fans
- fixed-direction bursts
- sweeps toward fixed points
- sweeps toward moving targets

## Per-Helper Characterization

These are still descriptive labels, not final public names.

`0x14b4`

- samples the row and feeds colors into `0x2f78`
- uses randomized angle and speed values
- best reading: random debris burst

`0x158c`

- alternates direction based on pixel parity
- uses two opposing angle families
- best reading: alternating left/right burst

`0x1694`

- uses a steadily increasing angle across the row
- best reading: sweeping arc burst

`0x1770`

- uses a fixed speed with an angle sweep
- best reading: structured radial fan

`0x1830`

- uses `0x3034` with a moving target point
- best reading: pull or streak toward a drifting destination

`0x192c`

- uses `0x3034` toward a fixed point near the lower gameplay area
- best reading: converge-to-point effect

`0x19dc`

- uses `0x2f78` with a fixed direction and changing magnitude
- best reading: angled spray

`0x1a9c`

- uses `0x3034` with a row-dependent target coordinate
- best reading: row-biased convergence sweep

`0x1b54`

- splits the row in half and throws each half in opposing directions
- best reading: mirrored half-row burst

`0x1c40`

- also splits the row, but with a different angle pairing
- best reading: dual-band directional spray

The exact art/theme mapping is still open, but the execution role of each helper is now clear: they are all variations on "sample the row, emit themed debris, erase the row image."

## `0x1d04` Manages The Multi-Frame Collapse

`0x1d04` is the controller for post-clear resolution.

When called with `EAX = 1`:

- resets the current cleared-row batch index at `0x2c78b`
- resets the per-row animation step counter at `0x2c78f`

When called during live gameplay with pending cleared rows:

- lazily finds the topmost occupied board row through `0x1de8`
- converts that row into a pixel anchor at `0x2c787`
- uses the cleared-row list stored at `0x2c743`
- calls `0x1e34` once per frame step
- advances `0x2c78f` until it reaches `8`

After the eighth step:

- compacts the logical board buffer at `0x2c23b` with `0x0eb0c`
- clears the newly exposed top logical row with `0x0e6b0`
- advances to the next cleared row in the batch
- decrements the live cleared-row count at `0x184df`

This is why the gameplay loop short-circuits into `0x1d04` whenever cleared rows are still pending: the game is intentionally spending several frames resolving the presentation and collapse before normal input-driven play resumes.

## `0x1e34` Drops Board Pixels Down One Scanline Per Frame

`0x1e34` is the visual collapse worker.

Inputs at call time are effectively:

- `EAX` -> cleared row index
- `EDX` -> current scanline step `0..7`
- `EBX` -> top occupied pixel anchor

What it does:

- computes the active scanline inside the cleared row
- uses `0x0eb0c`, which is a true `memmove(dest=EAX, src=EDX, len=EBX)` helper
- copies one `80`-pixel board scanline downward inside the gameplay area
- works from bottom toward top so overlap is safe
- clears the newly exposed top scanline afterward
- marks the updated band dirty through `0x2d30`

Practical reading:

- one board cell row falls over `8` visual frames
- the cell buffer and the screen image stay in sync through a staged animation instead of an instant snap

## Why This Matters For The Port

This gives the future source port a clean model:

- detect full rows immediately
- record the cleared row indices
- run the row's level-specific debris effect once
- animate the visible collapse over `8` ticks
- compact the logical board only after the visual drop completes

That is a much better preservation target than treating line clears as a single "erase and shift" step.

## Open Questions

- Which specific visual or particle styling resource corresponds to each of the ten line-clear helpers?
- What user-facing meaning do the shared alert IDs in `0x2008` map to during line clears, warnings, and game-over?
- Which asset bank provides the per-level visual identity for these row effects, if it is not chunk `7`?

## Recommended Next Move

The strongest next follow-up is to tie the resolved line-clear execution path back to visible gameplay evidence:

- map `0x2008` more precisely
- correlate the ten line-clear helpers with captured gameplay footage
- identify the actual asset source for the per-level line-clear styling

Update:

- the later executable-side correction in [chunk7-usage-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/chunk7-usage-pass-2026-04-13.md) indicates chunk `7` belongs to the frontend animation system rather than the gameplay line-clear theme path
