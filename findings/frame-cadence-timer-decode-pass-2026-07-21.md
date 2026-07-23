# Frame Cadence / Timer Decode Pass

Date: 2026-07-21

## Summary

This pass decodes the frame-pacing and timer architecture (`0x24d0`, the PIT
setup helpers, and the tick counter `0x2d2a3`) from the flat-relocated binary,
to address the open "tick-to-realtime cadence" parity item (shared by
`LEVEL-05` gravity and the music ramps). The runtime DOSBox debugger probes are
Unix-only, so this uses the same static capstone method as the line-clear decode.

Key conclusion: the game's tick is driven by a **variable-rate, event-scheduled
PIT timer**, not a fixed game-Hz, so a single static real-time cadence constant
does not exist. Certifying the cadence for a specific configuration still
requires a runtime capture; this pass recovers the architecture and bounds the
remaining question.

## Frame Pacer `0x24d0`

Called once per rendered frame; returns the number of gameplay steps to run.

1. Mode `AL==1`: latch the current tick as the frame baseline
   (`[0x2ca93] = [0x2d2a3]`) and return 0.
2. Otherwise:
   - page-flip by writing the VGA CRTC start-address-high register
     (`out 0x3d4, ax` with register `0x0c`) from the rotated page pointers
     (`[0x2c6ab]`/`[0x2c6bb]`/`[0x2c6bf]`) — hardware double/triple buffering.
   - wait for the tick counter `[0x2d2a3]` to advance at least once (spin).
   - then spin until at least `[0x2c607]` ticks have elapsed since the baseline
     (the per-frame tick budget).
   - return `elapsed_ticks` (updating the baseline), clamped to a **maximum of
     6**. This is the fixed-step catch-up count consumed by the session loop
     (`0x23b`), so gameplay advances 1..6 logic steps per rendered frame.

So one gameplay step == one timer tick, and the renderer decouples via the
`[0x2c607]`-tick budget with a 6-step catch-up cap.

## Tick Counter `0x2d2a3`

`0x65b3` (`inc dword ptr [0x2d2a3]; ret`) is the tick increment, in the
sound/timer ISR region. The counter is therefore incremented by the installed
timer interrupt. This is the **same global used to seed the gameplay RNG**
(`0x32b0` reads `[0x2d2a3]` into `srand`), which independently confirms the
conclusion of
[rng-piece-selection-port-parity-pass-2026-07-21](rng-piece-selection-port-parity-pass-2026-07-21.md):
the RNG seed is a free-running timer value, hence non-deterministic.

## PIT Programming

- `0x69b6` is a generic "set PIT channel-0 divisor" helper: `out 0x43, 0x30`
  (channel 0, lobyte/hibyte, mode 0), then the 16-bit divisor from `EAX`
  (`1193182 / divisor` Hz).
- Its callers (`0x6b13`, `0x6bd7`, `0x6d4a`) pass divisors from `[0x2d313]`
  (the base divisor) / `[0x2d323]` (a countdown); the ISR reprograms the PIT to
  interleave the base game-tick rate with sound-mixing sub-callbacks. Both
  globals are BSS (baked 0), set at init.
- A separate path at `0x6c2b` programs channel 0 to mode-2 with divisor 0
  (65536 -> the stock 18.2065 Hz), consistent with restoring the BIOS timer
  (e.g. on shutdown) or chaining the original handler.

## The Base Rate Is The VGA Vertical Refresh (~70 Hz)

The init at `0x6d0b` sets `[0x2d313]` from `call 0x6c62`, then programs the PIT
to it via `0x69b6`. `0x6c62` is a **VGA vertical-retrace calibration**:

1. poll VGA Input Status 1 (port `0x3da`) bit 3 (vertical retrace) to sync to a
   retrace edge,
2. latch the PIT counter (`0x69d6`),
3. wait one full refresh (next retrace edge),
4. read the elapsed PIT count (`0x6a1d`).

So the base divisor is the number of 1193182 Hz PIT clocks in one VGA vertical
refresh, i.e. the timer is programmed to fire **once per video frame**. ATET runs
in a 320-wide 256-color VGA mode (mode 13h; stride 0x140, well origin
`0x1a40 = 21*320`), whose refresh is **70.086 Hz**. The frame pacer (`0x24d0`)
consumes one tick per rendered frame (6-step catch-up), so the whole game — draw,
gravity, input repeat, music/SFX ramps — advances at the VGA refresh, ~70 Hz.

## Cadence Conclusion

The real-time cadence *is* knowable and fixed: the game runs at the VGA vertical
refresh rate of its 320x200 mode, **70.086 Hz** (14.27 ms/frame), not a
sound-config-coupled rate. (The sound system's own mixing callbacks are
interleaved as PIT sub-events, but the *game/frame* tick is the refresh rate.)

Port implication:

- the port previously paced at ~60 fps (`SDL_Delay(16)`), which is ~14% too slow;
  it should target ~70 fps to match the original's real-time speed. Implemented:
  the frame delay now targets the VGA rate (~14 ms). Frame-count fingerprints are
  unaffected (they count iterations, not wall-clock time).
- the faithful invariants preserved: gameplay advances in discrete ticks; the
  renderer runs one tick per frame with a 6-step catch-up cap; gravity/repeat/ramp
  counters are in ticks.

## Method Note

Decoded statically with capstone against
`research/disassembly/pmodew/extracted/flat/ATET.EXE.flat-relocated.bin`
(address == file offset); no Ghidra or DOSBox run required.
