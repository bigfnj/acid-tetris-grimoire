# Alert Particle Coexistence Pass - 2026-04-14

This pass stayed in the gameplay-owned render seam and tightened one very specific fidelity question:

- can alert-face updates and tracked transient pixels meaningfully coexist in the same dirty-cell space?

That question matters because the port should preserve the original layering and cleanup behavior without inventing a special-case exception for the alert block.

## New Artifact

- [alert-particle-coexistence.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/alert-particle-coexistence.json)

Primary executable evidence:

- [ATET.EXE.flat-relocated.bin.00002d30.FUN_00002d30.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/support-followup-pass/ATET.EXE.flat-relocated.bin.00002d30.FUN_00002d30.c)
- [raw-2008-2294.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/alert-particle-edge-pass/raw-2008-2294.asm)
- [raw-17821-178af.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/alert-particle-edge-pass/raw-17821-178af.asm)
- [raw-2e18-3033.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-present-order-pass/raw-2e18-3033.asm)
- [raw-1edc-23ff.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/topout-pass/raw-1edc-23ff.asm)
- [raw-09c8-0ae0.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-present-order-pass/raw-09c8-0ae0.asm)

## Main Result

The best current reading is:

- initial tracked-particle emission is currently proven disjoint from the alert block
- later transient overlap is still possible
- the original frame order is designed so that such overlap remains visually coherent

So the port should **not** make the alert region special or protected.
It should preserve ordering instead.

## The Alert Block Occupies A Small Fixed Dirty-Cell Region

From the already-mapped alert subsystem plus the direct `0x2d30` rectangle helper:

- alert pixel rect:
  - `x = 20`
  - `y = 140`
  - `width = 50`
  - `height = 50`

`0x2d30` converts that into the coarse dirty-cell grid as:

- x cells `2 .. 8`
- y cells `35 .. 47`

So the alert tile owns a tight `7 x 13` block of `8x4` cells.

## The Known Tracked-Particle Emitters Start In The Board Area, Not The Alert Block

The currently mapped tracked-particle callers are:

- the line-clear helper cluster in [raw-09c8-0ae0.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-present-order-pass/raw-09c8-0ae0.asm)
- the top-out dissolve worker in [raw-1edc-23ff.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/topout-pass/raw-1edc-23ff.asm)

Their known source pixels come from the gameplay board area:

- board pixel rect:
  - `x = 109`
  - `y = 21`
  - `width = 80`
  - `height = 160`

That maps to:

- x cells `13 .. 23`
- y cells `5 .. 45`

So in the currently owned caller set, initial tracked-particle emission is horizontally disjoint from the alert block.

That is a useful concrete result:

- there is no current evidence that particles are spawned directly inside the alert region

## Later Overlap Is Still Possible

I do not think we should overstate the previous point into:

- "tracked particles can never overlap the alert block"

That would go too far.

Why:

- top-out particles use randomized polar motion
- some line-clear helpers use point-to-point motion
- current owned evidence proves source regions much better than full lifetime travel envelopes

So the accurate reading is:

- initial overlap is disproven for the currently known emitters
- later overlap remains possible

That is a good confidence boundary.

## The Frame Order Makes Overlap Safe

This is the most important part of the pass.

The gameplay-owned present boundary is already well grounded as:

1. `0x17875`
   restore prior tracked transient pixels
2. `0x2e18`
   update transient objects
3. `0x09c8`
   gameplay step
4. `0x206c`
   alert refresh/reveal/restore
5. `0x2f24`
   draw fresh tracked transient pixels
6. `0x17719`
   flush dirty cells
7. `0x24d0`
   present

That ordering gives a very strong coexistence rule:

- if a tracked particle crossed into the alert region in the prior presented frame, `0x17875` restores the saved alert-region byte first
- if the alert subsystem needs to refresh, reveal, or restore in this frame, `0x206c` writes the alert region after tracked cleanup
- only after that does `0x2f24` draw fresh tracked pixels for the current presented frame

So tracked particles can temporarily overlay the alert block for a presented frame, but they do not permanently corrupt it.

On the next outer gameplay frame, cleanup runs first again.

## Porting Consequence

The faithful port should preserve this behavior by ordering, not by special rules.

Good preservation rule:

- restore tracked transient pixels
- run alert update
- then draw fresh tracked transient pixels

Bad modernization mistake:

- forbid tracked particles from ever touching the alert region
- or redraw the alert block after the transient pass every frame

Those would change the original layering behavior.

## 10 Strongest Next Moves

1. Expand the owned raw artifact set around credits presentation so the secondary frontend family has the same direct raw coverage throughout.
2. Keep the `0x1765a` question open, but narrow it strictly to reachability confirmation.
3. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
4. Tighten whether any indirect or computed dirty-map writes refine the current mark-`3` propagation model.
5. Revisit the high-score footer-exit-to-main-menu conceal rhythm against captures now that the prompt and commit side is tighter.
6. Tighten the spawn-collision and top-out presentation path one more step, especially the very first visible failed-spawn frame.
7. Continue converting these gameplay-layering and frontend-edge findings into direct implementation constraints for the future `C++23 + SDL3` port.
8. Refresh the root session log once the next cluster lands, because the render-layering model is getting much sharper.
9. Consider a port-facing renderer contract note that groups dirty-cell propagation, tracked overwrite semantics, alert coexistence, and page-ring presentation into one implementation reference.
10. If we get more gameplay video later, a clean line-clear-heavy capture would be the most useful new evidence for tightening late particle travel behavior.
