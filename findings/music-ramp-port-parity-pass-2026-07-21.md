# Music Ramp Port Parity Pass

Date: 2026-07-21

## Summary

This pass implements and verifies the original's music-volume ramp system in the
source port, closing the "modeled, not certified" gap on music track-change and
exit-fade timing. Before this pass the port switched tracks with an instant
`Player_Load` and quit instantly on `Exit Game` (an immediate `Player_Stop`),
with no fades at all. The port now reproduces the RE-derived ramp timing exactly
and the behavior is observable per-frame in `--debug-state`.

Evidence source: [frontend-exit-and-music-ramp-pass-2026-04-14](frontend-exit-and-music-ramp-pass-2026-04-14.md)
(the ramp controller `0x699e`/`0x69b0`/`0x6965`, the track loader `0x6544`, and
the state-`3` exit path `0x3830`).

## What The Original Does (recap from the RE pass)

- Ramp controller: `0x699e` = `schedule_music_volume_ramp(mode, duration_ticks)`;
  `0x69b0` = ramp-active query (0 = idle); `0x6965` = per-tick step that clamps
  progress to the duration and scales linearly against the configured user music
  volume percent (`0x2c6eb`).
- Ramp modes: `1` = fade in (0 -> configured), `2` = fade out (configured -> 0).
- Track loader `0x6544`: fade out (mode `2`, `0x20` ticks) -> wait for ramp idle
  -> load the new track -> fade in (mode `1`, `0x20` ticks).
- Exit path (state `3`, `0x3830`): wait for ramp idle -> schedule fade out
  (mode `2`, `0x40` ticks) -> frontend fade-out -> save `SETUP.DAT` -> final DOS
  exit `0x05cc`. The fade completes before the runtime closes.

## Port Implementation

`MusicPlayer` (`port/src/music_player.{h,cpp}`) gained a ramp controller pumped
once per frame from `Update()`. One "tick" maps to one rendered frame, matching
the port's other frame-count parity gates (top-out `660`/`1260`, `hsboot`).

- `RequestTrack(index, vol)` models `0x6544`. Same track -> continuity (no fade,
  no reload). Different track with something playing -> fade out `0x20`, then on
  completion load the new module and fade in `0x20`. Nothing playing (cold
  startup) -> load immediately and fade in `0x20`.
- `BeginExitFade(vol)` schedules the `0x40`-tick mode-2 fade and stops playback
  when it completes.
- `SetVolume(vol)` (the `Music Volume` option) sets the target; applies at once
  when idle, otherwise the running ramp lands on the new target.
- Per-tick volume is the linear percent-of-configured curve (`0x6965`).
- `main.cpp` routes `music-track-change`/startup/continuity events to
  `RequestTrack`, `exit-fade` to `BeginExitFade`, and defers the actual quit
  until `exit_fade_active()` clears — mirroring the original "finish the ramp,
  then exit" ordering. Quit paths that schedule no fade (Esc, sound-setup
  `Exit to Dos`) still stop immediately.
- New `--debug-state` fields expose the ramp: `music_track`, `music_ramp`
  (`none`/`fadein`/`fadeout`), `music_ramp_left`, `music_vol`.

## Verified (Windows/MSVC, offscreen + dummy audio)

Measured from the `--debug-state` stream (captured through the Bash tool; note
PowerShell `*>` line-wraps the long stderr debug lines and splits the music
suffix, so use a raw-byte capture for this):

- Cold startup: track 0 loads as `Continuum`, then fades in over exactly `0x20`
  = 32 ticks with a clean linear curve (`music_vol` 0 -> 96, then settles at
  100 as the ramp completes).
- `Music:` row activation: current track fades out over exactly 32 ticks
  (`music_vol` 100 -> 3), then track 1 (`Tearing Up Spacetime`) loads and fades
  in over 32 ticks — the `0x6544` fade-out/load/fade-in sequence.
- `Exit Game`: exactly `0x40` = 64 fade-out ticks (`music_vol` 100 -> 1), then
  the program self-quits (`EXIT=0`, run ended at frame 424 under a 450-frame cap)
  — the fade finishes before shutdown.
- No regressions: all five documented smoke scenarios `EXIT=0`; the scripted
  top-out fingerprint (`topout=660`/`1260`, `hsboot=72`, `savedunder` flip) is
  unchanged.

## Volume Curve (certified 2026-07-21)

The ramp step `0x6965` is linear in the configured percent, and the mixer
music volume is applied by `0x68bb` as `percent * 7 / 20` — i.e. 100% music ->
mixer value **35** on MikMod's 0..128 scale (stored at `[0x19be7]`, which is
written by `0x68bb` and read only by the linked MikMod mixer, so it is the music
driver volume). The port's `ToMikModVolume` now uses this exact `* 7 / 20` curve
(previously it used `* 128 / 100`, i.e. 100% -> 128, ~3.6x too loud). Music now
sits at the original's level below full.

## Remaining Gaps

- Tick-to-realtime cadence for the ramps: the tick counts (`0x20`, `0x40`) and
  the curve are RE-exact; the port now paces at the VGA ~70 Hz refresh (see
  `frame-cadence-timer-decode-pass`), matching the original's tick base.

## Checklist Impact

Updated notes for `AUD-18` (track-change loader timing), `AUD-21` (exit fade +
deferred quit), and `OPT-08` (volume target/curve). Behavior spec audio section
now records the `0x6544` loader and state-`3` exit tick counts.
