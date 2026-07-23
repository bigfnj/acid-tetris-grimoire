# `0x26D000` Anomaly-Seeded Replay Pass

Date: 2026-04-17

## Summary

This pass followed the session-4 anomaly correlation result:

- ordinary late-session progression did not regenerate either precursor anomaly
- bridge-side `0824` did not reappear when those anomalies were absent

The narrow question here was:

- can we seed the precursor anomaly family directly with startup-stage or prompt-handling perturbations
- and if so, do any seeded conditions also move the bridge back onto an object-`1` family

The broader three-slot shape stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I tested six single-clone scenarios:

- clean control
- `pre_vrt_delay_seconds = 0.2`
- `pre_vrt_delay_seconds = 0.5`
- `post_vrt_initial_command_delay_seconds = 0.2`
- `pre_atet_exec_break_count = 0`
- `pre_atet_exec_break_count = 2`

Main result:

- the exec-break overrides seeded both precursor anomaly statuses deterministically
- but those coarse seeds never produced a useful bridge-side family because target arming never happened
- the strongest positive result came from the mild prompt-handling seed instead:
  - `post_vrt_initial_command_delay_seconds = 0.2`
  - bridge -> `0008:0000149A`

So the best new closure is:

- the anomaly family is now controllable
- but the useful bridge lever is not coarse exec-break staging
- it is the timing of the first post-`VRT` action

## New Owned Artifacts

- [26d000-anomaly-seeded-replay-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-anomaly-seeded-replay/26d000-anomaly-seeded-replay-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T004420Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T004606Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T004752Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T004938Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T005124Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T005311Z/batch-summary.json)

Fixture root:

- [anomaly-seeded-replay-20260418T003500Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/anomaly-seeded-replay-20260418T003500Z)

## Harness Preset

This pass added and exercised:

- `selector-26d000-anomaly-seeded-single-clone`

Tooling additions used by this pass:

- `run_dosbox_protected_mode_breakpoint_probe.py`
  - `--pre-vrt-delay-seconds`
  - `--post-vrt-initial-command-delay-seconds`
- `run_branch_family_batch.py`
  - now forwards those delay knobs plus `pre_atet_exec_break_count`

## Results

### Control

- `no-inspect` -> `0868:00023A33`
- `no-inspect-again` -> `0868:0002B87C`
- `bridge` -> `0868:00025486`

### Pre-VRT Delay `0.2`

- `no-inspect` -> `0868:0002B880`
- `no-inspect-again` -> `0868:0002B87F`
- `bridge` -> `0868:00023979`

### Pre-VRT Delay `0.5`

- `no-inspect` -> `0868:000252A7`
- `no-inspect-again` -> `0868:00023926`
- `bridge` -> `0868:00023A05`

### Post-VRT Initial Delay `0.2`

- `no-inspect` -> `0868:0002481E`
- `no-inspect-again` -> `0868:00023ABA`
- `bridge` -> `0008:0000149A`

### Exec-Break Count `0`

- `no-inspect` -> `post_vrt_rebreak_not_observed`
- `no-inspect-again` -> `post_vrt_rebreak_not_observed`
- `bridge` -> `post_vrt_rebreak_not_observed`

Representative seeded first-live prompt:

- `F000:0000DAC6`

### Exec-Break Count `2`

- `no-inspect` -> `vrt_selector_not_discovered`
- `no-inspect-again` -> `vrt_selector_not_discovered`
- `bridge` -> `vrt_selector_not_discovered`

Representative prompt count:

- `3`

### File Diffs

Clone tree hash diffs:

- `control-clone` -> `0`
- `pre-vrt-0p2-clone` -> `0`
- `pre-vrt-0p5-clone` -> `0`
- `post-vrt-0p2-clone` -> `0`
- `execbreak-0-clone` -> `0`
- `execbreak-2-clone` -> `0`

So this pass again observed no writes into the mounted game trees.

## Findings

### 1. The Precursor Anomaly Family Is Now Seedable On Demand

This is the biggest structural result.

Two coarse startup-stage overrides produced the anomaly statuses deterministically:

- `pre_atet_exec_break_count = 0` -> `post_vrt_rebreak_not_observed`
- `pre_atet_exec_break_count = 2` -> `vrt_selector_not_discovered`

And they did so in all three slots of the lane, not just one.

That means the anomaly family is no longer a purely spontaneous observation.

### 2. Those Coarse Seeds Are Too Early To Be Useful For Bridge Closure

This is the key limitation.

Under both exec-break override seeds:

- no target breakpoint was armed
- no selected live selector was preserved
- no bridge-side object-`1` family emerged

So the coarse anomaly seeds do recreate the statuses, but they do not yet recreate the useful bridge condition.

### 3. The Strongest Positive Result Came From A Mild Post-`VRT` Timing Shift

This is the main practical result of the pass.

With:

- `post_vrt_initial_command_delay_seconds = 0.2`

the bridge slot reached:

- `0008:0000149A`

and armed:

- `bp 0008:17719`

That is not the original bridge-side `0824`, but it is bridge-side object `1`, which is much stronger than the fully flat `0868` bridge controls.

### 4. Pre-`VRT` Delay Did Not Show The Same Leverage

Both pre-`VRT` delay variants:

- `0.2`
- `0.5`

stayed in flat `0868` object `3` at the bridge.

So timing sensitivity exists, but it is concentrated after the first live `VRT` prompt rather than before `VRT` itself.

### 5. The Best Live Lever Is Now Prompt-Handling Timing, Not Startup Break Count

Putting the scenarios together:

- coarse exec-break count changes can force the anomaly statuses
- but mild first-post-`VRT` timing changes can improve the bridge without collapsing the run into an unusable anomaly

That makes post-`VRT` timing the highest-value control axis for the next pass.

### 6. The Differentiating State Still Is Not In The Clone Trees

All six clone hash diffs stayed empty.

So this pass again rules out:

- clone-local on-disk mutation

as the explanation for the seeded behavior.

## Practical Interpretation

This pass changes the frontier in a useful way.

We now have:

- a deterministic way to recreate both precursor anomaly statuses
- proof that those coarse seeds are too blunt for useful bridge closure
- and a much more promising fine-grained lever:
  - the first post-`VRT` action timing

So the next pass should stop chasing exec-break seeds for bridge closure and instead refine the post-`VRT` delay window around the new bridge-side `0008` hit.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Post-`VRT` Delay Refinement Pass**

Focus:

- keep `pre_atet_exec_break_count = 1`
- refine `post_vrt_initial_command_delay_seconds` around `0.2`
- test a small ladder such as:
  - `0.05`
  - `0.1`
  - `0.15`
  - `0.2`
  - `0.25`
- add bridge-side numeric selector capture so any bridge family on:
  - `0008`
  - `0870`
  - `0824`
  - `0868`
  leaves stronger evidence

## Bottom Line

The anomaly family is now seedable, but the coarse exec-break seeds are too blunt to recover a useful bridge-side family.

The strongest new result came from a mild `0.2s` delay before the first post-`VRT` command, which moved the bridge to `0008:0000149A` in object `1`. That makes post-`VRT` timing the best next lever.
