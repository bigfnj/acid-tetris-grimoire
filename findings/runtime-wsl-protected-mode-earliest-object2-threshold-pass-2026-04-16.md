# Runtime WSL Protected-Mode Earliest-Object2 Threshold Pass

Date: 2026-04-16

## Summary

This pass searched specifically for an earlier object-2 landing rather than another closure no-hit.

Main result:

- the current best steering family is now bounded much more tightly
- with `AUTOTYPE -w 6 enter enter`, there is a narrow object-2 budget island around `LOGC 0x3BF000` to `0x3C2000`
- the earliest object-2 landing found in this pass was `0868:000178B1`

That is better than the older `0x178CE` to `0x1790C` late-object2 band, but it is still too late for helper-family closure:

- object `2` begins at `0x175C5`
- `0x178B1` is already `+0x2EC` into object `2`
- so it still starts after `0x175C5`, `0x17613`, `0x1765A`, and `0x17719`

## Key Artifacts

Fine-grained budget sweep around the `0x3B` / `0x3C` transition:

- [20260416T225103Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225103Z/summary.json)
- [20260416T225103Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225103Z-01/summary.json)
- [20260416T225103Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225103Z-02/summary.json)
- [20260416T225103Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225103Z-03/summary.json)

Tight follow-up around the first earlier object-2 hit:

- [20260416T225211Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225211Z/summary.json)
- [20260416T225211Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225211Z-01/summary.json)
- [20260416T225211Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225211Z-02/summary.json)
- [20260416T225211Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225211Z-03/summary.json)

Input-shape sweep over the best budget island:

- [20260416T225321Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225321Z/summary.json)
- [20260416T225321Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225321Z-01/summary.json)
- [20260416T225321Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225321Z-02/summary.json)
- [20260416T225321Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T225321Z-03/summary.json)

## Findings

### 1. There is a real object-2 threshold island near `0x3C0000`

Under `AUTOTYPE -w 6 enter enter`:

- `LOGC 0x3BC000` -> object `3` at `0868:000236C6`
- `LOGC 0x3BE000` -> object `3` at `0868:000247D3`
- `LOGC 0x3C2000` -> object `2` at `0868:000178B1`
- `LOGC 0x3C4000` -> object `3` at `0868:000247C2`

This means the object-2 transition is no longer just "somewhere around `0x3C0000`". It is a narrow branch-sensitive island.

### 2. The earliest landing found in this pass was `0868:000178B1`

The tighter follow-up showed:

- `LOGC 0x3BF000` -> object `2` at `0868:000178D8`
- `LOGC 0x3C0800` -> object `2` at `0868:000178B1`
- `LOGC 0x3C1000` -> object `3` at `0868:00023B0E`
- `LOGC 0x3C1800` -> object `3` at `0868:00023B4E`

So the floor moved earlier from the previous `0x178CE` / `0x1790C` band down to `0x178B1`, but not below it.

### 3. Input-shape changes did not break under `0x178B1`

Holding the promising budget at `LOGC 0x3C0800`:

- `AUTOTYPE -w 5 enter enter` -> object `3` at `0868:00023B44`
- `AUTOTYPE -w 7 enter enter` -> object `2` at `0868:000178CE`
- `AUTOTYPE -w 6 enter enter enter` -> object `2` at `0868:000178C6`
- `AUTOTYPE -w 6 enter` -> object `2` at `0868:00017906`

That is useful in two ways:

- the `0x3C0800` budget island is not uniquely tied to one exact input string
- but none of the nearby input-shape variations improved the landing below `0x178B1`

### 4. The helper-family interpretation is still unchanged

Even the best new landing, `0868:000178B1`, is still after:

- `0x175C5`
- `0x17613`
- `0x1765A`
- `0x17719`

So this pass improved the steering boundary, but did not yet create a closure-grade first-entry gate.

## Practical Next Step

Use this pass as a boundary map:

1. treat `0x3BF000` to `0x3C2000` as the current best earliest-object2 search lane
2. stop expecting nearby timing tweaks alone to move the landing below `0x178B1`
3. add a different steering axis on top of that lane rather than more same-family `AUTOTYPE` nudges

The important handoff fact is:

- we now have a bounded object-2 threshold island
- but its current floor is still too late for `0x175C5` / `0x17613` / `0x1765A` / `0x17719` closure
