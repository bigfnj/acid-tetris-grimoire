# High-Score Live Redraw Closure Pass

Date: 2026-04-21

## Summary

This pass closes the follow-up HUD question left open after the `LINES` / `SCORE` label pass:

- does the gameplay `HIGH-SCORE` field live-redraw during an in-progress run?

Main result:

- owned executable evidence now says no ordinary in-progress gameplay redraw path updates the gameplay HUD `HIGH-SCORE` field
- `HIGH-SCORE` is present in the gameplay HUD bootstrap through `0x05e0`
- but the live gameplay update band around `0x09c8` redraws only:
  - `LEVEL`
  - `LINES`
  - `SCORE`
  - preview and piece-stat counters
- the only owned writes to `[0x2c623]` belong to the high-score-table qualification path, not to live gameplay HUD refresh

So the current best model is now specific enough to close:

- gameplay `HIGH-SCORE` is a bootstrap / static HUD field during ordinary in-progress play in the owned shipped binary

## New Owned Artifact

- [high-score-live-redraw-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/high-score-live-redraw-closure.json)

This artifact records:

- the owned `0x2c58` caller set
- which of those callers source `[0x2c623]`
- where `[0x2c623]` is written
- the resulting closure rule for gameplay HUD behavior

## Updated Living Artifacts

- [gameplay-present-order.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-present-order.json)
- [gameplay-edge-paths.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-edge-paths.json)
- [behavior-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/behavior-spec.md)

## Key Artifacts Reused

- [gameplay-hud-counter-label-closure-pass-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-hud-counter-label-closure-pass-2026-04-21.md)
- [gameplay-helper-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-helper-pass-2026-04-13.md)
- [26d000-state4-bootstrap-contributor-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-contributor-pass-2026-04-20.md)
- [frontend-secondary-state-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-secondary-state-pass-2026-04-14.md)
- [ATET.EXE.flat-relocated.bin.000005e0.FUN_000005e0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-entry-pass/ATET.EXE.flat-relocated.bin.000005e0.FUN_000005e0.c)
- [ATET.EXE.flat-relocated.bin.000009c8.FUN_000009c8.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-helper-pass/ATET.EXE.flat-relocated.bin.000009c8.FUN_000009c8.c)
- [raw-50b0-5a93.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/highscore-edge-pass/raw-50b0-5a93.asm)

## Findings

### 1. The Owned `0x2C58` Gameplay Caller Set Is Small And Stable

The current owned `0x2c58` caller set is:

- `0x05e0`
  new-game bootstrap
- `0x09c8`
  live gameplay update step
- `0x1348`
  preview / piece-stat update helper

Within that set:

- `0x05e0` draws:
  - `LINES`
  - `SCORE`
  - `LEVEL`
  - `HIGH-SCORE`
  - seven zeroed piece-stat counters
- `0x09c8` draws:
  - conditional `LEVEL`
  - `LINES`
  - `SCORE`
- `0x1348` draws:
  - one per-piece statistics counter

So the gameplay-side direct counter redraw model is now concrete enough to compare by field instead of by broad category.

### 2. Only `0x05E0` Draws The Gameplay HUD `HIGH-SCORE` Field

The bootstrap-side `0x05e0` path contains the already-closed site:

- `0x06ed`
  load `ECX = [0x2c623]`
- `0x06f3`
  call `0x2c58`

With the already-closed draw arguments:

- `EAX = 0x24`
- `EDX = 0x59`
- `EBX = 7`

That is the gameplay HUD `HIGH-SCORE` field at:

- `x = 36`
- `y = 89`

No owned `0x09c8` or `0x1348` site matches that source field or those coordinates.

### 3. The Live `0x09C8` Band Redraws Only The Fields It Directly Changes

The owned `0x09c8` redraw band now closes as:

- `0x0ead`
  redraw `LEVEL` on ten-line rollover
- `0x0ff2`
  redraw `LINES`
- `0x100c`
  redraw `SCORE`
- `0x1011`
  redraw preview and one piece-stat counter through `0x1348`

There is no matching live redraw of:

- `[0x2c623]`
- gameplay HUD coordinates `x = 36`, `y = 89`

So the local gameplay redraw band is selective rather than whole-panel.

### 4. Owned Writes To `[0x2C623]` Belong To High-Score Qualification, Not To Live Gameplay HUD Refresh

The owned code-side writes to `[0x2c623]` appear in the high-score qualification flow at `0x50b0`:

- compare current run score `[0x2c6cb]` against the saved five-entry table rooted at `[0x2c623]`
- if the run qualifies, shift lower entries
- store the current run score into the record score field at `[slot + 0x2c623]`
- store total lines into `[slot + 0x2c627]`

That is persistence-side high-score-table maintenance.
It is not a gameplay HUD redraw path.

So even when `[0x2c623]` changes, the owned change site is:

- post-run qualification flow

not:

- an in-progress gameplay HUD update loop

### 5. The Best Current Model Is Now Specific Enough To Close

Putting the evidence together:

- gameplay bootstrap draws `HIGH-SCORE`
- live gameplay updates redraw `LEVEL`, `LINES`, and `SCORE`
- high-score qualification can modify the saved high-score table after a run
- but there is no owned ordinary in-progress redraw path for the gameplay HUD `HIGH-SCORE` field

So the practical preservation rule is now:

- during ordinary in-progress play, the gameplay HUD `HIGH-SCORE` field is static after bootstrap
- it should not be modeled as a per-clear or per-score-change live counter unless new evidence appears

## Practical Porting Impact

The future port should currently preserve gameplay HUD redraw behavior as:

- bootstrap:
  - `HIGH-SCORE`
  - `SCORE`
  - `LEVEL`
  - `LINES`
- ordinary in-progress live redraw:
  - `LEVEL` conditionally
  - `LINES`
  - `SCORE`
  - preview and piece-stat counters

That is a cleaner and more faithful rule than treating the whole left HUD block as a fully live-refreshed panel.

## Next Strongest Move

Pause the gameplay-HUD redraw branch here unless one of these becomes newly useful:

1. a port milestone wants all gameplay HUD redraw ownership collapsed into one implementation contract
2. a capture pass wants frame-accurate confirmation of HUD stability during long in-progress play
3. a later static pass finds a previously unowned gameplay path that sources `[0x2c623]` into `0x2c58`

If decompilation keeps moving now, another unresolved subsystem is likely a stronger use of time.

## Bottom Line

The important closure is:

- in the owned shipped gameplay paths, `HIGH-SCORE` behaves like a bootstrap/static HUD field, not an ordinary in-progress live redraw field
