# Gameplay HUD Counter Label Closure Pass

Date: 2026-04-21

## Summary

This pass closes the small remaining HUD-label gap on the gameplay-side spawn / line-clear edge path.

Main result:

- `0x0ff2` is now closed as the live `LINES` redraw site
- `0x100c` is now closed as the live `SCORE` redraw site
- the nearby conditional `0x0ead` site is the live `LEVEL` redraw on ten-line rollover
- there is no matching `HIGH-SCORE` redraw in this local `0x09c8` slice

That means the gameplay-edge and gameplay-present-order artifacts no longer need to carry a vague "two HUD counters" label at this seam.

## New Owned Artifact

- [gameplay-hud-counter-label-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-hud-counter-label-closure.json)

This artifact records:

- the exact `0x2c58` sites in the owned `0x09c8` gameplay-edge slice
- the source values each site draws
- the on-screen coordinates and digit widths
- the match against the already-closed gameplay HUD layout

## Updated Living Artifacts

- [gameplay-present-order.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-present-order.json)
- [gameplay-edge-paths.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-edge-paths.json)

## Key Artifacts Reused

- [gameplay-edge-paths-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-edge-paths-pass-2026-04-14.md)
- [gameplay-helper-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-helper-pass-2026-04-13.md)
- [26d000-state4-bootstrap-contributor-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-contributor-pass-2026-04-20.md)
- [26d000-state4-bootstrap-layout-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-layout-pass-2026-04-20.md)
- [ATET.EXE.flat-relocated.bin.000009c8.FUN_000009c8.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-helper-pass/ATET.EXE.flat-relocated.bin.000009c8.FUN_000009c8.c)
- [ATET.EXE.flat-relocated.bin.00002c58.FUN_00002c58.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-helper-pass/ATET.EXE.flat-relocated.bin.00002c58.FUN_00002c58.c)
- [raw-0ec0-10b7.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-edge-pass/raw-0ec0-10b7.asm)

## Findings

### 1. `0x0FF2` Is The Live `LINES` Counter Redraw

The owned raw `0x09c8` slice immediately before `0x0ff2` does this:

- load `EBX = [0x2c72f]`
- add the current clear count
- store the result back to `[0x2c72f]`
- then call `0x2c58` with:
  - `EAX = 0x90`
  - `EDX = 0xc3`
  - `EBX = 4`
  - `ECX = updated [0x2c72f]`

Already-owned `0x2c58` behavior makes those argument roles concrete:

- `EAX`
  x-position
- `EDX`
  y-position
- `EBX`
  digit count
- `ECX`
  numeric value

So `0x0ff2` is:

- draw 4-digit value
- at `x = 144`, `y = 195`
- from `total cleared lines`

That exactly matches the already-closed gameplay HUD `LINES` site.

### 2. `0x100C` Is The Live `SCORE` Counter Redraw

The same slice then continues:

- load `ESI = [0x2c6cb]`
- compute the score increment from:
  - `current level + 1`
  - current clear-count score table entry
- store the result back to `[0x2c6cb]`
- then call `0x2c58` with:
  - `EAX = 0x24`
  - `EDX = 0x65`
  - `EBX = 7`
  - `ECX = updated [0x2c6cb]`

So `0x100c` is:

- draw 7-digit value
- at `x = 36`, `y = 101`
- from `current run score`

That exactly matches the already-closed gameplay HUD `SCORE` site.

### 3. The Nearby Conditional `0x0EAD` Site Is `LEVEL`, Which Helps Disambiguate The Pair

The same broader slice also contains the already-visible rollover path:

- compute `(total_lines % 10) + cleared_lines`
- if that crosses `10`, increment `[0x2c6df]`
- then call `0x2c58` with:
  - `EAX = 0x3c`
  - `EDX = 0x70`
  - `EBX = 3`
  - `ECX = updated [0x2c6df]`

That is the live `LEVEL` redraw at:

- `x = 60`
- `y = 112`

This matters because it proves the open pair at `0x0ff2` / `0x100c` were not ambiguous with level progression.
`LEVEL` already has its own nearby conditional redraw site.

### 4. There Is No Matching `HIGH-SCORE` Redraw In This Local Spawn / Clear Slice

The local `0x09c8` slice updates and redraws:

- `LEVEL` conditionally
- `LINES`
- `SCORE`
- preview state
- piece-stat counter
- live piece

But it does **not** contain a matching `0x2c58` redraw sourced from:

- `[0x2c623]`
  the gameplay-side `HIGH-SCORE` value

So this edge path is not a generic full-HUD refresh.
It is a narrower live-update band:

- redraw only the counters that this local gameplay event directly changes

### 5. The Gameplay-Edge Model Is Now More Specific

The old wording in the gameplay-edge artifact was:

- `0xff2` redraws one gameplay HUD counter
- `0x100c` redraws another gameplay HUD counter

That was directionally right but weaker than the evidence we now own.

The stronger model is:

- `0x0ead`
  redraw `LEVEL` on ten-line rollover
- `0x0ff2`
  redraw updated `LINES`
- `0x100c`
  redraw updated `SCORE`
- `0x1011`
  run preview and piece-stat bootstrap through `0x1348`

That is now precise enough to preserve directly in the future port and in later gameplay-edge handoffs.

## Practical Porting Impact

The live gameplay redraw contract around line-clear / spawn transition is now:

- if ten-line rollover occurs, redraw `LEVEL`
- redraw `LINES`
- redraw `SCORE`
- redraw preview and one piece-stat counter
- then continue to spawn-collision and live-piece draw

That is stronger than a generic "some HUD counters update here" rule and should reduce avoidable parity drift in the future port.

## Next Strongest Move

Pause this HUD-label branch here unless one of these becomes newly useful:

1. a port milestone wants all gameplay HUD redraw sites grouped into one direct implementation contract
2. a capture pass needs exact timing of counter changes against the first visible post-clear frame
3. a later static pass wants to answer whether `HIGH-SCORE` ever live-redraws during an in-progress run

If decompilation keeps moving now, another unresolved subsystem is likely a stronger use of time than further squeezing this now-closed label seam.

## Bottom Line

The important closure is:

- the remaining HUD-label gap in the `0x09c8` spawn / clear slice is now gone: `0x0ff2 = LINES`, `0x100c = SCORE`
