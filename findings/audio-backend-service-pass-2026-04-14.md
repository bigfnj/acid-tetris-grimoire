# Audio Backend Service Pass

Date: 2026-04-14

## Summary

This pass followed the remaining setup/audio glue around:

- `0x68d8`
- `0x6ca2`
- `0x6c1f`
- `0x6ea0`
- `0x6ee9`

The important result is that the original DOS audio stack is now better understood as layered services rather than one opaque subsystem:

- a music fade controller
- a periodic interrupt-driven audio service
- a second legacy channel/scheduler service with six logical slots
- cleanup hooks that unregister those services on shutdown

For the future Windows port, that is good news:
we almost certainly do not need to reproduce the IRQ/PIT mechanics literally.
We mainly need to preserve the behavioral contracts that sit above them.

## `0x68d8` Chooses And Arms Audio Service Layers

`0x68d8` is more than "configure audio" in the abstract.
It stores the requested mode in `0x2d297` and then arms different lower-level services depending on that mode.

High-confidence behavior:

- if called with `EAX = 1`
  it installs the periodic audio IRQ service through `0x6ea0`
- if called with `EAX = 0`
  it initializes the six-slot legacy scheduler through `0x6ca2`
  and then configures callback/function pointers and timing values for that service
- in both cases it registers the cleanup callback at `0x6948`

This means `0x68d8` is best treated as:

- audio-service bootstrap / mode selector

not just a leaf configuration helper.

## `0x6ea0` / `0x6ee9` Are A Matched Install/Remove Pair

### `0x6ea0`

If the service-active byte at `0x19933` is clear:

- installs handler `0x6e51`
- saves the previous vector
- programs the timer through `0x6f22` with divisor `0x4dae`
- sets `0x19933 = 1`

The installed handler at `0x6e51`:

- increments timing state
- services the music-volume ramp path through `0x6620`
- ends by calling the lower runtime/hardware output helper at `0xf863`

Practical reading:

- this is an interrupt-driven periodic audio service
- it is directly tied to the music fade/update layer

### `0x6ee9`

If `0x19933 == 1`:

- restores the saved vector
- reprograms the timer through `0x6f22` with a neutral/restored configuration
- clears `0x19933`

This is the clean teardown partner for `0x6ea0`.

## `0x6ca2` / `0x6c1f` Form A Second Legacy Audio Scheduler Layer

### `0x6ca2`

If the legacy-service flag at `0x19927` is clear:

- clears six 16-byte channel/slot records
- installs handler `0x6a98`
- seeds default callback pointers for the slot system
- seeds timing values at `0x2d30b` and `0x2d30f` to `0x10000`
- samples timing state through `0x6c62`
- stores the resulting base period in `0x2d313`
- arms the service and sets `0x19927 = 1`

This is best modeled as:

- initialize legacy multi-slot audio scheduler

### `0x6c1f`

If `0x19927` is set:

- restores the previously saved vector / timer state
- clears `0x19927`

This is the teardown partner for `0x6ca2`.

## What This Means About The Original Audio Architecture

The executable is no longer pointing toward a single monolithic "sound driver" function.

The current best model is:

1. gameplay/frontend code uses high-level helpers such as:
   - `load_music_track_by_index`
   - `play_audio_slot`
   - `set_music_volume_internal_from_percent`
   - `schedule_music_volume_ramp`
2. those helpers depend on:
   - a music fade/update controller
   - one periodic IRQ-driven service
   - one six-slot legacy scheduler service
3. shutdown paths remove those services through registered cleanup callbacks

That is a much more portable mental model than "reverse engineer every PIT register effect."

## Porting Impact

This pass gives us a cleaner modernization boundary.

For the Windows port, the likely preservation target is:

- music fade-in and fade-out behavior
- volume scaling behavior
- track-switch sequencing
- SFX slot playback behavior
- shutdown / cleanup ordering

The likely non-preservation target is:

- exact IRQ vector installation mechanics
- exact PIT divisor programming
- exact DOS timer and port I/O details

So the modern audio layer can probably be:

- one SDL audio/mixer service
- one music fade controller
- one SFX slot registry
- optional voice/channel bookkeeping only if gameplay evidence later proves it matters perceptibly

## Recommended Next Move

The strongest remaining audio-side target is now the device/configuration path centered on:

- `0x668c`
- `0x67f8`
- `0x6817`

That should tell us how much of the original "device family" setup is truly gameplay-relevant and how much is just DOS-era backend selection that we can replace outright.
