# Frontend Presentation Sequencing Pass

Date: 2026-04-14

## Summary

This pass resolves the steady-state menu render loops around:

- `0x40c0` main menu
- `0x4584` options menu

The most important result is that the frontend menus are not using a simple "draw once per present" loop.

They use the same broad structure already seen on the gameplay side:

- clear previously drawn transient/pointfield pixels
- run one or more fixed logic steps based on `0x184cb`
- redraw the current frontend pointfield state once
- flush dirty cells once
- present once through `0x24d0`

So the frontend and gameplay paths are much more architecturally similar than they first appeared.

## Shared Steady-State Loop Pattern

Both `0x40c0` and `0x4584` follow the same steady-state presentation structure after `0x3df4` returns:

1. clear currently drawn chunk-7 points with `0x17613`
2. set `ESI = 1`
3. process one logic step for each `ESI <= 0x184cb`
4. after all catch-up steps, redraw current chunk-7 points with `0x175c5`
5. flush changed `8x4` cells with `0x17719`
6. call `0x24d0(0)` to present and get the next catch-up step count
7. if no exit is ready, repeat
8. if exit is ready, run `0x3ee4` and return the next frontend state

This means:

- `0x39c4` can advance the frontend pointfield multiple logical steps before the next present
- pointfield pixels are cleared before the catch-up phase and redrawn only once at the end
- menu text/highlight logic and pointfield logic share the same catch-up cadence

## Main Menu State Loop

### Local Roles

The main menu loop uses:

- `EBP`
  action mode
- `[esp]`
  exit-ready flag
- `[esp + 4]`
  return frontend state
- `0x1886f`
  selected menu row index

Action-mode meanings:

- `0`
  idle
- `1`
  pending activate / exit
- `2`
  pending move up
- `3`
  pending move down

### Input Model In Practice

The main menu mixes both input representations:

- live key-state table at `0x2c22b`
  used for the direct `Esc`-to-return path
- release latch through `0x964`
  used for `Enter`, `Up`, and `Down`

Important detail:

- `Esc` in the main menu only triggers the direct return path when the live-game flag at `0x2c72b` is set
- that path stages return state `2`, then uses the same gated exit flow as other menu actions

### How Highlight And Selection Really Work

Each logic step does:

1. process pending input
2. call `0x39c4`
3. if action mode is idle, call `0x3fd8(selected_row, 0)`
4. if action mode is non-idle, call `0x3fd8(selected_row, 1)` and wait for it to return `1`

The one-shot `0x3fd8(..., 1)` return gates state changes:

- mode `1`
  sets the exit-ready flag
- mode `2`
  decrements the selected row, with wrap
- mode `3`
  increments the selected row, with wrap

So the selected row does not change immediately when Up/Down is pressed.
It changes only after the highlight pulse helper signals completion.

### Immediate In-Place Text Mutations

Two main-menu rows mutate their text immediately before the next presented frame:

- music row
  clears the row band with `0x61c8`, cycles track via `0x6544`, and rewrites the selected track string
- level row
  clears the row band with `0x61c8`, increments/wraps the stored start level, and rewrites `Level %d`

That means the menu text content itself is part of the fixed-step state, not just a static background.

## Options Menu State Loop

### Local Roles

The options loop uses:

- `EBP`
  selected row index
- `[esp]`
  action mode
- `[esp + 4]`
  return frontend state
- `[esp + 8]`
  exit-ready flag

Action-mode meanings match the main menu:

- `0`
  idle
- `1`
  pending activate / exit
- `2`
  pending move up
- `3`
  pending move down

### Immediate Left/Right Edits

Unlike the main menu, the options menu uses live key-state checks for left/right edits on the current row.

Behavior:

- selected row `0`
  decrement/increment music volume
- selected row `1`
  decrement/increment SFX volume

Each edit:

- clears only the affected row through `0x61c8`
- updates the stored value
- rewrites just that formatted row in place through `0x0f787`
- calls `0x68bb` when the music-volume row changes

So the options screen is a real live-edit UI, not a redraw-on-exit menu.

### Gated Enter/Navigation Behavior

`Enter`, `Up`, and `Down` still use the release-latch path:

- `Enter` on row `2`
  stages return state `7` `Keyboard Setup`
- `Enter` on row `3`
  stages return state `1` `Main Menu`
- Up/Down
  use the same one-shot `0x3fd8(..., 1)` gate before changing selection

As with the main menu, exit is not immediate:

- `0x3fd8` must report completion first
- only then is the exit-ready flag set
- then the outer loop presents once more, runs `0x3ee4`, and returns

## What `0x3df4` / `0x3ee4` Mean More Precisely Now

The earlier transition labels were correct, but this pass sharpens them:

- `0x3df4`
  entry-reveal transition that shares the same catch-up/present model as the steady menu loops
- `0x3ee4`
  exit-conceal transition using the same structure in reverse

So the frontend presentation stack is:

- entry transition
- steady-state menu loop
- exit transition

with all three stages sharing the same `0x39c4 -> redraw -> flush -> 0x24d0` rhythm.

## Porting Impact

This matters for the future SDL port because it gives us a faithful menu-loop target:

- keep a fixed-step frontend update path
- allow multiple logical menu steps per present when catch-up is needed
- separate pointfield clear/project/draw from the actual present
- treat highlight movement and menu activation as gated animated state changes, not instantaneous row swaps
- allow row-local text rewrites for value changes instead of assuming a full-screen redraw model

## Recommended Next Move

The strongest remaining frontend-side target is now:

- the sibling loops in `0x49bc` and `0x5a94`

Those should confirm whether keyboard setup and sound setup use exactly the same steady-state sequencing and local action-flag model as the main and options menus.
