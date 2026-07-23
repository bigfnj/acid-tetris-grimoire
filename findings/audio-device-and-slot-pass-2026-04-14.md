# Audio Device And Slot Pass

Date: 2026-04-14

## Summary

This pass tightened the higher-level audio path centered on:

- `0x668c`
- `0x676a`
- `0x67aa`
- `0x67cf`
- `0x67f8`
- `0x6817`

The practical result is that the audio layer now separates cleanly into:

- backend setup from persisted configuration
- music-object start/stop
- SFX slot registration
- per-slot playback
- startup fallback to "no audio" when backend init fails

That is exactly the kind of split we want before designing the future SDL audio layer.

## `0x668c` Uses The Saved Setup State To Initialize Audio

`0x668c` is the real high-level audio-backend initializer.

The startup path calls it twice:

1. first with the persisted setup values:
   - device index from `0x2c6d7`
   - mixing rate from `0x2c70f`
   - bit-depth flag from `0x2c6e3`
   - stereo flag from `0x2c723`
2. if that fails, a second time with device index `4`

That second call is the key behavioral result:

- startup explicitly falls back to the device-table entry `4`
- from the earlier sound-setup work, entry `4` is `None`

So the original executable behavior is:

- try configured audio backend
- if init fails, fall back to `None`
- continue startup rather than aborting the game

That is a very useful preservation target for the modern port.

### What `0x668c` Records

The function resets the slot table at `0x2d257`, clears audio-active state, and writes a compact configuration block rooted at:

- `0x1a337`
  device index
- `0x1a339`
  mixing rate
- `0x1a33b`
  format flags
- `0x1a33d`
  constant `0x1000`

The flag byte/word at `0x1a33b` is built from:

- bit `0`
  set when the stacked boolean argument is non-zero
  this matches the stereo/mono setup field
- bit `1`
  set when the register boolean argument is non-zero
  this matches the 16-bit/8-bit setup field

So this initializer is definitely consuming the persisted sound-setup state rather than using unrelated runtime values.

### Backend-Init Outcome

If the lower init chain succeeds:

- `0x19917` is set to `1`
- `0x1a33f` is set to `2`
- cleanup callback `0x67cf` is registered
- the function returns success

If the lower init chain fails:

- it returns `0`
- startup immediately retries with device `4` `None`

Porting implication:

- we do not need device-family emulation
- we do want the same tolerant behavior:
  try configured backend, then gracefully continue without audio if needed

## `0x676a` / `0x67aa` Manage The Current Music Object

### `0x676a`

This helper takes a lower-level music handle, stops any current music through `0x67aa`, creates or activates a music object from the new handle, stores it at `0x2d29f`, and marks music active at `0x2d29b`.

It also:

- sets `0x19be3 = 1`
- reads the first byte of the created object
- stores that byte plus `2` into `0x1a33f`

Current best model:

- `load_and_start_music_from_chunk_handle`

This matches the track-switch flow already seen in `0x6544`.

### `0x67aa`

This is the paired stop helper.

If music is active:

- clear `0x2d29b`
- destroy or release the current music object at `0x2d29f`
- set `0x1a33f = 2`

Current best model:

- `stop_current_music_if_active`

## `0x67cf` Is Full Audio Cleanup

This helper is now much clearer than before.

It:

- iterates through all `16` SFX slots and releases them through `0x680b`
- clears the backend-active flag at `0x19917`
- stops current music through `0x67aa`
- runs two lower-level backend cleanup calls

Current best model:

- `shutdown_audio_backend_and_release_resources`

That is also the callback registered during backend init, which fits the lifecycle cleanly.

## `0x67f8` Registers SFX Slot Objects

This helper is no longer just "load audio slot from chunk handle" in vague terms.

It:

- takes a positioned chunk handle in `EDX`
- passes it into the lower loader at `0xa1b6`
- stores the resulting object pointer into `0x2d257[slot_index]`

Startup uses it to register the twelve recovered WAV chunks into slots `0..11`.

So its real role is:

- create an SFX object from a chunk handle and place it in the numbered slot table

That numbered slot registry is almost certainly the right preservation abstraction for the modern port.

## `0x6817` Plays One Registered SFX Slot

`0x6817` now has a stronger behavioral shape even if every sub-parameter is not fully named yet.

Confirmed behavior:

- caller chooses the slot index in `EAX`
- caller supplies a small offset or variant term in `EDX`
- caller supplies a commonly `0x80`-valued term in `ECX`
- caller supplies user-facing SFX percent in `EBX`

Inside the helper:

- SFX percent is mapped into the engine scale `0..0x40`
- the slot object is fetched from `0x2d257[slot]`
- if music is active, a byte from the current music object is added into the playback-side value path
- slot metadata fields are forwarded into a lower playback chain

The `ECX` meaning is still not final, but the calling pattern suggests it is a playback-control term such as pan, priority, or a fixed playback class value rather than a game-rule value.

Practical porting conclusion:

- we should preserve the concept of numbered SFX slots
- we should preserve user-facing SFX percent scaling
- we may not need to preserve the exact internal playback-control fields as long as audible behavior matches

## Porting Impact

This pass gives the future SDL port a much cleaner audio boundary:

- one backend-init path from saved settings
- one graceful fallback to silent mode
- one current-music object
- one numbered SFX slot registry
- one slot-play helper
- one shared cleanup path

That is much healthier than trying to port raw DOS device families directly.

## Recommended Next Move

The strongest next move is to tighten the remaining playback-parameter uncertainties in:

- `0x6817`
- the immediate callers around menu, gameplay, and warning sounds

That should tell us whether the caller-controlled terms are best modeled in the port as:

- pan
- priority
- pitch offset
- or simply fixed playback presets per event
