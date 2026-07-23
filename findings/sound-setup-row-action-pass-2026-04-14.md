# Sound Setup Row Action Pass

Date: 2026-04-14

## Summary

This pass resolves the exact row-action semantics inside `0x5a94`, the first-run sound setup menu.

The most useful result is that this screen is a pure configuration UI.
It does **not** live-reinitialize or preview the audio backend while the user edits rows.

Instead it:

- mutates the saved setup values in memory
- rewrites the affected row text in place
- exits through the normal frontend transition path

That is a very helpful preservation boundary for the future port.

## Enter-Key Jump Table Is Now Resolved

The `Enter` handler in `0x5a94` dispatches through the jump table at `0x5a7c`.

Resolved row actions:

- row `0`
  cycle sound device index at `0x2c6d7`
- row `1`
  cycle mixing-rate index and rewrite `0x2c70f`
- row `2`
  toggle stereo flag at `0x2c723`
- row `3`
  toggle bit-depth flag at `0x2c6e3`
- row `4`
  stage return state `2` `Play Game`
- row `5`
  stage return state `3` `Exit to Dos`

So the first-run sound screen is fully menu-driven rather than a special external setup tool.

## Device Row

Row `0`:

- clears the device row band through `0x61c8`
- increments the device index at `0x2c6d7`
- wraps from `5` back to `0`
- rewrites the device-name row from the table at `0x18878`

This confirms the menu is cycling a fixed six-entry device table.

## Mixing-Rate Row

Row `1`:

- clears the mixing-rate row through `0x61c8`
- increments the local rate-table index
- wraps from `6` back to `0`
- writes the selected rate back into `0x2c70f`
- rewrites the visible `Mixing Rate:%d` row in place

The copied seven-entry rate table at `0x3570` is:

- `11025`
- `16357`
- `22050`
- `27562`
- `33074`
- `38586`
- `44100`

Those values are not just round user-facing defaults.
They look like engine/backend-oriented supported rates, which is useful context for the future port.

## Stereo And Bit-Depth Rows

Row `2`:

- clears the stereo row through `0x61c8`
- toggles the boolean at `0x2c723`
- rewrites the row as either:
  - `Stereo`
  - `Mono`

Row `3`:

- clears the bit-depth row through `0x61c8`
- toggles the boolean at `0x2c6e3`
- rewrites the row as either:
  - `16 Bit`
  - `8 Bit`

These are immediate in-memory config edits, not delayed apply actions.

## Play / Exit Rows

Row `4` `Play Game`:

- stages return state `2`
- stages action mode `1`
- exits through the normal `0x3fd8` gate and final `0x3ee4` path

Row `5` `Exit to Dos`:

- stages return state `3`
- stages action mode `1`
- exits through the same gated frontend path

## Esc Behavior

`Esc` does **not** stage the DOS-exit path.

It stages the same return state as `Play Game`:

- return state `2`
- action mode `1`

So `Esc` in first-run sound setup means:

- leave setup and continue

not:

- abort to DOS

That is an important behavioral detail for the future port.

## Negative Result: No Live Audio Reinit

Within `0x5a94`, this pass found no calls to:

- `0x668c` audio backend init
- `0x67cf` audio shutdown
- `0x68bb` music-volume apply
- `0x6544` music-track load

That means row edits do **not**:

- reinitialize the device backend
- test-play sound
- reconfigure the mixer live

Current best reading:

- this menu only edits the persisted configuration block in memory
- backend initialization happens later through the normal startup / runtime path

## Porting Impact

For the future Windows port, the faithful first pass should treat first-run sound setup as:

- a frontend menu sibling of main/options/keyboard
- a pure configuration editor
- a screen that rewrites row text immediately
- a screen that applies changes only through later normal audio initialization

That means we do not need to overcomplicate it with live device-probe or live test-play behavior unless we add that deliberately as a later modernization feature.
