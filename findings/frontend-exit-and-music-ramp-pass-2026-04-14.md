# Frontend Exit And Music Ramp Pass

Date: 2026-04-14

## Summary

This pass resolves the two main loose ends left in the frontend dispatcher work:

- what state `0` actually is in the dispatcher jump table
- what the hard exit-to-DOS path really coordinates before shutdown

The important result is that state `0` is not a hidden frontend handler, and state `3` is now clearly a coordinated music-fade-plus-runtime-exit path rather than a generic "leave menu" branch.

## State `0` Is A Direct No-Op Redispatch Slot

The dispatcher jump table at `0x3804` decodes to:

- state `0` -> `0x3938`
- state `1` -> `0x38bf`
- state `2` -> `0x38c9`
- state `3` -> `0x38d8`
- state `4` -> `0x3906`
- state `5` -> `0x390a`
- state `6` -> `0x3911`
- state `7` -> `0x3918`
- state `8` -> `0x391f`
- state `9` -> `0x3928`
- state `10` -> `0x3931`

State `0` does not call a handler at all.
It jumps straight to the dispatcher loop tail at `0x3938`, which only checks the break flag and redispatches.

Practical meaning:

- state `0` is a no-op or invalid-state redispatch slot
- it is not evidence of a missing frontend screen
- it should be modeled as a placeholder / redispatch state in the future port, not as a real menu

## The Hard Exit Path Uses A Timed Music Fade Controller

The state-`3` branch in `0x3830` now reads much more clearly:

1. poll `0x69b0` until it returns `0`
2. call `0x699e` with:
   - `EAX = 2`
   - `EDX = 0x40`
3. run the frontend animation fade-out through `0x62e0`
4. save `SETUP.DAT` through `0x3604`
5. call `0x05cc`

This means the exit path explicitly waits for any current music-ramp command to finish, then schedules a new ramp command before the final visual and runtime shutdown.

## `0x699e` / `0x69b0` / `0x6965` Form A Small Music Ramp System

These helpers are now coherent as a group.

### `0x699e`

Stores:

- ramp mode at `0x1991b`
- remaining ticks at `0x1991f`
- total ticks at `0x19923`

This is best modeled as:

- `schedule_music_volume_ramp(mode, duration_ticks)`

### `0x69b0`

Returns the current value at `0x1991b`.

Practical reading:

- `0` = no active ramp
- non-zero = ramp still active

### `0x6965`

Takes a progress-style value in `EAX`, clamps it to the configured total duration, converts it against the stored user music volume percent at `0x2c6eb`, and forwards the computed internal volume to `0x68bb`.

This is the helper that turns a ramp step into an actual music-volume update.

## Why The Ramp Meaning Is Now Clear

The clearest confirmation comes from `0x6544`, the music-track loader.

When switching tracks:

1. call `0x699e(mode = 2, duration = 0x20)`
2. wait until `0x69b0()` returns `0`
3. load the new track
4. call `0x699e(mode = 1, duration = 0x20)`

That gives the mode meanings cleanly:

- mode `2`
  fade out current music
- mode `1`
  fade in music

The ramp math in `0x65ba` / `0x6620` and `0x6965` matches that reading:

- mode `2` decreases from full configured volume toward zero
- mode `1` increases from zero toward the configured volume

So the hard exit path is specifically staging a music fade-out over `0x40` ticks before the final DOS exit.

## `0x05cc` Is The Final DOS Exit Helper

`0x05cc` now looks like a compact shutdown helper:

1. set BIOS video mode `3`
2. call `0x0970`
3. jump through the PMODE/W-era runtime exit trampoline at `0xe6e7` with `EAX = 0`

The exact role of `0x0970` is still unclear, but it is very small:

- load pointer from `0x184cf`
- copy byte `[ptr + 0x1c]` into `[ptr + 0x1a]`

That is most likely a tiny runtime-state or device-state handoff immediately before the protected-mode exit trampoline, not gameplay logic.

The important operational conclusion is already clear:

- `0x05cc` is not returning to gameplay or to the frontend
- it is the final DOS shutdown path

## Porting Impact

This pass improves the future source-port model in three ways:

- state `0` can stay a placeholder rather than forcing us to invent a missing screen
- exit-to-desktop behavior should preserve the original "finish or queue music fade-out, then close" intent
- the music system should have an explicit fade controller abstraction instead of baking fades into menu code

## Recommended Next Move

The strongest nearby target now is the remaining setup / audio glue around:

- `0x68d8`
- `0x6ca2`
- `0x6ea0`

That should tell us how much of the original audio backend we need to preserve behaviorally versus replace outright in the modern port.
