# Sound Event Mapping Pass

Date: 2026-04-14

## Summary

This pass turns the earlier audio-slot work into a usable event map for the future source port.

The key results are:

- startup loads the twelve shipped WAV chunks into a fixed `0..11` SFX slot table
- the gameplay loop copies explicit line-clear alert, sound, and score tables from flat data at `0x980`, `0x990`, and `0x9a0`
- most observed `0x6817` call sites can now be described as real gameplay or frontend events instead of anonymous slot numbers

That gives us a much better porting boundary:

- keep symbolic sound events in the modern code
- map those events to extracted assets by slot / chunk provenance
- preserve caller-controlled volume, pan, and coarse playback-class behavior

## Startup SFX Slot Registration Order

Startup uses `0x67f8` to register the twelve WAV chunks into the runtime slot table at `0x2d257`.

Observed load order:

- slot `0` <- chunk `15`
- slot `1` <- chunk `16`
- slot `2` <- chunk `23`
- slot `3` <- chunk `24`
- slot `4` <- chunk `25`
- slot `5` <- chunk `26`
- slot `6` <- chunk `19`
- slot `7` <- chunk `20`
- slot `8` <- chunk `21`
- slot `9` <- chunk `22`
- slot `10` <- chunk `17`
- slot `11` <- chunk `18`

The corresponding extracted WAVs live under:

- [audio](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/audio)

## The Gameplay Loop Carries Explicit Line-Clear Tables

At the start of `0x09c8` the executable copies three `4`-entry tables from flat data:

- `0x980` -> alert IDs by lines cleared
- `0x990` -> SFX slots by lines cleared
- `0x9a0` -> score values by lines cleared

Recovered values:

- alert IDs: `[2, 0, 9, 7]`
- SFX slots: `[9, 7, 7, 10]`
- scores: `[100, 300, 600, 1200]`

This means the generic line-clear sound mapping is now direct:

- `1` line -> slot `9`
- `2` lines -> slot `7`
- `3` lines -> slot `7`
- `4` lines -> slot `10`

Special overrides in the same path:

- if the current alert tile is ID `11` and the player clears `1` line, the game triggers alert ID `12` and plays slot `8`
- if the previous clear count was `4` and the current clear count is also `4`, the game triggers alert ID `8` and plays slot `11`

That lines up cleanly with the earlier alert-face work:

- alert ID `11` is the sleepy idle state
- alert ID `12` is the wake-up or recovery state
- alert ID `8` is the special repeated four-line reward state

## Current Best Sound Event Map

### Slot `0`

- chunk `15`
- current best role: piece lock / board impact
- evidence:
  - `0x0d4e` fires immediately after downward collision resolution succeeds
  - caller pan is derived from piece position around `0x80`
  - caller class offset is the base `0` path
- confidence:
  - medium_high

### Slot `1`

- chunk `16`
- current best role: frontend row-move navigation sound
- evidence:
  - used by main menu row changes in `0x40c0`
  - used by options row changes in `0x4584`
  - caller pan is centered at `0x80`
- confidence:
  - high

### Slot `2`

- chunk `23`
- current best role: unresolved
- evidence:
  - startup loads the asset into the fixed slot table
  - no direct `0x6817` caller has been identified yet in the current export set
- confidence:
  - low

### Slot `3`

- chunk `24`
- current best role: low stack warning sound
- evidence:
  - `0x22e3` plays slot `3` in the lowest danger band of `0x21c4`
  - paired alert effect is ID `3`
- confidence:
  - medium_high

### Slot `4`

- chunk `25`
- current best role: medium/high stack warning sound
- evidence:
  - `0x2244` and `0x2294` both play slot `4`
  - paired alert effects are IDs `5` and `4`
- confidence:
  - medium_high

### Slot `5`

- chunk `26`
- current best role: top-out / game-over sound
- evidence:
  - `0x1065` plays slot `5` immediately after the spawn-collision failure path
  - the same path also triggers alert ID `6`
- confidence:
  - high

### Slot `6`

- chunk `19`
- current best role: sleepy no-clear timeout sound
- evidence:
  - `0x10dc` plays slot `6` when the idle timer reaches `0x708`
  - the same path triggers alert ID `11`
- confidence:
  - high

### Slot `7`

- chunk `20`
- current best role: generic two-line / three-line clear sound
- evidence:
  - the generic line-clear SFX table at `0x990` maps both `2`-line and `3`-line clears to slot `7`
- confidence:
  - high

### Slot `8`

- chunk `21`
- current best role: wake-up clear reward from sleepy state
- evidence:
  - when alert ID `11` is active and the player clears `1` line, `0x09c8` triggers alert ID `12` and plays slot `8`
- confidence:
  - high

### Slot `9`

- chunk `22`
- current best role: generic single-line clear sound
- evidence:
  - the generic line-clear SFX table at `0x990` maps `1` line to slot `9`
- confidence:
  - high

### Slot `10`

- chunk `17`
- current best role: generic four-line clear sound
- evidence:
  - the generic line-clear SFX table at `0x990` maps `4` lines to slot `10`
- confidence:
  - high

### Slot `11`

- chunk `18`
- current best role: repeated four-line clear reward sound
- evidence:
  - if the previous clear count was `4` and the current clear count is also `4`, `0x09c8` overrides the generic path and plays slot `11`
  - the paired alert effect is ID `8`
- confidence:
  - high

## Porting Implications

The future SDL audio layer should preserve:

- fixed symbolic sound events rather than raw chunk numbers in gameplay code
- per-play SFX volume from the saved config percent
- centered versus gameplay-derived pan
- at least two coarse playback routing classes, since caller `EDX` still looks like a small mixer voice or class offset

The future port does **not** need to preserve:

- literal DOS backend families
- IRQ / PIT details
- exact lower mixer object layout

## New Machine-Readable Artifact

This pass also adds:

- [sound-event-map.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/sound-event-map.json)

That file is intended to be the forward-looking bridge between the decompilation notes and the eventual `SoundEvent` enum in the C++23 port.
