# Branch-Family Batch Harness Pass

Date: 2026-04-17

## Summary

This pass turned the new selector-aware runtime model into a reusable batch harness so we can compare whole branch families in one owned run instead of stitching together one-off probe commands.

Main result:

- the selector split between the bounded `0870` cold lane and the flat `0868` frontier family is now reproducible through one command
- the first harness batch re-confirmed both families cleanly
- and it re-hit a known object-2 frontier landing at `0868:000178C2` inside the harness itself

## New Owned Artifacts

- [run_branch_family_batch.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_branch_family_batch.py)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T161451Z/batch-summary.json)

## Harness Preset

The first owned preset is:

- `selector-transition-scout`

It runs four lanes:

- cold no-autoexec lane, target `0x03DAC`, inspect selector `0870`
- cold no-autoexec lane, target `0x03DDC`, inspect selector `0870`
- default frontier lane, target `0x17719`, inspect selector `0868`
- `mono-only` frontier lane, target `0x17719`, inspect selector `0868`

Each child lane is still executed by:

- `run_dosbox_protected_mode_breakpoint_probe.py`

The harness then harvests:

- child `summary.json`
- parsed `SelectorInfo`
- parsed `LDT`
- selected live stop
- selected PMW1 object
- armed breakpoint

into one comparative batch summary.

## Results

### 1. Cold `0870` Family

- `cold_object1_conditional_clear`
  - child run `20260417T161451Z`
  - first live VRT stop `0824:0000006A`
  - selected live stop `0870:000003B9`
  - object `1`
  - armed `0870:03DAC`
  - `SelectorInfo 0870`
    - base `0x0000A3C0`
    - limit `0x0000FFFF`
    - flags `11000`
  - result: no hit before timeout
- `cold_object1_unconditional_draw`
  - child run `20260417T161526Z`
  - first live VRT stop `0824:0000006A`
  - selected live stop `0870:00000430`
  - object `1`
  - armed `0870:03DDC`
  - `SelectorInfo 0870`
    - base `0x0000A3C0`
    - limit `0x0000FFFF`
    - flags `11000`
  - result: no hit before timeout

### 2. Flat `0868` Frontier Family

- `frontier_default_enter_enter`
  - child run `20260417T161601Z`
  - first live VRT stop `0824:0000006A`
  - selected live stop `0868:000178C2`
  - object `2`
  - armed `0868:17719`
  - `SelectorInfo 0868`
    - base `0x00000000`
    - limit `0xFFFFFFFF`
    - flags `11011`
  - result: no hit before timeout
- `frontier_mono_only_enter_enter`
  - child run `20260417T161637Z`
  - first live VRT stop `0824:0000006A`
  - selected live stop `0868:00023B42`
  - object `3`
  - armed `0868:17719`
  - `SelectorInfo 0868`
    - base `0x00000000`
    - limit `0xFFFFFFFF`
    - flags `11011`
  - result: no hit before timeout

### 3. Harness-Level Grouping

The batch summary grouped the fresh runs cleanly as:

- selector `0870`
  - `cold_object1_conditional_clear`
  - `cold_object1_unconditional_draw`
- selector `0868`
  - `frontier_default_enter_enter`
  - `frontier_mono_only_enter_enter`

That is exactly the selector-family split the previous runtime-state pass predicted.

## Findings

### 1. The Selector Split Is Now Reproducible In One Owned Command

This is the main practical win from the pass.

We no longer need to manually compare:

- one early `0870` run
- then one later `0868` run
- then another rerun for confidence

The harness does that comparison directly and records the selector grouping in one artifact.

### 2. The Default Frontier Lane Re-Hit The Known `0868:000178C2` Object-2 Stop

Inside the harness batch itself, the default frontier lane landed at:

- `0868:000178C2`

with:

- object `2`
- flat selector family `0868`

That matters because it means the harness is not just structurally valid.
It can also reproduce a known useful runtime frontier inside the same comparative batch.

### 3. The `mono-only` Lane Is Still Real But Not A Stable Frontier Control

In this harness run, `mono-only` regressed to:

- `0868:00023B42`
- object `3`

So the harness also made a subtle but useful point easier to see:

- `mono-only` is still selector-family consistent with the flat `0868` frontier
- but it is not currently stable enough to serve as a dependable object-2 control

### 4. Step 10 Remains Gated

The original closure-retry step was supposed to wait until we had a materially earlier object-2 entry than:

- `0868:000178B1`

This batch did **not** beat that floor.

The best fresh object-2 stop here was:

- `0868:000178C2`

So the direct helper-family closure retry is still premature.

### 5. The Best Next Runtime Use Of The Harness Is Transition-Focused, Not Helper-Focused

Because the harness now reproduces:

- `0824` staging
- bounded `0870`
- flat `0868`

the next high-value target is the transition family between them, not another late helper retry.

## Practical Interpretation

This pass improved the workflow as much as it improved the evidence.

We now have one command that can:

- reproduce the selector-family split
- compare early and late lanes side by side
- and preserve the child-run IDs needed for deeper follow-up

That is exactly the kind of tooling the project needed before taking another serious swing at the transition seam.

## Recommended Next Move

The strongest next move is now:

- **Selector-Transition Breakpoint Refinement Pass**

Focus:

- use the new harness as the control surface
- replace the current seam targets with breakpoint families nearer the `0824 -> 0870` and `0824 -> 0868` selector emergence
- keep Step 10 helper-family closure gated until a fresh run actually beats `0868:000178B1`

## Bottom Line

The branch-family batch harness is now real, documented, and exercised.

More importantly, the first owned batch validated the selector-aware runtime model in one artifact and re-confirmed that the project is still not ready for the final helper-family closure retry.
