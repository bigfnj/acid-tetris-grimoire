# Persisted Audio Matrix Pass

Date: 2026-04-17

## Summary

This pass expanded the earlier one-off persisted-audio experiment into a structured 10-profile matrix over the current best threshold lane.

Main result:

- persisted audio state is definitely a real steering family
- but the strong lever is **not** "make everything as minimal as possible"
- the best single-field mover in this matrix was:
  - `mono-only`
  - landing in object `2` at `0868:000178C2`

That is still later than the global best floor `0868:000178B1`, but it materially sharpens the audio branch story.

## Tooling Added

New owned fixture-builder:

- [build_audio_matrix_runtime_fixtures.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/build_audio_matrix_runtime_fixtures.py)

Updated script index:

- [README.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/README.md)

This builder creates disposable cloned game dirs under:

- [audio-matrix](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/audio-matrix)

and writes:

- [audio-matrix.manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/audio-matrix/audio-matrix.manifest.json)

## New Owned Artifact

- [audio-matrix.probe-results.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/audio-matrix/audio-matrix.probe-results.json)

## Probe Control

All matrix runs used the same scout lane:

- `DOSBOX_X_BIN=.tools/bin/dosbox-x-linux-debug`
- `--post-vrt-log-mode logc`
- `--post-vrt-log-steps 0x3C0800`
- `--autoexec-line 'AUTOTYPE -w 6 enter enter'`
- `--target-offset 0x17719`

## Matrix Results

### Baseline

- `control-default`
  - object `3` at `0868:00024807`

### One-Field Audio Changes

- `device-none-only`
  - object `3` at `0868:000247D7`
- `rate-11025-only`
  - object `3` at `0868:000247D3`
- `mono-only`
  - object `2` at `0868:000178C2`
- `bit8-only`
  - object `3` at `0868:000247FD`
- `music0-only`
  - object `1` at `0008:00000ED0`
- `sfx0-only`
  - object `2` at `0868:00017927`
- `track5-only`
  - object `3` at `0868:000247D3`

### Combined Audio Changes

- `device-none-lowfi`
  - object `3` at `0868:00023B57`
- `device-none-min-audio`
  - object `2` at `0868:000178E2`

## Findings

### 1. Mono Is The Strongest Single Audio Lever In This Matrix

The strongest one-at-a-time audio mutation was:

- `mono-only`
  - object `2` at `0868:000178C2`

That is useful for two reasons:

- it confirms that persisted output-format state can bias the lane by itself
- it shows that the important lever is more specific than a vague "minimal audio" idea

So the audio branch is not merely about turning everything down or off.
The stereo/mono flag specifically matters.

### 2. SFX Volume Zero Also Matters, But Later

The next positive one-field result was:

- `sfx0-only`
  - object `2` at `0868:00017927`

That is later than:

- `mono-only`
  - `0868:000178C2`

So SFX volume is a real persisted lever, but weaker than the stereo flag in the current lane.

### 3. The Previous Minimum-Audio Combo Still Works, But It Is Not The Best Audio Setting

The prior combined low-audio profile still reached object `2`:

- `device-none-min-audio`
  - object `2` at `0868:000178E2`

But that is later than:

- `mono-only`
  - `0868:000178C2`

This is important because it breaks the naive reduction story.

The lesson is:

- combining more "minimal" audio settings does not necessarily improve the landing

### 4. Low-Fi Reductions Are Not Monotonic Improvements

The cleaner combined low-fi profile without zero volumes did worse:

- `device-none-lowfi`
  - object `3` at `0868:00023B57`

So the audio family is not monotonic:

- lower rate is not automatically better
- 8-bit is not automatically better
- device none is not automatically better
- combining them can erase a helpful branch bias instead of amplifying it

### 5. Music Volume Zero Is A Different Kind Of Structural Lever

The strangest single-field result was:

- `music0-only`
  - object `1` at `0008:00000ED0`

That did not merely move the landing to a different later object.
It moved the scout back into object `1`.

So music volume should now be treated as a structurally different branch lever rather than just another scalar in the same family as:

- stereo flag
- SFX volume
- output rate
- sample format

That is a useful result even though it did not improve the floor.

## Practical Interpretation

This pass narrows the persisted-audio story to something more precise:

- `stereo_enabled = false` is the strongest clean single-field mover found so far
- `sound_fx_volume_percent = 0` is a second real but weaker mover
- the old "minimum audio" bundle still works, but it is not the best audio-state control
- some combinations wash out the benefit entirely

So future audio-state work should stop treating audio settings as one undifferentiated "low audio" axis.

## Recommended Next Move

The strongest follow-up inside this family would be a small targeted second-pass matrix around:

- `mono-only`
- `sfx0-only`
- `mono + sfx0`
- `mono + track5`
- `mono + music0`

That is likely to be more valuable than another wide sweep over all low-fi combinations.

## Bottom Line

Persisted audio state is now better bounded:

- yes, it can move the threshold lane
- no, the best answer is not simply "turn everything down"
- the strongest single lever found here is the stereo/mono flag

That is a real branch-shaping result, even though it still does not beat the global floor at `0868:000178B1`.
