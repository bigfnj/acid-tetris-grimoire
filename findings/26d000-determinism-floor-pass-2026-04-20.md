# `0x26D000` Determinism-Floor Pass

Date: 2026-04-20

## Summary

This pass was meant to gate further delay-ladder work before the `0x26D000` line gets any denser.

The narrow question was:

- on one unchanged mounted fixture, do repeated runs at the same post-`VRT` delay converge on one bridge-side selector family
- or is the current ladder sitting near the repeat-variance floor already

I used the existing preset:

- `selector-26d000-post-vrt-delay-refinement-single-clone`

and held everything fixed except the delay.

Method:

- one fixed clone
- `0.18` repeated `10` times
- then `0.20` repeated `10` times on that same fixture
- no other deliberate changes

Main result:

- `0.18` did not fully converge:
  - `0868` -> `8`
  - `0008` -> `2`
- `0.20` also did not fully converge:
  - `0868` -> `9`
  - `0008` -> `1`

So the best closure is:

- both delays mostly collapse to flat `0868`
- both delays can still emit rare bridge-side `0008` outliers on the same unchanged fixture
- the present delay ladder is close enough to the repeat-variance floor that further densification is not justified yet

## New Owned Artifacts

- [determinism-floor-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-determinism-floor/determinism-floor-results-2026-04-20.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T175306Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T175452Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T175638Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T175824Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T180011Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T180157Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T180343Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T180529Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T180715Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T180901Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T181048Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T181234Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T181420Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T181607Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T181753Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T181939Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T182125Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T182312Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T182458Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T182645Z/batch-summary.json)

Fixture root:

- [determinism-floor-20260420T000000Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/determinism-floor-20260420T000000Z)

## Harness Preset

This pass reused:

- `selector-26d000-post-vrt-delay-refinement-single-clone`

No tooling changes were needed inside the pass itself.

## Results

### Delay `0.18` Repeats

- `r01` -> `0868:0002B845`
- `r02` -> `0868:00025419`
- `r03` -> `0008:00001491`
- `r04` -> `0008:000014A6`
- `r05` -> `0868:0002481E`
- `r06` -> `0868:00025AA1`
- `r07` -> `0868:0002B882`
- `r08` -> `0868:0002B885`
- `r09` -> `0868:00025154`
- `r10` -> `0868:00025143`

Selector-family counts:

- `0868` -> `8`
- `0008` -> `2`

Metrics:

- modal selector -> `0868`
- convergence rate -> `0.80`
- variance rate -> `0.20`
- entropy -> `0.7219` bits

### Delay `0.20` Control Repeats

- `r01` -> `0868:00025159`
- `r02` -> `0868:0002B82A`
- `r03` -> `0868:0002B824`
- `r04` -> `0868:00025A49`
- `r05` -> `0868:0002B826`
- `r06` -> `0868:000238E4`
- `r07` -> `0868:00025163`
- `r08` -> `0868:00023937`
- `r09` -> `0008:000014A7`
- `r10` -> `0868:00024824`

Selector-family counts:

- `0868` -> `9`
- `0008` -> `1`

Metrics:

- modal selector -> `0868`
- convergence rate -> `0.90`
- variance rate -> `0.10`
- entropy -> `0.4690` bits

### Bridge Family Closure

Across all twenty repeats:

- no bridge run reached:
  - `0870`
  - `0824`
- the only bounded outlier family was:
  - `0008`
- the dominant bridge family at both tested delays was:
  - `0868`

### File Diffs

Fixture hash diff:

- `fixed-clone.hash-diff.txt` -> `0` bytes

So this pass again observed no writes into the mounted game tree.

## Findings

### 1. The Current Lead Delay `0.18` Is Not Selector-Deterministic

This is the main gate result.

On the same unchanged fixture:

- `0.18` produced `0868` eight times
- but it still produced bridge-side `0008` twice

So `0.18` did not converge on a single bridge family.

### 2. The `0.20` Control Also Fails Full Convergence

The control was slightly more stable:

- `0868` nine times
- `0008` once

But it still did not converge perfectly either.

So the repeat-variance floor is not zero even on one unchanged fixture.

### 3. The Same Two Bridge Families Appear At Both Delays

This is the practical signal/noise closure.

Both delays produced:

- dominant flat `0868`
- rare bounded `0008`

That means the presence of a single bounded hit is not enough to claim a delay-specific family.

### 4. No `0870` Family Survived This Gate

This matters because `0870` had looked like the strongest current live lead in earlier ladder work.

In this pass:

- `0870` never appeared at the bridge

So the current ladder evidence for `0870` is below the determinism floor established here.

### 5. The Delay Ladder Is Close To The Repeat-Variance Floor

The useful numeric summary is:

- `0.18` variance rate -> `0.20`
- `0.20` variance rate -> `0.10`

That is large enough that additional fine ladder points would be hard to interpret cleanly.

## Practical Interpretation

This pass did what it was supposed to do.

We now have a variance number:

- about `10-20%` bridge-family variance on repeated same-fixture runs

That means the current delay ladder is too noisy to justify finer densification right now.

Per the gating rule:

- same-delay repeats did not fully converge on one bridge family

So the ladder is still measuring meaningful noise, not just clean delay signal.

## Recommended Next Move

The outcome of this gate is:

- **step 3 is mandatory before further delay-ladder densification**

Operationally that means:

- treat isolated `0008` or `0870` bridge hits as below the current determinism floor unless reproduced
- avoid adding finer delay points until the next planned noise-control or coupling-control step is completed

## Bottom Line

The determinism floor is now measured.

On one unchanged fixture, `0.18` converged only to `8/10` `0868`, and `0.20` converged only to `9/10` `0868`, with both delays still producing rare bridge-side `0008` outliers. That means the current ladder is close to the repeat-variance floor, so further densification should stop until the next control step is done.
