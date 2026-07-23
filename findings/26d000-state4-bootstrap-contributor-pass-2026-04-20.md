# `0x26D000` State-4 Bootstrap Contributor Pass

Date: 2026-04-20

## Summary

This pass resolved the exact visible bootstrap contributors inside `0x05e0` so the remaining state-`4`-unique first-flush surface is no longer described only in broad categories.

Main result:

- the staged state-`4` bootstrap inside `0x05e0` now breaks into exact visible contributor families:
  - high score
  - score
  - level
  - lines
  - seven per-piece counters
  - next-piece preview
  - alert reset
- `0x05e0` does **not** draw the live falling piece into the playfield
- the live piece is added later by the first real `0x09c8` step before the same presented frame

So the state-`4`-unique slice is now smaller and more exact:

- staged HUD + preview + piece-stat bootstrap

not:

- the live playfield piece itself

## New Owned Artifact

- [26d000-state4-bootstrap-contributor.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-state4-bootstrap-contributor.json)

This artifact records:

- the exact `0x2c58` sites inside `0x05e0`
- their resolved gameplay-HUD roles
- the preview / piece-stat bootstrap from `0x1348`
- the later first-live-piece draw that is **not** part of the state-`4`-unique slice

## Key Artifacts Reused

- [26d000-state4-first-gameplay-flush-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-first-gameplay-flush-pass-2026-04-20.md)
- [gameplay-helper-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-helper-pass-2026-04-13.md)
- [gameplay-edge-paths-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-edge-paths-pass-2026-04-14.md)
- [new-game-visible-bootstrap-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/new-game-visible-bootstrap-pass-2026-04-14.md)
- [behavior-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/behavior-spec.md)
- [ATET.EXE.flat-relocated.bin.000005e0.FUN_000005e0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-entry-pass/ATET.EXE.flat-relocated.bin.000005e0.FUN_000005e0.c)
- [ATET.EXE.flat-relocated.bin.00002c58.FUN_00002c58.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-helper-pass/ATET.EXE.flat-relocated.bin.00002c58.FUN_00002c58.c)

## Findings

### 1. The Four Non-Loop `0x2c58` Sites Inside `0x05e0` Resolve Cleanly

The non-loop counter draws in `0x05e0` are now specific enough to label directly from their backing variables:

- `ECX = [0x2c623]`
  - stored best score table head
  - gameplay HUD role: `HIGH-SCORE`
- `ECX = [0x2c6cb]`
  - current run score
  - gameplay HUD role: `SCORE`
- `ECX = [0x2c6df]`
  - current live level
  - gameplay HUD role: `LEVEL`
- `ECX = [0x2c72f]`
  - total cleared lines in the run
  - gameplay HUD role: `LINES`

That means the staged state-`4` first-flush surface definitely includes the gameplay HUD counters a player expects to see around the well.

### 2. The `0x05e0` Loop Clears The Seven Per-Piece Counters Before Spawn Bootstrap

The loop in `0x05e0`:

- sets `EAX = 0xf8`
- walks `EDX = 0x2f, 0x3a, 0x45, 0x50, 0x5b, 0x66, 0x71`
- passes `ECX = 0`
- uses `EBX = 4`
- repeats seven times

That is a clean structural match for the seven per-piece statistics rows shown on the right panel of gameplay.

So before `0x1348` runs, `0x05e0` stages:

- seven visible zeroed piece counters

### 3. `0x1348` Then Makes The Piece-Stats Panel And Preview Non-Zero Again

After the zeroing loop, `0x05e0` calls `0x1348`.

Already-owned `0x1348` behavior:

- promote previous next-piece ID into current-piece slot
- erase the old preview piece
- choose a new next-piece ID
- redraw the new preview piece
- increment the promoted current piece's statistics counter
- redraw that specific per-piece counter through `0x2c58`
- reset current piece state to:
  - `X = 4`
  - `Y = 0`
  - `rotation = 0`
  - gravity accumulator = `0`

So the state-`4` bootstrap does not leave the right panel as seven zeros.
By the time the first gameplay flush happens, the state-`4`-unique staged surface should include:

- six zero piece counters
- one promoted current-piece counter at `1`
- a newly chosen next-piece preview

That is a much sharper visual target than the earlier generic "piece bootstrap" phrasing.

### 4. The Live Piece In The Well Is Not Part Of The State-`4` Bootstrap Slice

This is the most important separation from the current pass.

The already-owned raw `0x09c8` spawn-edge slice shows:

- `0x1011`
  call `0x1348`
- later:
  - `0x10a1`
    call `0x10ec`
    draw the newly spawned live piece

So `0x1348` prepares the piece state and preview, but the visible live piece in the playfield is added later by the first real gameplay step.

That means the first gameplay-side present after state `4` contains both:

- state-`4`-unique staged bootstrap:
  - counters
  - preview
  - piece-stat panel
  - alert reset
- shared first-step gameplay addition:
  - live piece draw in the well

This separation is exactly what we needed to keep shrinking the unique state-`4` slice.

### 5. The Remaining State-`4`-Unique First-Flush Surface Is Now Concrete

After this pass, the unique staged bootstrap left dirty by `0x05e0` is no longer vague.

It is:

- `HIGH-SCORE`
- `SCORE`
- `LEVEL`
- `LINES`
- seven per-piece counters, with one promoted-piece counter redrawn to `1`
- next-piece preview redraw
- alert reset surface

And it explicitly excludes:

- the live current piece in the well
- tracked cleanup
- transient particle overlay
- ordinary first-step gameplay cadence

That is the tightest current description of the state-`4`-unique first-flush surface.

## Practical Porting Impact

For the future port, this means the restart bootstrap should be modeled in layers:

1. visible board / preview clear from the early upload
2. staged HUD + preview + piece-stat bootstrap
3. first gameplay-step live-piece draw and other shared frame work

If we later compare captured restart frames against the port, this is now specific enough to test panel-by-panel.

## Next Strongest Move

Do a focused static pass on the alert side only:

1. isolate exactly what the `0x206c(EAX = 1)` reset contributes to the state-`4` first-flush surface
2. separate that reset from the later ordinary `0x206c` call in the first live gameplay step
3. finish shrinking the state-`4`-unique first-flush slice to the smallest still-visible bootstrap set

## Bottom Line

The important closure is:

- the state-`4`-unique first-flush surface is now concrete enough to name, and it stops at staged HUD / preview / piece-stat / alert bootstrap

The live piece itself belongs to the shared first gameplay step, not to the unique state-`4` bootstrap.
