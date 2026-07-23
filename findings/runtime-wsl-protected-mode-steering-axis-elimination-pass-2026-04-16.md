# Runtime WSL Protected-Mode Steering-Axis Elimination Pass

Date: 2026-04-16

## Summary

This pass tested two debugger-side steering axes on top of the new earliest-object2 threshold lane:

- staged `LOGC` re-break plans
- `LOGL` trace mode

Main result:

- neither axis improved the current earliest object-2 floor of `0868:000178B1`
- staged `LOGC` plans can still reach object `2`, but not earlier than the best single-hop `LOGC 0x3C0800`
- `LOGL` appears actively worse for this search lane and collapsed all tested runs back into object `3`

## Key Artifacts

Staged `LOGC` plans targeting the `0x3C0800` total-travel region:

- [20260416T230302Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T230302Z/summary.json)
- [20260416T230302Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T230302Z-01/summary.json)
- [20260416T230302Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T230302Z-02/summary.json)
- [20260416T230302Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T230302Z-03/summary.json)

`LOGL` mode checks over the same lane:

- [20260416T230413Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T230413Z/summary.json)
- [20260416T230413Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T230413Z-01/summary.json)
- [20260416T230413Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T230413Z-02/summary.json)

## Findings

### 1. Staged `LOGC` did not improve the floor

Using `AUTOTYPE -w 6 enter enter`:

- `LOGC 0x3A0000,0x20800` -> object `2` at `0868:000178BF`
- `LOGC 0x3B0000,0x10800` -> object `3` at `0868:000236B4`
- `LOGC 0x3B8000,0x8800` -> object `3` at `0868:000247FB`
- `LOGC 0x3BE000,0x2800` -> object `3` at `0868:000236BC`

This means the extra prompt transition can change the branch, but it did not produce an earlier object-2 landing than the single-hop floor:

- best prior single-hop floor: `0868:000178B1`
- best staged-plan result in this pass: `0868:000178BF`

### 2. `LOGL` is worse than `LOGC` for this lane

With the same input bias:

- `LOGL 0x3BF000` -> object `3` at `0868:000247C8`
- `LOGL 0x3C0800` -> object `3` at `0868:000247C4`
- `LOGL 0x3A0000,0x20800` -> object `3` at `0868:00023B57`

So `LOGL` should not be treated as a promising near-term way to push the object-2 floor earlier.

### 3. The current best floor remains unchanged

After this pass, the best-known earliest object-2 scout is still:

- `LOGC 0x3C0800`
- `AUTOTYPE -w 6 enter enter`
- landing at `0868:000178B1`

That remains `+0x2EC` into object `2`, which is still downstream of:

- `0x175C5`
- `0x17613`
- `0x1765A`
- `0x17719`

## Practical Next Step

This pass removes two tempting but unproductive directions:

1. do not spend more time on nearby staged-`LOGC` plan variations unless another axis is added at the same time
2. do not spend more time on `LOGL` for earliest-object2 search
3. move the next pass to a more external steering axis:
   - different shell-side helper combinations
   - different delayed interaction shape not equivalent to nearby `AUTOTYPE` tweaks
   - or a debugger-visible state discriminator that changes runtime before the threshold lane is consumed
