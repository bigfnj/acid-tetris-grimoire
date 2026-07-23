# Audio Caller Completion Pass

Date: 2026-04-14

## Summary

This pass closed the remaining small gaps around `0x6817`.

The strongest results are:

- the shipping flat-relocated payload contains exactly `13` direct calls to `0x6817`
- the two previously missed call sites are in keyboard setup at `0x4c9e` and `0x4cd7`
- no direct caller selects slot `2`
- the caller `EDX` term now looks like a simple two-class routing selector in the observed code:
  - `0` for UI and piece-lock style playback
  - `1` for warnings, line-clear rewards, top-out, and sleepy-timeout effects

That makes the audio system more reconstruction-friendly than before.

## Full Direct Caller Count

A direct scan of the flat relocated payload for `CALL rel32 -> 0x6817` finds these `13` call sites:

- `0x0d4e`
- `0x0f4f`
- `0x1065`
- `0x10dc`
- `0x2244`
- `0x2294`
- `0x22e3`
- `0x4445`
- `0x447c`
- `0x487f`
- `0x48ba`
- `0x4c9e`
- `0x4cd7`

That means the earlier event map was almost complete already, but keyboard setup was missing two navigation callers.

## The New Callers Are Keyboard Setup Navigation

`0x4c9e` and `0x4cd7` live inside `0x49bc`, the keyboard-setup handler.

Both calls:

- use slot `1`
- use centered pan `0x80`
- pass `EDX = 0`

So keyboard setup shares the same frontend row-move navigation sound as:

- main menu
- options menu

This raises confidence that slot `1` is the single shared UI navigation sound across the frontend menu family.

## Why The `EDX` Split Is Now Stronger

Three helper details matter here:

- `0x964` returns the input byte in `AL` and does not touch `EDX`
- `0x94c` clears the input latch and restores `EDX`
- `0x1f8c` preserves `EDX` across the alert-side helper path

That means caller-provided `EDX` values are not just noisy leftovers from nearby calls.

They survive through the short helper sequences that lead into `0x6817`.

## Current Best `EDX` Model

Observed direct callers divide cleanly into two classes.

### `EDX = 0`

Used by:

- piece lock / board impact at `0x0d4e`
- main menu navigation at `0x4445` and `0x447c`
- options menu navigation at `0x487f` and `0x48ba`
- keyboard setup navigation at `0x4c9e` and `0x4cd7`

Best reading:

- base playback class
- non-alert routing group
- default mixer voice family

### `EDX = 1`

Used by:

- generic and special line-clear rewards at `0x0f4f`
- top-out / game-over at `0x1065`
- sleepy idle timeout at `0x10dc`
- stack warnings at `0x2244`, `0x2294`, and `0x22e3`

Best reading:

- alert or reward playback class
- secondary mixer voice family
- coarse routing group for more prominent in-game effects

This is still an interpretation, but it is now a much tighter one than the earlier broad "small voice/channel offset" wording.

## Slot `2` Now Looks Unused In The Shipping Binary

Startup definitely registers slot `2` from chunk `23`.

But after scanning the flat relocated payload for all direct `0x6817` callers:

- no caller uses `EAX = 2`
- no currently known gameplay or frontend event resolves to slot `2`

Current best reading:

- slot `2` is loaded but unused in the shipped executable

That could mean:

- cut content
- a disabled UI/gameplay event
- a reserved fallback sound that never fires in normal execution

For the future port, this means slot `2` should be preserved in extracted assets and metadata, but it does not currently need a bound `SoundEvent` name.

## Porting Implication

The modern audio layer can now model playback routing much more simply:

- `SoundEvent`
- slot mapping
- per-play pan
- one small playback class flag or enum

That is enough to preserve the observed behavior without reproducing the original DOS mixer internals literally.
