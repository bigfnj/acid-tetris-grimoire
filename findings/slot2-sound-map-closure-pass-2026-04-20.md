# Slot 2 Sound Map Closure Pass

Date: 2026-04-20

## Summary

This pass closes the stale "unresolved" wording around `slot 2` at the sound-map level.

The evidence is now strong enough to separate two questions cleanly:

- shipped-runtime status: closed at high confidence
- original intended semantic event: still inferential, but bounded enough for port planning

Current best closure:

- `slot 2` is loaded at startup from chunk `23`
- no owned shipped-binary playback path selects `slot 2`
- the extracted sound still fits the short UI / small impact family best
- `slot 2` should remain preserved but unbound in a faithful first-pass `SoundEvent` map

## Why This Pass Was Needed

Two owned artifacts had drifted apart:

- [sound-event-mapping-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/sound-event-mapping-pass-2026-04-14.md) still carried `slot 2` as unresolved at findings level
- [sound-event-map.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/sound-event-map.json) already labeled it `unused_small_ui_or_impact_alt`

This pass resolves that mismatch so the forward-looking audio map no longer carries `slot 2` as an active open question.

## Consolidated Evidence

### Startup Registration Is Closed

Startup registration remains fixed:

- `slot 2 <- chunk 23`

That mapping is already established in:

- [sound-event-mapping-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/sound-event-mapping-pass-2026-04-14.md)

### Executable-Side Playback Coverage Is Closed

The direct playback closure remains:

- runtime SFX playback funnels through `play_audio_slot` at `0x6817`
- the full direct caller set covers slots `{0,1,3,4,5,6,7,8,9,10,11}`
- `slot 2` is the only startup-loaded shipped slot with no owned playback caller

That evidence comes from:

- [audio-caller-completion-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/audio-caller-completion-pass-2026-04-14.md)
- [slot2-resolution-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/slot2-resolution-pass-2026-04-14.md)
- [audio-slot-call-closure-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/audio-slot-call-closure-pass-2026-04-15.md)

### Asset-Side Family Fit Is Bounded

The extracted WAV for chunk `23` remains:

- very short (`0.230` s)
- compact relative to the warning / reward family
- closest to the small UI / impact neighborhood rather than to long alerts or major rewards

That evidence comes from:

- [slot2-resolution-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/slot2-resolution-pass-2026-04-14.md)

## Closure

`slot 2` no longer belongs in the unresolved active shipped sound-event map.

Closed status:

- shipped-binary role:
  - startup-loaded but unplayed auxiliary SFX
- best semantic family:
  - unused short UI / small impact alternate

Confidence split:

- high:
  - `slot 2` is not part of the active shipped gameplay/frontend sound map
- medium_high:
  - the sound itself belongs to the short UI / impact family

## Port Implication

For a faithful first-pass port:

- preserve the extracted asset and provenance
- do not bind `slot 2` to any required `SoundEvent`
- treat it as shipped-but-unused preserved content
- if surfaced later, label it as unused / cut / unknown-original-purpose rather than inventing a required event name

## Artifact Update

This pass updates the living machine-readable map:

- [sound-event-map.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/sound-event-map.json)

and adds a small closure artifact:

- [slot2-sound-map-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/slot2-sound-map-closure.json)

## What I Now Treat As Resolved

- `slot 2` should not be tracked as an unresolved shipped event mapping problem
- `slot 2` does not block a faithful first-pass `SoundEvent` enum
- the remaining uncertainty is only historical intent, not shipped behavior

## Next Ordered Step

- pause the `slot 2` branch unless one of these appears:
  - a newly recovered build that actually plays chunk `23`
  - an indirect playback path not present in the current owned executable evidence
  - external author or design evidence naming the original intended event
