# Frontend Shared Present Boundary Pass

Date: 2026-04-14

## Summary

This pass closes the remaining ambiguity around the frontend-side callers of the shared dirty flush backend.

The important result is that the menu transitions and the frontend pointfield fade loops all share the same end-of-frame presentation boundary:

- redraw into the linear working screen
- flush dirty `8x4` cells through `0x17719`
- present once through `0x24d0`
- store the returned catch-up count in `0x184cb`

That makes the frontend renderer much more coherent than it first looked.

## `0x3ebc` And `0x3fad` Are Not Special-Case Effects

These two sites are simply the entry-transition and exit-transition uses of the common present boundary.

### `0x3df4` Entry Transition

After clearing previously drawn chunk-7 points and running one or more catch-up logic steps:

- redraw all projected points through `0x175c5`
- call `0x17719` at `0x3ebc`
- call `0x24d0(0)`
- store the returned step count in `0x184cb`

That loop repeats until the local transition frame counter reaches `0x30`.

### `0x3ee4` Exit Transition

The exit transition is structurally the same:

- clear old points
- run catch-up logic steps
- redraw all points
- call `0x17719` at `0x3fad`
- call `0x24d0(0)`
- store the returned step count in `0x184cb`

The only real difference is the text parameter sent to `0x60cc`:

- entry: increasing reveal
- exit: `max(0x2f - frame_index, 0)`

So `0x3ebc` and `0x3fad` are best understood as symmetric transition-frame present boundaries, not unique effect logic.

## The Pointfield Fade Loops Use The Same Present Rhythm

The fade helpers:

- `0x62e0` frontend pointfield fade-out
- `0x63b8` frontend pointfield fade-in

use the same frame cadence:

1. clear previously drawn points
2. advance `0x39c4` for one or more catch-up steps from `0x184cb`
3. compute a temporary scaled palette with `0x284c`
4. upload that palette with `0x2574`
5. redraw current points through `0x175c5`
6. flush dirty cells through `0x17719`
7. present through `0x24d0`
8. store the returned catch-up count in `0x184cb`

The difference from the text transitions is that these loops are time-based instead of fixed-48-frame:

- they run until the tick delta from `0x2d2a3` reaches `0x40`

So the frontend uses one presentation architecture for:

- text entry transition
- text exit transition
- chunk-7 fade in
- chunk-7 fade out
- steady menu loops already resolved earlier

## Why This Matters

This is a strong architectural checkpoint for the future port.

We now have a stable frontend model:

- logical updates can run multiple times before a present
- pointfield pixels are cleared first and redrawn once at the end of the catch-up batch
- palette transforms happen before the flush/present boundary
- `0x17719` is the last dirty-region backend step before the frame is presented

That is very close in shape to the outer gameplay loop, which means the original game is more internally consistent than it first appeared.

## Porting Impact

For the C++23/SDL3 port, the faithful renderer should preserve this order conceptually:

1. clear frame-local transient frontend pixels
2. run fixed-step frontend logic zero or more times
3. apply any palette-state update for the current presented frame
4. redraw the current frontend pointfield state
5. present

We do not need literal planar VGA dirty-cell writes, but we do want the same sequencing and catch-up behavior.

## Bottom Line

This pass turns `0x3ebc` and `0x3fad` from anonymous call sites into something useful:

- they are the shared frontend frame-present boundary around `0x17719 -> 0x24d0`

That closes most of the remaining uncertainty in the frontend render/presentation stack.
