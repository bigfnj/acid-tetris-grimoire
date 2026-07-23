# Runtime WSL Protected-Mode External Steering Pass

Date: 2026-04-16

## Summary

This pass moved away from debugger-side steering and tested external input-path changes over the best current threshold lane:

- deliberate menu routing via `AUTOTYPE`
- held-input carry into `New Game` via `ADDKEY`

Main result:

- both external branches can still reach PMW1 object `2`
- neither branch beat the current best floor of `0868:000178B1`

So the search is now sharper in a useful way:

- debugger-side `LOGC` / `LOGL` tuning did not improve the floor
- external path changes can affect the branch
- but the tested menu-routing and held-`Down` carry variants still stayed at or after the current late floor

## Key Artifacts

First external steering batch:

- [20260416T231154Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231154Z/summary.json)
- [20260416T231154Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231154Z-01/summary.json)
- [20260416T231154Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231154Z-02/summary.json)
- [20260416T231154Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231154Z-03/summary.json)

Held-`Down` refinement batch:

- [20260416T231304Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231304Z/summary.json)
- [20260416T231304Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231304Z-01/summary.json)
- [20260416T231304Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231304Z-02/summary.json)
- [20260416T231304Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231304Z-03/summary.json)

## Findings

### 1. Menu routing can reach object `2`, but not earlier

With `LOGC 0x3C0800`:

- `AUTOTYPE -w 6 -p 0.3 down enter` -> object `3` at `0868:000247C6`
- `AUTOTYPE -w 6 -p 0.3 down enter , down down enter` -> object `2` at `0868:0001790C`

The second sequence is consistent with deliberately routing from the main menu into options and then keyboard setup, but it still lands in the same late object-2 neighborhood rather than earlier than the best control.

### 2. Held-`Down` carry is a real branch lever

The held-input branch was grounded by the earlier cold-boot finding that a physically held direction can carry into the first gameplay frame after `New Game`.

With `LOGC 0x3C0800`:

- `ADDKEY p6000 l1200 down l0 p200 enter` -> object `2` at `0868:000178C1`
- `ADDKEY p6000 l1800 down l0 p250 enter` -> object `3` at `0868:000236BE`

So held-`Down` carry is a meaningful external steering axis, not just a theoretical one.

### 3. Nearby held-`Down` refinements did not beat the floor

Additional held-`Down` variants gave:

- `ADDKEY p6000 l900 down l0 p180 enter` -> object `2` at `0868:0001791F`
- `ADDKEY p6000 l1000 down l0 p200 enter` -> object `3` at `0868:00023B17`
- `ADDKEY p6000 l1200 down l0 p120 enter` -> object `3` at `0868:000247DD`
- `ADDKEY p6000 l1500 down l0 p220 enter` -> object `3` at `0868:000247CA`

That gives two useful constraints:

- the branch is sensitive to held-input shape
- but none of the tested held-`Down` variants beat the best single control at `0868:000178B1`

### 4. The best control remains unchanged

After this pass, the best-known earliest object-2 scout is still:

- `LOGC 0x3C0800`
- `AUTOTYPE -w 6 enter enter`
- selected live stop `0868:000178B1`

That remains too late for closure on:

- `0x175C5`
- `0x17613`
- `0x1765A`
- `0x17719`

## Practical Next Step

This pass keeps the search moving, but it also narrows what should happen next:

1. keep `LOGC 0x3C0800` plus `AUTOTYPE -w 6 enter enter` as the baseline earliest-object2 control
2. remember that external input really can affect the branch
3. stop expecting simple options-menu routing or nearby held-`Down` carry tweaks to beat the floor by themselves
4. the next external branch should be more structurally different:
   - another menu family or state route
   - another held-input family that is not just minor `Down` retiming
   - or a mixed helper strategy that deliberately changes state before the threshold lane is consumed
