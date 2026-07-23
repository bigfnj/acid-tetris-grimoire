# Slot 2 Resolution Pass

Date: 2026-04-14

## Summary

This pass stayed focused on `slot 2` until the remaining uncertainty was mostly asset-side rather than control-flow-side.

Current best resolution:

- `slot 2` is loaded at startup from chunk `23`
- no known runtime path in the shipped executable plays it
- the sound itself is a short burst that fits the small UI/impact family much better than the warning or reward family

So the most honest current label is:

- unused shipped SFX
- probably a cut or abandoned small UI / impact-style cue

## Executable-Side Resolution

The flat relocated payload now has a complete direct-caller scan for `0x6817`.

Observed facts:

- there are exactly `13` direct `play_audio_slot` callers
- they cover slots:
  - `0`
  - `1`
  - `3`
  - `4`
  - `5`
  - `6`
  - `7`
  - `8`
  - `9`
  - `10`
  - `11`
- no direct caller selects `slot 2`

The slot table at `0x2d257` is only referenced through:

- backend init clear
- startup slot loading
- slot release
- `play_audio_slot`

No additional hidden slot-table consumer was found in the flat relocated binary.

That means the control-flow evidence is now very strong:

- `slot 2` is loaded but not played in the shipped executable

## Asset-Side Characterization

Chunk `23` / slot `2` metadata:

- source chunk: `23`
- converted file:
  - [atet-dat.chunk-0023.off-001A5ADC.len-00000BFC.wav](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/audio/atet-dat.chunk-0023.off-001A5ADC.len-00000BFC.wav)
- sample rate: `13000`
- duration: `0.230` s
- mono 8-bit PCM

Useful shape features:

- very short total duration
- low zero-crossing rate compared with the more hissy/noisy warning family
- strong early attack with a fast decay
- rough autocorrelation peak around `325 Hz`

Compact normalized envelope shape:

- `0.23, 0.43, 0.49, 0.79, 0.99, 0.78, 1.00, 0.38, 0.40, 0.08, ...`

ASCII envelope sketch:

- `--#@#=-.:::.:..`

That profile does **not** look like:

- a long stack-warning cue
- a sleepy timeout cue
- a major line-clear reward
- a game-over sound

It looks much more like:

- a short UI confirmation
- a click / chirp / blip
- a small impact or movement cue

## Similarity Context

Using simple envelope and duration comparison against the rest of the shipped WAV set:

- chunk `23` is much closer to the short-sound family than to the long warning/reward family
- its nearest coarse neighbors are chunks `15`, `16`, and `18`

That is useful context because those are all relatively compact effects, not the long escalating cues.

In plain terms:

- slot `2` belongs to the "small sound" neighborhood
- it does not belong to the "major gameplay event" neighborhood

## Best Current Interpretation

Because the executable never plays slot `2`, the exact intended event cannot be proven from shipped runtime behavior alone.

But combining:

- no caller usage
- short burst shape
- similarity to compact effects rather than long alerts

the strongest current interpretation is:

- an unused shipped effect asset
- likely intended for a small UI or small gameplay feedback event

Current best label:

- `unused_small_ui_or_impact_alt`

Confidence:

- medium

## What I Would Treat As Resolved

I would now treat these points as resolved enough for the preservation spec:

- slot `2` is not part of the active shipped gameplay/frontend sound map
- it should remain extracted and preserved
- it should not be bound to a required `SoundEvent` in the faithful first-pass port
- if we expose it later, it should be marked as unused / cut / unknown-original-purpose content

## What Would Increase Confidence Further

Only a few things could move this from "best current resolution" to "near-proof":

- a hidden indirect playback path not yet found
- author testimony
- an earlier build that actually uses the sound
- an external design note naming the effect

Without one of those, the best honest conclusion is still:

- unused in the shipped game
- probably a cut short UI/impact sound
