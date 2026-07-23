# Runtime-State Snapshot Pass

Date: 2026-04-17

## Summary

This pass used the new stable pre-threshold object-1 live-stop window to capture runtime selector state before the breakpoint arm, then compared that state against later `0868`-based runtime lanes.

Main result:

- the early object-1 window is not just "earlier on the same path"
- it is running under a different selector family than the later flat `0868` frontier
- which changes how we should interpret the Step 7 no-hit result

## New Owned Artifact

- [runtime-state-snapshot-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/runtime-state-snapshot/runtime-state-snapshot-results-2026-04-17.json)

## Probe Control

This pass reused the existing protected-mode probe and queued debugger inspection commands before the target breakpoint arm:

- `selinfo <live selector>`
- `ldt`

The resulting selector metadata came from the owned `dosbox.log` files for each run.

## Snapshot Set

### 1. Stable Object-1 Pre-Threshold Window

Run:

- `20260417T160411Z`

Control:

- `Original.Game`
- no autoexec steering
- `LOGC 0x200000`

Selected live stop:

- `0870:00000433`
- object `1`
- relative offset `0x433`

Captured selector state:

- `SelectorInfo 0870`
  - base `0x0000A3C0`
  - limit `0x0000FFFF`
  - type `0x1A`
  - flags `11000`
- `LDT Base:00000000 Limit:00000000`

### 2. Later `0868` Comparison Lane, Default Control

Run:

- `20260417T160524Z`

Control:

- `Original.Game`
- `AUTOTYPE -w 6 enter enter`
- `LOGC 0x3C0800`

Selected live stop:

- `0868:00023B1B`
- object `3`
- relative offset `0xC0F8`

Captured selector state:

- `SelectorInfo 0868`
  - base `0x00000000`
  - limit `0xFFFFFFFF`
  - type `0x1A`
  - flags `11011`
- `LDT Base:00000000 Limit:00000000`

### 3. Later `0868` Comparison Lane, `mono-only` Fixture

Run:

- `20260417T160636Z`

Control:

- `Decompilation.Effort/research/runtime-fixtures/audio-matrix/mono-only`
- `AUTOTYPE -w 6 enter enter`
- `LOGC 0x3C0800`

Selected live stop:

- `0868:00023B54`
- object `3`
- relative offset `0xC131`

Captured selector state:

- `SelectorInfo 0868`
  - base `0x00000000`
  - limit `0xFFFFFFFF`
  - type `0x1A`
  - flags `11011`
- `LDT Base:00000000 Limit:00000000`

### 4. Historical Confirmed Object-2 Reference

Existing owned run:

- `20260417T143252Z`

Confirmed selected live stop:

- `0868:000178C2`
- object `2`
- relative offset `0x2FD`

This run did not capture `SelectorInfo`, but it matters because it ties a confirmed object-2 landing directly to selector `0868`.

## Findings

### 1. `0824:0000006A` Is Still The Stable Pre-Transition Re-Entry Point

Across the new snapshot runs and the historical object-2 reference, the first live VRT prompt remained:

- `0824:0000006A`

So the runtime still appears to pass through the same early protected-mode staging point before fanning out into later selector families.

### 2. The Stable Early Window On `0870` Is A Bounded Non-Flat Descriptor

The most important new fact in this pass is:

- selector `0870` is not flat

Its descriptor shows:

- base `0x0000A3C0`
- limit `0x0000FFFF`

That means the early pre-threshold window is running inside a bounded relocated segment, not the flat zero-based code view we were implicitly approximating from the debugger offset alone.

### 3. The Later `0868` Frontier Family Is Flat

Both fresh comparison runs showed the same selector state for `0868`:

- base `0x00000000`
- limit `0xFFFFFFFF`

And the historical confirmed object-2 landing also uses selector `0868`.

So it is reasonable to infer that the known object-2 frontier belongs to the same flat `0868` selector family.

This is an inference from owned evidence, not a directly captured descriptor on the historical run itself.

### 4. Step 7’s No-Hit Result Needs To Be Reinterpreted

Before this pass, the Step 7 no-hit result looked like:

- maybe the object-1 window is just too early
- or maybe the cold lane is the wrong branch

After this pass, there is an additional and stronger nuance:

- the early `0870` window is not the same selector family as the flat `0868` frontier

So arming:

- `0870:03DAC`
- `0870:03DDC`

may be semantically mismatched with the flat frontend seam we actually care about.

That does not make the Step 7 pass wrong.
It makes it more precisely interpretable.

### 5. The Next Runtime Move Should Be Selector-Aware

The main takeaway is no longer just "find an earlier offset."

It is:

- identify where execution transitions from the stable `0824` staging point into either the bounded `0870` family or the flat `0868` family
- then batch breakpoint families around that transition rather than treating all displayed offsets as equivalent flat addresses

## Practical Interpretation

This pass tightened the runtime model materially.

We now know:

- `0824` is the stable early re-entry point
- `0870` is a bounded non-flat live window
- `0868` is the flat selector family associated with the current frontier

That means the project should stop treating the Step 7 object-1 window as a simple earlier version of the flat frontend seam.

## Recommended Next Move

The strongest next move is now:

- **Step 9: Branch-Family Batch Harness Pass**

But it should be refocused around:

- selector-aware breakpoint batches
- especially the `0824 -> 0870` and `0824 -> 0868` transition family

## Bottom Line

This pass showed that the stable pre-threshold window is real, but it is living in a different selector context from the known `0868` object-2 frontier.

That turns the previous early-arm no-hit result from a vague timing failure into a much sharper runtime-model insight: the next closure attempt needs to track selector transition, not just nearby flat offsets.
