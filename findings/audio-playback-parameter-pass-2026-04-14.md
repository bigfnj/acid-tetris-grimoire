# Audio Playback Parameter Pass

Date: 2026-04-14

## Summary

This pass focused on the remaining uncertain caller-controlled parameters to `0x6817`.

The most useful outcome is that the helper now looks much less like a black-box "play sound" call and much more like:

- choose an SFX slot
- choose a mixer voice / channel index
- set per-play pan
- scale the user-facing SFX volume
- start playback using the slot object's metadata

That is a very good preservation boundary for the future SDL port.

## `0x6817` Parameter Model

Current best interpretation:

- `EAX`
  SFX slot index
- `EDX`
  small voice/channel offset or playback-class offset
- `ECX`
  pan or stereo balance term
- `EBX`
  user-facing SFX percent

Inside `0x6817`:

1. scale `EBX` from user-facing percent into engine range `0..0x40`
2. if music is active, read the first byte of the current music object at `0x2d29f`
3. add caller `EDX` to that byte
4. use the resulting value as the playback-side index sent into the lower mixer interface
5. feed:
   - scaled volume
   - caller `ECX`
   - slot metadata
   into the lower playback chain

That means the caller is not directly choosing pitch or raw frequency.
It is choosing:

- which sound slot to use
- what pan/balance to use
- which mixer voice/class offset to bias toward

## Why `ECX` Now Looks Like Pan

The strongest evidence comes from the gameplay call at `0x0d4e`.

Right before `0x6817`, the code computes:

- `ECX = <derived value> + 0x80`

and that value depends on gameplay-side spatial state rather than a static UI constant.

In contrast:

- menu and warning calls almost always use `ECX = 0x80`

That is a classic center-pan pattern:

- `0x80`
  centered
- values offset from `0x80`
  spatially biased left or right

So the future port should preserve a per-play pan value, even if the exact DOS mixer internals are replaced.

## Why `EDX` Now Looks Like A Voice/Class Offset

The lower calls inside `0x6817` all use:

- `current_music_first_byte + caller_EDX`

as the main playback-side index.

That combined value is then passed to a lower mixer interface before sample metadata is supplied.

This makes `EDX` unlikely to be:

- direct pitch
- direct volume
- direct pan

It fits much better as:

- mixer voice offset
- playback class
- small channel-group bias

The observed call-site values also fit that reading:

- menu navigation
  usually `EDX = 0`
- gameplay / warning / larger in-game effects
  commonly `EDX = 1`

So this appears to be a coarse playback-routing choice rather than a user-facing acoustic parameter.

## Useful Call-Site Patterns

### Menu Navigation

Main menu, keyboard setup, and sound setup all use:

- slot `1`
- pan `0x80`
- a zero-like `EDX` path

That is consistent with:

- one shared UI navigation sound
- centered playback
- same playback class each time

### Board / Gameplay-Spatial Effects

The gameplay-side call at `0x0d4e` uses:

- slot `0`
- pan derived from board position around `0x80`
- a zero-like `EDX` path

That is our clearest evidence that:

- the game does spatialize at least some SFX horizontally

### Warning / Alert Effects

Warning-related paths such as `0x2244`, `0x2294`, and `0x22e3` use:

- slots `4` and `3`
- pan `0x80`
- `EDX = 1`

That fits a pattern of:

- centered non-spatial warning sounds
- routed through a different playback-class offset than menu UI sounds

## Lower Playback Chain

The immediate lower helpers called by `0x6817` are thin dispatch wrappers:

- `0xb247`
- `0xb26f`
- `0xb25e`
- `0xb286`

They forward into a vtable-like interface rooted at `0x42c87`.

The call sequence strongly suggests:

1. set channel/voice volume
2. set channel/voice pan
3. bind sample/resource
4. start playback with slot metadata

That is not enough to fully rename those helpers yet, but it is enough to justify the new `0x6817` model above.

## Porting Impact

This pass gives us a much better SDL-side contract:

- keep numbered SFX slots
- keep user-facing SFX volume scaling
- keep per-play pan
- keep at least two coarse playback classes or voice-offset groups
- do not worry about literally reproducing the DOS mixer/vtable layer

The most important audible parity targets are now:

- centered versus spatialized playback
- relative SFX loudness
- event-to-slot mapping
- whether UI sounds and gameplay sounds share or separate playback classes

## Recommended Next Move

The strongest next move is to document event-to-slot mapping more explicitly from the current call sites, especially:

- slot `0`
- slot `1`
- slots `3` and `4`
- slots `5` and `6`

That should give us the first draft of a source-port `SoundEvent -> slot/pan/class` table.
