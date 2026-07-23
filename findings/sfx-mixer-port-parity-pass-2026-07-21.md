# SFX Mixer Port Parity Pass

Date: 2026-07-21

## Summary

This pass implements the original SFX mixer entry point `0x6817` as a real
parameter model in the source port, and wires the previously-missing slot-`0`
piece-lock sound. Before this pass the port played every SFX with a flat
`percent/100` SDL stream gain, centered, with no notion of the engine volume
range, per-play pan, or playback class.

Evidence sources:
[audio-playback-parameter-pass-2026-04-14](audio-playback-parameter-pass-2026-04-14.md)
(the `0x6817` parameter model) and
[sound-event-mapping-pass-2026-04-14](sound-event-mapping-pass-2026-04-14.md)
(the slot -> role / pan / class table).

## What The Original Does (recap)

`0x6817` is the single SFX playback call. Its caller controls:

- `EAX` slot index (0..11)
- `EDX` coarse voice/playback-class offset: `0` for UI/menu and the gameplay
  piece-lock, `1` for stack warnings
- `ECX` pan: `0x80` centered, offsets bias left/right
- `EBX` saved user-facing SFX percent, which `0x6817` scales into the engine
  mixer range `0..0x40` before mixing

Known call-site values: menu navigation (slot `1`) centered/class `0`; warnings
(slots `3`/`4`) centered/class `1`; piece-lock / board impact (slot `0`, caller
`0x0d4e`) class `0` with pan derived from the piece's board column around `0x80`.

## Port Implementation

- `AudioProbe` (`port/src/audio_probe.{h,cpp}`): clips are decoded once to
  normalized mono float (`SDL_ConvertAudioSamples`); the output stream is F32
  stereo. `PlaySfxSlot(slot, percent, pan)` scales the percent through
  `EngineVolume()` (`percent * 0x40 / 100`), derives left/right gains from pan
  (balance law: `0x80` keeps both channels at full so centered SFX are
  unchanged), and bakes gain+pan into an interleaved stereo buffer. This renders
  pan as real left/right level, since SDL3 exposes only a scalar stream gain and
  no pan control.
- `MilestoneADemo`: the audio event now carries `pan` (default `0x80`) and
  `voice_class`. `EmitAudioEvent(name, slot, pan, voice_class)` sets them.
  Warnings emit class `1`; the new `piece-lock` event (slot `0`) is emitted at
  `CommitLivePieceToBoard` with `PanForPieceColumn(live_piece_.x)` (a modeled
  column-to-pan curve around `0x80`, since the exact original curve is not
  recovered) and class `0`.
- `main.cpp`: passes `event.pan` into `PlaySfxSlot` and logs the realized mixer
  parameters per play: `SFX mix: slot=S vol%=P engine=E/64 pan=N class=C`.

## Verified (Windows/MSVC, offscreen + dummy audio; via the Bash tool)

- Volume curve: 90% -> `engine=57/64`, 100% -> `engine=64/64` — the exact
  `percent * 0x40 / 100` scaling, not a flat percent gain.
- Piece-lock (slot `0`) now fires at lock, class `0`, with pan tracking the
  column: piece at center column -> pan `118`; pushed to the right (column 7) ->
  pan `181` (matches `PanForPieceColumn`). Confirms the mono->stereo pan render
  varies with board position.
- Menu row move (slot `1`): pan `128` centered, class `0`.
- Line-clear (slot `10`): pan `128` centered, class `0`.
- Stack warnings (slots `3`/`4`): a tall left-wall pile drove the stack into the
  warning bands, and both fired centered (pan `128`) through voice class `1`
  (`SFX mix: slot=3 ... class=1`, `slot=4 ... class=1`), then the pile topped out.
- No regressions: all five documented smoke scenarios `EXIT=0`; the music-ramp
  and top-out fingerprints from the prior pass are unchanged.

## Notes / Remaining Gaps

- The board-column-to-pan curve for slot `0` is now recovered exactly from the
  piece-lock caller `0x0d4e`: `pan = (pieceCol - 5) * 21 + 0x80` (centered at
  column 5). The port uses this (verified: column 4 -> 107, column 7 -> 170).
  The lock sound also plays at half the stored SFX volume: its `0x6817` volume
  arg is `[0x2c6e7] / 2` (`[0x2c6e7]` is confirmed the SFX volume — every other
  SFX caller loads it full as `EBX`). The port now halves the piece-lock volume
  too (verified: piece-lock logs `vol%=45` when other SFX log `vol%=90`).
- Voice class (`EDX`) is carried and logged but not used to alter playback: in a
  modern per-event stream mixer the coarse DOS voice/class offset has no audible
  consequence, and the RE notes explicitly say the port need not reproduce the
  DOS mixer/vtable layer. It is preserved as data for fidelity/observability.
- Warnings require a tall non-clearing stack (highest row `<= 8` for band 1).
  Because the smoke loop runs in real time (16 ms/frame), observing them needs a
  long run (~6500 frames of piling every piece into the left wall); that run is
  slow enough to background. See the handoff for the exact command.

## Checklist Impact

Updated `OPT-09` (engine volume curve), `AUD-22` (the `0x6817` model + slot-`0`
piece-lock + pan render), `AUD-23` (menu center/class 0), `AUD-24`/`AUD-25`
(warning class 1). Behavior spec audio section now records the `0x6817`
parameter model and the known per-slot pan/class values.
