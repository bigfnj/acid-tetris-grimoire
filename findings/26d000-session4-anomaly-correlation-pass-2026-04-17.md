# `0x26D000` Session-4 Anomaly Correlation Pass

Date: 2026-04-17

## Summary

This pass followed the bridge-side `0824` reproduction result:

- clean bridge isolation did not reopen `0824`
- the strongest remaining model was that the original `0824` lead belonged to a broader session-anomaly family

The narrow question here was:

- if we return to the original broader single-clone session-progression shape and extend it across a longer fresh-clone chain, do the precursor anomaly statuses and bridge-side `0824` reappear together

The lane stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I ran eight fresh-clone sessions in order with the original `selector-26d000-shape-b-env-control-single-clone` preset.

Main result:

- no precursor anomalies reappeared
- no bridge-side `0824` reappeared
- every bridge run stayed on `0868` in object `3`

So the best new closure is:

- ordinary late-session progression alone is not enough to recreate the rare bridge-side `0824` family in this sweep
- and the anomaly-link model is still the best fit, though not yet proven

## New Owned Artifacts

- [26d000-session4-anomaly-correlation-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-session4-anomaly-correlation/26d000-session4-anomaly-correlation-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T001328Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T001514Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T001701Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T001847Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T002033Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T002219Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T002406Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260418T002552Z/batch-summary.json)

Fixture root:

- [shape-b-session4-anomaly-correlation-20260418T001500Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-session4-anomaly-correlation-20260418T001500Z)

## Harness Preset

This pass reused:

- `selector-26d000-shape-b-env-control-single-clone`

No harness changes were needed.

## Results

### Session 1

- `no-inspect` -> `0868:00024831`
- `no-inspect-again` -> `0868:00024827`
- `bridge` -> `0868:0002482B`

### Session 2

- `no-inspect` -> `0868:00024828`
- `no-inspect-again` -> `0868:00024812`
- `bridge` -> `0868:00024816`

### Session 3

- `no-inspect` -> `0008:00001480`
- `no-inspect-again` -> `0868:00024824`
- `bridge` -> `0868:00024812`

### Session 4

- `no-inspect` -> `0868:00024812`
- `no-inspect-again` -> `0868:00024816`
- `bridge` -> `0868:00024821`

### Session 5

- `no-inspect` -> `0868:00024810`
- `no-inspect-again` -> `0868:00024815`
- `bridge` -> `0868:0002482E`

### Session 6

- `no-inspect` -> `0868:00024819`
- `no-inspect-again` -> `0868:00024821`
- `bridge` -> `0868:00024812`

### Session 7

- `no-inspect` -> `0868:00024824`
- `no-inspect-again` -> `0868:0002481E`
- `bridge` -> `0868:00024827`

### Session 8

- `no-inspect` -> `0868:00024815`
- `no-inspect-again` -> `0868:00024819`
- `bridge` -> `0868:00024812`

### Status Counts

- `post_vrt_rebreak_not_observed` -> `0`
- `vrt_selector_not_discovered` -> `0`
- bridge-side `0824` -> `0`
- bridge-side object-`1` -> `0`

### File Diffs

Clone tree hash diffs:

- `session-1-clone` -> `0`
- `session-2-clone` -> `0`
- `session-3-clone` -> `0`
- `session-4-clone` -> `0`
- `session-5-clone` -> `0`
- `session-6-clone` -> `0`
- `session-7-clone` -> `0`
- `session-8-clone` -> `0`

So this pass again observed no writes into the mounted game trees.

## Findings

### 1. The Precursor Anomaly Family Did Not Reappear

This is the main result of the pass.

Across all eight sessions:

- every run completed as a normal `protected_mode_breakpoint_not_hit_before_timeout`

None of the earlier precursor anomaly statuses came back:

- no `post_vrt_rebreak_not_observed`
- no `vrt_selector_not_discovered`

### 2. Bridge-Side `0824` Also Did Not Reappear

Every bridge stop stayed on:

- selector `0868`
- object `3`

So the rare bridge-side `0824` family did not return anywhere in this longer clean progression sweep.

### 3. The Only Object-`1` Leak Was An Early Opening-Slot `0008`

The only object-`1` signal in the whole pass was:

- session 3 opening `no-inspect` -> `0008:00001480`

That leak did not propagate into:

- the second `no-inspect`
- or the bridge

So the clean multi-session chain can still surface a small pre-bridge object-`1` leak, but that by itself is not enough to regenerate the earlier anomaly family or the bridge-side `0824` lead.

### 4. Clean Session Progression Can Stay Entirely Inside The Flat `0868` Family For A Long Stretch

Seven of the eight sessions stayed fully inside the flat `0868` object-`3` family from start to bridge.

This is useful closure because it shows that merely running later in the sequence is not sufficient to force the more interesting anomaly states back into existence.

### 5. The Differentiating State Still Is Not In The Clone Trees

All eight clone hash diffs stayed empty.

So this pass again rules out:

- clone-local on-disk mutation

as the explanation for the anomaly-linked behavior.

## Practical Interpretation

This pass does not prove the anomaly-link model, but it does tighten it.

We now know:

- a longer clean session chain can remain entirely ordinary
- the precursor anomaly statuses do not appear automatically with session depth
- and bridge-side `0824` does not appear when those anomalies are absent

So the next best move is to stop waiting for the anomaly family to reappear naturally and instead target it directly.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Anomaly-Seeded Replay Pass**

Focus:

- target the precursor anomaly family directly
- reuse the broader three-slot session shape
- perturb startup or prompt-handling conditions that can plausibly influence:
  - post-VRT re-break observation
  - selector discovery
- test whether bridge-side `0824` returns only when one of those anomaly conditions is reintroduced

## Bottom Line

Across eight fresh-clone sessions of the original broader progression shape, neither the precursor anomaly statuses nor the bridge-side `0824` family reappeared.

That supports the idea that `0824` belongs to the anomaly family rather than to ordinary late-session drift, but it does not prove causation because this sweep regenerated neither anomaly.
