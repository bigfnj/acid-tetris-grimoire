# Gameplay Entry Pass

Date: 2026-04-13

## Summary

This pass followed the code that runs immediately after the frontend hands control to play, especially the state-`4` new-game path and the high-score row animation helper reached from the score-entry screen.

The useful result is that we now have a much clearer boundary between:

- frontend completion
- new-game initialization
- live-game flag setup
- high-score row animation after a qualifying score

## `0x05e0` Starts A New Game Session

`0x05e0` is the clearest new-game initialization function seen so far.

High-confidence behaviors:

- resets the current score at `0x2c6cb` to `0`
- resets the secondary run stat at `0x2c72f` to `0`
- copies the menu-selected starting level from `0x2c713` into the current in-game level at `0x2c6df`
- redraws the gameplay-side HUD and frame buffers
- clears the per-piece statistics area rooted at `0x2c6f3`
- redraws visible numeric HUD fields:
  - current score
  - current level
  - high score
  - secondary run stat
- refills the random table through `0x32b0`
- calls the early gameplay setup helpers at `0x1330`, `0x09c8`, and `0x1348`
- chooses an initial tetromino index modulo `7` and stores it at `0x2c6ef`
- computes and stores the live gravity increment at `0x2c717` as `(current_level << 9) + 0x200`
- sets the live-game flag at `0x2c72b` to `1`
- triggers the popup/tile animation helper at `0x206c`

Update:

- the `0x09c8(EAX = 1)` call in this path is now known to reset the gameplay repeat timers and the held-Down lockout field before the new run starts
- the gravity formula and repeat-timer model are documented in [gameplay-input-timing-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-input-timing-pass-2026-04-14.md)

Practical source-port reading:

- this is the right place to model "start a fresh run"
- it is not just a render refresh
- it owns score reset, level bootstrap, first-piece setup, and the transition into active gameplay

## `0x2d88` Clears A Large Runtime Region Map

`0x2d88` is small but important.

It zeroes `0x960` bytes at `0x1ad97`, and the frontend dispatcher calls it immediately before `0x05e0` when leaving the main menu through the new-game path.

Current best interpretation:

- clear the gameplay dirty-cell map or a closely related playfield-side region map before a fresh run begins

This lines up well with the already established dirty-rectangle helper at `0x2d30`.

## `0x5868` Animates A High-Score Row

`0x5868` is called from the high-score entry path inside `0x50b0`.

It uses:

- the wave/offset table rooted at `0x2c6cf`
- the animation phase at `0x19913`
- three text columns at:
  - `0x2d0d3`
  - `0x2cc93`
  - `0x2cb53`

The helper repeatedly redraws one score row with animated horizontal offsets derived from the wave table.

Two operating modes are visible:

- `EDX = 0`
  continuous redraw/update mode
- `EDX = 1`
  one-shot highlight mode that returns `1` when the animation reaches a completion boundary

Practical consequence:

- the high-score name-entry flow is not static text
- the selected row is given its own animated treatment before the screen settles back to the normal table display

## What This Means For The Port

The port structure is getting clearer:

- frontend menu code should hand off to an explicit `start_new_game()` routine modeled on `0x05e0`
- the run-start path should preserve the original order:
  reset stats, seed/randomize, prepare first piece, compute timing, then mark the game live
- the high-score screen should preserve the animated row treatment, not just the final stored data

## Recommended Next Move

The next strongest reverse-engineering targets are:

- `0x1330`
- `0x1348`
- `0x09c8`
- `0x6274`
- `0x2c58`

That group should answer:

- how the first piece and next piece are initialized
- how the HUD box primitives are drawn
- which exact value is represented by the secondary score-table column
