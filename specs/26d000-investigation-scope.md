# `0x26D000` Investigation Scope

Date: 2026-04-20

## Question

The `0x26D000` investigation is answering whether the post-`VRT` bridge lane contains a reproducible bounded bridge-selector family distinct from flat `0868`, or whether the observed bounded hits are below the runtime determinism floor.

## Current Gate Inputs

This scope is anchored to the owned runtime closure from:

- [26d000-determinism-floor-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-determinism-floor-pass-2026-04-20.md:1)
- [determinism-floor-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-determinism-floor/determinism-floor-results-2026-04-20.json:1)

Current measured bridge-family floor on one unchanged mounted fixture:

- delay `0.18`
  - `0868` -> `8/10`
  - `0008` -> `2/10`
  - variance rate -> `0.20`
- delay `0.20`
  - `0868` -> `9/10`
  - `0008` -> `1/10`
  - variance rate -> `0.10`

No same-fixture repeat in that gate reached:

- `0870`
- `0824`

So the currently owned runtime floor is:

- roughly `10-20%` bridge-family variance on identical-input repeats

## What Counts As "Done Enough"

The runtime branch is considered done enough when one of the following labels can be assigned mechanically.

### 1. Reproducible Bounded Bridge Family Confirmed

This label is allowed only if one exact scenario:

- same preset
- same delay tuple
- same input shape

produces the same non-`0868` bridge selector on:

- `>= 8/10` runs
- across `>= 3` distinct fresh clones

and:

- the combined result is `>= 24/30`
- the same selector is the modal bridge family on each clone
- fixture trees stay hash-identical before/after the runs

Allowed bounded-family labels are currently:

- `0008`
- `0870`
- `0824`

If this threshold is met, runtime branch work may stop and the selector family can be labeled as reproducible.

### 2. Flat-`0868` Dominant Lane Confirmed

This label is allowed if every remaining tested coarse scenario keeps:

- bridge selector `0868` as the modal family at `>= 8/10`
- across `>= 3` distinct fresh clones per scenario

and no bounded family reaches the threshold in rule 1.

If this threshold is met, runtime branch work may stop and the lane can be labeled:

- flat-`0868` dominant with bounded outliers below reproducibility threshold

### 3. Runtime Ladder Exhausted Without Reproducible Bounded Family

This label is allowed if:

- the remaining permitted coarse pass is completed
- no candidate scenario meets rule 1
- and no scenario falsifies rule 2 strongly enough to justify another coarse runtime branch

If this threshold is met, runtime ladder work stops and the branch hands off to static analysis.

## Abandonment Conditions

The runtime delay-ladder branch must be abandoned immediately if any of the following becomes true.

### A. Identical-Input Variance Stays Above The Practical Floor

If identical-input repeats continue to show:

- `> 10%` bridge-family variance

then:

- no finer delay-ladder densification is allowed

The determinism-floor pass already triggered this condition.

So from this point forward:

- no more sub-step ladder widening
- no more "just one more nearby delay" passes

unless rule 1 is already within reach from a coarse scenario.

### B. Claimed Signal Does Not Clear The Floor By Enough Margin

A candidate bounded-family signal is not actionable unless it exceeds the currently owned noise floor by a meaningful margin.

Operational rule:

- if a bounded family appears only as sparse outliers
- or improves over control by less than `20` percentage points

then treat it as below the current runtime floor.

### C. One Remaining Pass Fails To Meet A Closure Threshold

This scope permits only one more runtime pass after this spec:

- **`0x26D000` Opening/Second Delay Seeding Pass**

That pass may vary:

- opening post-`VRT` delay
- second post-`VRT` delay

across the already-owned coarse values:

- `0.10`
- `0.18`
- `0.20`

while holding:

- bridge post-`VRT` delay = `0.18`

If that one remaining pass does not meet rule 1 and does not cleanly support rule 2, then:

- stop runtime ladder work on `0x26D000`
- switch to static Ghidra analysis of the caller / bridge-family seam

## Not Allowed After This Spec

The following follow-up shapes are explicitly out of scope unless rule 1 is already nearly met:

- denser delay ladders
- micro-brackets around `0.18`
- single-hit selector chasing
- new runtime passes justified only by one rare `0008` or `0870` hit
- additional "fresh clone" or "order sensitivity" variants that do not change the core coarse scenario

## Allowed One Remaining Runtime Pass

Only this one runtime pass remains authorized under the current branch:

- **`0x26D000` Opening/Second Delay Seeding Pass**

Required shape:

- reuse the per-slot delay preset
- hold bridge = `0.18`
- vary opening+second together over:
  - `0.10`
  - `0.18`
  - `0.20`
- preserve bridge-side numeric selector capture for:
  - `0824`
  - `0008`
  - `0870`
  - `0868`

If that pass succeeds, evaluate it against:

- rule 1
- rule 2

If it fails, runtime branch closes and static analysis takes over.

## Mechanical Decision Check

Every next `0x26D000` decision should now be made by this checklist.

1. Did the latest scenario produce the same bounded bridge selector on `>= 8/10` runs across `>= 3` fresh clones?
If yes: label bounded family confirmed and stop runtime branch work.

2. Did every remaining coarse scenario keep `0868` modal at `>= 8/10` across `>= 3` fresh clones, with no bounded family meeting rule 1?
If yes: label flat-`0868` dominant and stop runtime branch work.

3. Has the one remaining authorized runtime pass already been spent without satisfying 1 or 2?
If yes: stop runtime ladder work and switch to static Ghidra analysis.

4. Is a proposed next pass only densifying delays or chasing a rare one-off bounded hit?
If yes: reject it as out of scope.

## Bottom Line

The `0x26D000` branch no longer stays open by intuition.

One more coarse runtime pass is allowed. After that, the branch either:

- proves a reproducible bounded bridge family
- closes as flat-`0868` dominant with bounded outliers below threshold
- or hands off to static caller analysis

No other outcome should generate more runtime ladder passes.
