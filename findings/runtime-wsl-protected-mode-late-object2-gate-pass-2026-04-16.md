# Runtime WSL Protected-Mode Late-Object2 Gate Pass

Date: 2026-04-16

## Summary

This pass refined the synthetic-input steering branch under the native WSL protected-mode probe.

Main result:

- `AUTOTYPE -w 6 enter enter` is the current strongest shell-side bias toward PMW1 object `2`
- but when it succeeds, it lands late in object `2`, around `0x178CE` to `0x1790C`
- that landing zone is already downstream of `0x175C5`, `0x17613`, `0x1765A`, and `0x17719`

So this pass did improve steering, but it also clarified a limit:

- the current best gate is a late-object2 gate, not an entry-window gate
- no-hit results for the helper family from that gate are therefore "already past target" evidence, not final reachability closure

## Key Artifacts

Variant comparison:

- [20260416T223829Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223829Z/summary.json)
- [20260416T223829Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223829Z-01/summary.json)
- [20260416T223829Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223829Z-02/summary.json)
- [20260416T223829Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223829Z-03/summary.json)

Follow-up helper and target probes under the strongest variant:

- [20260416T223934Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223934Z/summary.json)
- [20260416T223934Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223934Z-01/summary.json)
- [20260416T223934Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223934Z-02/summary.json)
- [20260416T223934Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223934Z-03/summary.json)

Budget refinement around the new best input shape:

- [20260416T224142Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T224142Z/summary.json)
- [20260416T224142Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T224142Z-01/summary.json)
- [20260416T224142Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T224142Z-02/summary.json)
- [20260416T224255Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T224255Z/summary.json)
- [20260416T224255Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T224255Z-01/summary.json)
- [20260416T224255Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T224255Z-02/summary.json)

## Findings

### 1. `AUTOTYPE -w 6 enter enter` is the strongest nearby variant

With `LOGC 0x400000`:

- `AUTOTYPE -w 6 enter enter` reached object `2` at `0868:000178CE`
- `AUTOTYPE -w 5 enter` reached object `3` at `0868:00023B13`
- `AUTOTYPE -w 7 enter` reached object `3` at `0868:00024853`
- `AUTOTYPE -w 6 space` reached object `3` at `0868:00023B48`

This makes `AUTOTYPE -w 6 enter enter` the best current steering recipe, but only as a bias.

### 2. The best recipe is still branch-variable

A repeat scout with the same recipe did not stay in object `2`:

- [20260416T223934Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223934Z/summary.json)
  landed in object `3` at `0868:00023B51`

So the branch is not yet collapsed into a deterministic object-2 stop.

### 3. The object-2 landings are too late for helper-family closure

The object-2 follow-up runs landed at:

- `0868:000178CE`
- `0868:0001790C`

PMW1 object `2` begins at `0x175C5`, so those stops are already:

- `+0x309` into object `2`
- `+0x347` into object `2`

That is downstream of:

- `0x175C5`
- `0x17613`
- `0x1765A`
- `0x17719`

The follow-up no-hit runs therefore have a narrower interpretation:

- [20260416T223934Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223934Z-01/summary.json)
  armed `BP 0868:175C5` from `0868:0001790C`
- [20260416T223934Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T223934Z-02/summary.json)
  armed `BP 0868:17613` from `0868:000178CE`

Those no-hits are consistent with "the current gate already starts after those offsets" and should not be treated as first-entry closure.

### 4. The object-2 onset remains narrow and late

Budget refinement with the same input recipe showed:

- `LOGC 0x390000` -> object `3`
- `LOGC 0x3A0000` -> object `3`
- `LOGC 0x3B0000` -> object `3`
- `LOGC 0x3C0000` -> object `2`, but still at `0868:0001790C`
- `LOGC 0x3E0000` -> object `3`

So the current recipe does not reveal an earlier object-2 landing zone below `0x17800`.

## Practical Next Step

Use the current result as a constraint, not a closure:

1. stop spending passes on helper-family no-hits from the late-object2 gate
2. search specifically for an earlier object-2 stop, ideally near the `0x175C5` object base and handoff window
3. keep `AUTOTYPE -w 6 enter enter` as the best known input bias while exploring other steering axes

The important handoff fact from this pass is:

- we now know how to bias into object `2`
- but we also know that the current bias lands too late to settle `0x175C5`, `0x17613`, `0x1765A`, or `0x17719`
