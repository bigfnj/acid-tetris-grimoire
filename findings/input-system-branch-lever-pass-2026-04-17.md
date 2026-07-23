# Input-System Branch Lever Pass

Date: 2026-04-17

## Summary

This pass returned to the input-side steering family after the startup/config closure work and tested whether:

- release-latch menu routing
- or held gameplay-key carry into `New Game`

could still produce a materially earlier runtime seam than the current late object-2 floor.

Main result:

- input is still a real branch-sensitive family
- but under the current `LOGC 0x3C0800` lane it behaves more like an unstable bias than a dependable floor improver
- no tested input variant beat the known global floor `0868:000178B1`

## New Owned Artifact

- [input-branch-lever-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/input-branch-lever/input-branch-lever-results-2026-04-17.json)

## Probe Control

All runs reused the same current scout lane:

- `DOSBOX_X_BIN=.tools/bin/dosbox-x-linux-debug`
- `--post-vrt-log-mode logc`
- `--post-vrt-log-steps 0x3C0800`
- `--target-offset 0x17719`
- `--absolute-timeout-seconds 35`

## Tested Input Families

Control:

- `AUTOTYPE -w 6 enter enter`

Held-key carry:

- `ADDKEY p6000 l1200 down l0 p200 enter`
- `ADDKEY p6000 l1200 left l0 p200 enter`
- `ADDKEY p6000 l1200 right l0 p200 enter`
- `ADDKEY p6000 l1200 a l0 p200 enter`

Release-latch menu routing:

- `AUTOTYPE -w 6 -p 0.3 down enter down down enter`

Stability reruns:

- held `Down`
- held `Right`

## Results

### Control

- `AUTOTYPE -w 6 enter enter`
  - object `2` at `0868:000178C1`

### Held-Key Carry

- held `Down`
  - object `3` at `0868:00024847`
- held `Left`
  - object `3` at `0868:000247F0`
- held `Right`
  - object `2` at `0868:000178CE`
- held `A`
  - object `3` at `0868:000236B9`

### Menu Routing

- options -> keyboard route
  - object `3` at `0868:00023B57`

### Stability Reruns

- held `Down` rerun
  - object `3` at `0868:000247C6`
- held `Right` rerun
  - object `3` at `0868:000247E0`

## Findings

### 1. No Input Variant Beat The Known Global Floor

The known best floor from the earlier threshold pass remains:

- `0868:000178B1`

Nothing in this pass improved on that.

So even the strongest current input-side branch does not yet give us an earlier object-2 seam than the one we already had.

### 2. The Current Batch Control Landed Later Than The Known Floor

The control rerun in this pass landed at:

- `0868:000178C1`

That is still object `2`, but later than:

- known floor `0868:000178B1`

This matters because it means there is still some run-to-run drift in the current scout lane even before adding more exotic input branches.

### 3. Release-Latch Menu Routing Looks Weak Under The Current Lane

The routed menu sequence:

- `AUTOTYPE -w 6 -p 0.3 down enter down down enter`

landed at:

- object `3` `0868:00023B57`

That is worse than both:

- the current batch control
- the known global floor

So the specific release-latch route into options/keyboard is no longer a promising path toward earlier object-2 entry under the current lane.

### 4. Held-Key Carry Is Real, But Not Stable Enough To Treat As A Reliable Control

The most interesting primary-batch result was:

- held `Right`
  - object `2` at `0868:000178CE`

That told us the family is not just “held Down.”
A non-menu-overlap gameplay-bound key can also perturb the lane.

But the stability rerun for the same recipe fell back to:

- object `3` at `0868:000247E0`

And held `Down`, which had previously produced a positive object-2 result in the older external-steering pass, now regressed twice:

- `0868:00024847`
- `0868:000247C6`

So the family is real, but too unstable to call a dependable floor improver.

### 5. The Input Family Is Still Worth Keeping Conceptually, But Not As The Best Immediate Runtime Bet

This pass does **not** mean input is irrelevant.

What it does mean is:

- release-latch routing alone is weak here
- held-key carry can change the lane
- but the current recipes are not reproducible enough to anchor the next closure attempt on their own

That is a useful narrowing result.

## Practical Interpretation

The input-side branch family is now better bounded:

- menu routing into adjacent frontend states is not helping
- held gameplay-key carry is broader than just held `Down`
- but none of the tested hold shapes are stable enough to replace the plain baseline as our practical control

So input remains a secondary steering family, not the main owned control.

## Recommended Next Move

The strongest next move is now:

- **Step 7: Pre-Threshold Breakpoint Family Pass**

Reason:

- startup and config predicates are now closed
- input-side branches remain real but unstable
- the next highest-value path is to arm breakpoints closer to the last object-1 decision points before the late `0x178b0` helper cluster consumes the lane

## Bottom Line

This pass did not find a better control than the known floor.

What it did provide is a cleaner branch map:

- release-latch menu routing is weak
- held gameplay-key carry is real
- but the current held-key recipes are unstable and not reliable enough to treat as the next main closure tool
