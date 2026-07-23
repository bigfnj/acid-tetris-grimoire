# Persisted Audio Focus Pass

Date: 2026-04-17

## Summary

This pass ran the recommended second-pass persisted-audio mini-matrix around the strongest April 17 full-matrix levers:

- `mono-only`
- `sfx0-only`
- `mono + sfx0`
- `mono + track5`
- `mono + music0`

Main result:

- no focused combination beat the current global floor `0868:000178B1`
- `mono-only` remained the best focused profile
- but it only landed at `0868:000178C9`
- the new combinations mostly taught us which audio levers cancel each other out rather than revealing an earlier object-2 seam

## Reused Inputs

Fixture set:

- [audio-matrix-focus](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/audio-matrix-focus)
- [audio-matrix-focus.manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/audio-matrix-focus/audio-matrix-focus.manifest.json)

New owned result artifact:

- [audio-matrix-focus.probe-results.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/audio-matrix-focus/audio-matrix-focus.probe-results.json)

Fresh runtime summaries used by this pass:

- [20260417T145707Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260417T145707Z/summary.json)
- [20260417T145742Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260417T145742Z/summary.json)
- [20260417T145817Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260417T145817Z/summary.json)
- [20260417T145853Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260417T145853Z/summary.json)
- [20260417T145928Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260417T145928Z/summary.json)

## Probe Control

All focused runs reused the same native WSL protected-mode lane:

- `DOSBOX_X_BIN=.tools/bin/dosbox-x-linux-debug`
- `--post-vrt-log-mode logc`
- `--post-vrt-log-steps 0x3C0800`
- `--autoexec-line 'AUTOTYPE -w 6 enter enter'`
- `--target-offset 0x17719`

## Focused Results

- `mono-only`
  - object `2` at `0868:000178C9`
- `sfx0-only`
  - object `2` at `0868:00017927`
- `mono-sfx0`
  - object `3` at `0868:000247C8`
- `mono-track5`
  - object `2` at `0868:000178CE`
- `mono-music0`
  - object `3` at `0868:000247D0`

## Findings

### 1. No Focused Combo Improved The Global Earliest Object-2 Floor

The best focused landing was:

- `mono-only`
  - `0868:000178C9`

That is still later than:

- global floor
  - `0868:000178B1`

So this pass did not produce the earlier object-2 handoff we need for helper-family closure.

### 2. Mono Remains The Strongest Audio Lever, But It Looks Like A Stable Late-Helper Cluster Rather Than A New Seam

The broader matrix had already shown:

- `mono-only`
  - `0868:000178C2`

This rerun landed at:

- `mono-only`
  - `0868:000178C9`

That is only a small shift inside the same late `0x178b0` glyph-helper cluster.

So the focused pass strengthens the interpretation that mono is a real lane bias, but not one that escapes the late frontend text-render phase.

### 3. SFX Volume Zero Is Stable But Not Stronger

The rerun for:

- `sfx0-only`
  - `0868:00017927`

matched the earlier full-matrix result exactly.

That is useful because it shows the SFX lever is reproducible, but it is still weaker than the mono lane and does not point to a better threshold family.

### 4. Positive Single-Field Levers Are Not Additive

The strongest negative result in this pass was:

- `mono-sfx0`
  - object `3` at `0868:000247C8`

That matters because both component levers had previously produced object-2 landings on their own.

So this pass closes an important branch question:

- mono bias and SFX-zero bias do not stack
- combining them destroys the helpful late object-2 lane instead of improving it

### 5. Track Index Does Not Improve The Mono Lane

The `mono-track5` combination stayed in object `2`:

- `0868:000178CE`

But that is later than:

- `mono-only`
  - `0868:000178C9`

So track index is not a productive follow-up axis inside the mono branch.

### 6. Mono Suppresses The Earlier Music-Zero Structural Peel

Earlier, the broader matrix found:

- `music0-only`
  - object `1` at `0008:00000ED0`

The focused combination:

- `mono-music0`
  - object `3` at `0868:000247D0`

did not preserve that peel-back into object `1`.

So the music-zero structural branch is not simply composable with mono; mono appears to suppress it and steer the scout back into the later object-3 family.

## Practical Interpretation

This pass narrows the persisted-audio story again:

- `mono-only` is still the best audio-state mover in the current lane
- `sfx0-only` is reproducible but weaker
- `track5` does not improve mono
- `mono + sfx0` is actively bad
- `mono + music0` cancels the earlier music-zero structural diversion instead of strengthening it

That means the persisted-audio family is now much better bounded.
It is still a real steering family, but the interesting part appears to be isolated single-field biases rather than additive combinations.

## Recommended Next Move

The strongest next move is now to leave the audio family and return to:

- **Step 4: Config Schema Closure Pass**

Specifically:

- tighten which `SETUP.DAT` fields are startup/control-flow meaningful
- separate true branch levers from cosmetic or late-only state
- use that closure to decide whether any remaining config-family mutations are structurally different enough to justify more runtime probing

## Bottom Line

The focused mini-matrix did useful closure work, even without improving the floor:

- no new earlier object-2 landing
- mono still best
- positive single-field audio levers are not additive
- the music-zero branch is conditional and can be suppressed by mono

That is enough to de-prioritize more audio-combination fishing and move on to config-schema closure.
