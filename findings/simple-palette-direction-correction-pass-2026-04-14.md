# Simple Palette Direction Correction Pass

Date: 2026-04-14

## Summary

This pass corrects the direction labels for the simple palette helpers:

- `0x6498`
- `0x64f0`

The important result is:

- `0x6498` is the simple palette **fade-in** helper
- `0x64f0` is the simple palette **fade-out** helper

The earlier labels were reversed because the shared scaler `0x284c` was being described as if `EBX` were a brightness term.
It is actually a **darkness** term.

## What `0x284c` Really Does

The raw assembly for `0x284c` is decisive:

- it computes `ECX = 0x40 - EBX`
- then multiplies each source palette byte by `ECX`
- then divides by `0x40`

So the effective scale factor is:

- `(0x40 - EBX) / 0x40`

That means:

- `EBX = 0`
  full brightness
- `EBX = 0x40`
  black

So `EBX` is not a direct brightness level.
It is a darkness amount.

## Corrected Meaning Of `0x6498`

`0x6498` computes:

- `EBX = 0x40 - elapsed_ticks`

and then calls `0x284c`.

Because `0x284c` uses `0x40 - EBX`, the actual visible scale factor becomes:

- `elapsed_ticks / 0x40`

So `0x6498` ramps from black toward full brightness over roughly 64 ticks.

It is the simple palette-only **fade-in** helper.

## Corrected Meaning Of `0x64f0`

`0x64f0` computes:

- `EBX = elapsed_ticks`

and then calls `0x284c`.

So the visible scale factor becomes:

- `(0x40 - elapsed_ticks) / 0x40`

That ramps from full brightness down toward black over roughly 64 ticks.

It is the simple palette-only **fade-out** helper.

## Why The Chunk-7 Fade Pair Stays Correct

This correction does **not** reverse the already named chunk-7 fade helpers.

The assembly there still lines up:

- `0x62e0` passes `EBX = elapsed_ticks`
  -> fade-out
- `0x63b8` passes `EBX = 0x40 - elapsed_ticks`
  -> fade-in

So only the simple-palette pair needed renaming.

## Transition Impact

This also sharpens two user-visible transitions.

### Frontend Entry Through `0x3830`

At the top of `0x3830`, the code:

1. saves the current working screen into the gameplay snapshot buffer
2. calls `0x64f0(0x2c303)`
3. copies the title/menu base screen into the working and visible pages
4. runs `0x63b8(0x2cdd3)`

So `0x64f0` is not a reveal.
It is a simple palette **blackout** stage that happens after the gameplay snapshot is saved and before the title/menu base is shown and the chunk-7/frontend fade-in begins.

### `New Game` Exit Through State `4`

At the end of the state-`4` dispatcher tail, after:

- gameplay base restore
- `0x05e0` run-start seeding

the code does:

- `0x6498(0x2c303)`
- `0x2574(0x2c303)`

So that final pair is best read as:

- palette-only reveal of the gameplay palette from black toward full brightness
- then a final direct full-palette upload to settle the DAC state exactly

That makes the gameplay handoff look more like a palette reveal than a fade-down.

## Bottom Line

The corrected palette-helper model is now:

- `0x62e0`
  animated frontend fade-out
- `0x63b8`
  animated frontend fade-in
- `0x64f0`
  simple palette-only fade-out
- `0x6498`
  simple palette-only fade-in

That correction makes the startup/frontend/gameplay transition story more internally consistent and safer to carry forward into the source port.
