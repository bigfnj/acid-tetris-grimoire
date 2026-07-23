# Gameplay Frame Driver Pass

Date: 2026-04-14

## Summary

This pass resolves the missing outer caller for `0x09c8`.

The real gameplay-step call is not only the startup initializer at `0x0707`. There is a second direct call at `0x0595`, and it sits inside the large session loop rooted at `0x023b`.

That gives us the gameplay cadence model we were missing:

- one outer render/flip loop
- an inner fixed-step catch-up loop
- gameplay logic, particle updates, and board-alert animation all advancing per step
- page flip plus pacing at the end of each outer frame

## `0x023b` Is The Game Session Loop

The large function rooted at `0x023b` performs game-session setup, then enters the persistent gameplay loop.

High-confidence behaviors from the recovered raw flat-binary disassembly:

- loads and decompresses the verified gameplay-side resources into their runtime buffers
- prepares the saved `50x50` alert region through `0x2188`
- loads the selected music track through `0x6544`
- calls `0x3830(1)` before entering the live loop
- then runs the long outer gameplay/render loop beginning at `0x048f`

The important direct proof is the inner-loop call:

- `0x0595 -> CALL 0x000009c8`

That is the real live gameplay-step invocation.

## `0x24d0` Returns A Catch-Up Step Count

`0x24d0` is more informative than we had documented before.

Two modes are visible:

- `AL = 1`
  seeds the pacing baseline by copying the current tick at `0x2d2a3` into `0x2ca93`
- `AL != 1`
  flips the page, waits for tick progress, enforces the minimum delay at `0x2c607`, then returns `current_tick - previous_tick`

That returned elapsed value is clipped to a maximum of `6`.

Practical consequence:

- `0x184cb` is not an arbitrary loop bound
- it is the number of logic steps the game wants to process before the next rendered frame, capped at `6`

## The Inner Gameplay Catch-Up Loop

Inside the outer loop, the game starts with `ECX = 1` and processes one simulation step for each value up to `0x184cb`.

Confirmed per-step order:

1. pre-step input/latch handling
2. in-game hotkey checks
3. `0x2e18` particle-object update
4. `0x09c8` gameplay update
5. `0x206c` board-alert tile update

Then, after all catch-up steps for that rendered frame are complete:

1. `0x2f24` draws the particle objects
2. `0x17719` updates another render layer that still needs a deeper pass
3. `0x24d0(0)` flips pages, paces output, and returns the next catch-up count

This is the clearest fixed-step source-port evidence we have so far.

## In-Game Hotkeys Inside The Session Loop

The same session loop also exposes some gameplay-time hotkeys through the live key-state table:

- key state `0x3f` decrements the stored music-volume percent at `0x2c6eb`
- key state `0x40` increments the stored music-volume percent at `0x2c6eb`
- both paths call `0x68bb`, which matches the already identified internal music-volume setter
- key states `0x41` and `0x42` decrement and increment the stored sound-FX volume at `0x2c6e7`
- the sound-FX path clamps to the upper bound already held in `EDI = 0x40`

That means the gameplay loop itself can manipulate runtime audio levels without returning to the frontend options screen.

## Porting Impact

This is a very important source-port milestone.

The future Windows port should model gameplay timing as:

- a fixed-step simulation loop
- multiple logic catch-up steps per rendered frame when needed
- a hard cap of `6` catch-up steps from the original executable
- gameplay, particles, and board-alert animation advancing in the same per-step order as the DOS original
- rendering and page-flip/pacing treated as a separate outer phase

That is a much stronger preservation target than a naive "one logic step per present" loop.

## Recommended Next Move

The strongest follow-up targets around this loop are:

- `0x17875`
  likely another per-frame visual or state-prep helper at the top of the outer loop
- `0x23ac`
  reached from the session loop when `0x184db == -2`
- `0xe6c8`
  the latch/helper call used immediately before the no-input side branch

Resolving those three should finish the outer gameplay driver story and remove most of the remaining ambiguity around pause/transition handling.

Update:

- the `0x17875`, `0x23ac`, and `0xe6c8` follow-up is now covered by [gameplay-escape-and-transient-render-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-escape-and-transient-render-pass-2026-04-14.md)
