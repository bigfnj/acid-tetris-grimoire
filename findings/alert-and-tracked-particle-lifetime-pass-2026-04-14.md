## Alert And Tracked Particle Lifetime Pass - 2026-04-14

This pass stayed on gameplay-owned first-frame and per-frame lifetime behavior.

The two targets were:

1. the alert-tile trigger, refresh, reveal, and expiry rules
2. the tracked-particle plot and restore rules, especially when the next draw set becomes empty

That turned out to be a good pairing because both systems can materially change a gameplay-owned frame right before the shared flush boundary.

### New Owned Artifacts

- `research/ghidra/exports/decompilations/alert-particle-edge-pass/raw-2008-2294.asm`
- `research/ghidra/exports/decompilations/alert-particle-edge-pass/raw-17821-178af.asm`
- `research/ghidra/exports/decompilations/alert-particle-edge-pass/alert-particle-edge-pass.index.json`
- `research/ghidra/alert-and-tracked-particle-lifetimes.json`

### Main Result

This pass produced one important refinement and one real correction.

The refinement:

- finite alerts expire cleanly by restoring the saved alert region in the same gameplay step that reaches lifetime `0`

The correction:

- `0x17821` does **not** only draw when the destination pixel is zero
- it overwrites unconditionally, queues the previous byte plus both restore pointers, and relies on `0x17875` to unwind those writes in reverse order on the next outer gameplay frame

That is a meaningful renderer-model correction for the port.

### `0x2008` Has Staged-Start And Refresh Modes

The raw trigger helper is sharper than the old “stores an effect and duration” wording.

It has two distinct behaviors.

#### New Alert Start

When the requested effect differs from the current one and the current lifetime is `0`:

- store the new effect ID at `0x2c76f`
- store the supplied lifetime at `0x2c75f`
- seed the reveal counter at `0x2c767` with `6`

That is the staged-start path.

#### Refresh Or Replace-While-Active

If the requested effect is the same as the current one, or another alert is still active:

- redraw the full current alert tile immediately through `0x1793d`
- mark the alert region dirty through `0x2d30`
- update the lifetime at `0x2c75f`
- do **not** reseed the reveal counter

That means alert refreshes do not restart the six-step reveal.
They keep the tile visible and simply refresh its live duration.

### `0x206c` Has A Stronger Expiry Rule Than Before

The update helper now reads more tightly:

1. if initialized with `EAX = 1`, it force-resets the subsystem, restores the saved region, clears the effect ID, and zeroes the warning-side cooldown fields
2. otherwise, if lifetime is `0`, it does nothing
3. if lifetime is finite and not `-1`, it decrements it first
4. if the reveal counter is still positive, it draws one reveal step and decrements that reveal counter
5. if lifetime is now `0`, it restores the saved region and clears the effect ID to `-1`

That last ordering matters.

It means a finite alert whose lifetime reaches `0` on this gameplay step can:

- still do one final reveal-step write
- and then immediately restore the clean saved region afterward

So the final visible result of the expiry step is the restored clean region, not a lingering last partial reveal.

### Infinite Alerts Are Real

`0x206c` treats lifetime `-1` specially:

- it does not decrement it
- it can still run the remaining reveal steps
- once reveal completes, the alert can remain fully visible indefinitely until a reset or replacement path changes it

That matches the game-over alert usage much better than a “everything counts down to zero” model.

### `0x17821` Is An Unconditional Overwrite Queue

This pass corrects one earlier helper claim.

The raw `0x17821 .. 0x178af` export shows:

- load previous byte from the working screen
- write the new transient byte from `CL`
- mark the dirty cell as `3`
- append three restore fields to the queue:
  - previous byte
  - pixel pointer
  - dirty-cell pointer

There is no destination-zero check here.

So tracked transient pixels are not “plot if empty” pixels.
They are queue-backed overwrite pixels.

### `0x17875` Restores In Reverse Order

The restore partner also matters more now.

It:

- walks the restore queue from the last queued entry backward
- rewrites the dirty-cell byte to `3`
- restores the saved previous pixel byte
- clears the queue count back to `0`

That reverse order is exactly what you would want if multiple tracked transient writes hit the same pixel in one rendered frame.

So the best current model is:

- `0x17821` can stack multiple transient overwrites
- `0x17875` unwinds them in reverse order on the next outer gameplay frame

That is a much stronger preservation target than the earlier zero-only interpretation.

### Cleanup-Only Frames Are Real

With the corrected `0x17821` reading plus the existing `0x2e18 -> 0x2f24` order, one useful edge case is now clear.

`0x2e18` can recycle objects because:

- age reached `0x400`
- or the updated position moved out of bounds

If it recycles every active object before `0x2f24` runs, then:

- `0x17875` still restores the previous frame’s tracked pixels first
- `0x2f24` can see an empty object list and draw nothing

So a gameplay-owned frame can legitimately be:

- cleanup only
- with no replacement tracked transient draws afterward

That is exactly the kind of small but visible behavior that matters for fidelity.

### Practical Porting Consequence

The future port should preserve these rules:

- alerts have staged start, refresh-without-reveal-reset, finite expiry-to-clean-region, and infinite-lifetime modes
- tracked transient pixels are queue-backed overwrite pixels, not zero-only decorative specks
- tracked cleanup can produce a visible delta even when the next draw set is empty
- reverse-order restoration should be preserved if the implementation still allows overlapping transient writes

### Next 10 Strongest Moves

1. Expand the owned raw artifact set around options, keyboard setup, and sound setup edge paths so the secondary frontend states are grounded like the main menu.
2. Tighten the spawn-collision and top-out presentation path one more step, especially the first visible game-over frame before frontend state `9`.
3. Revisit the first visible `state 9` bootstrap against captures using the newer frontend layering and gameplay-edge model together.
4. Keep searching for indirect or computed dirty-map writes that could weaken or refine the current mark-`3` propagation model.
5. Keep the `0x1765a` question open, but narrow it to reachability confirmation rather than broad behavior analysis.
6. Continue converting these alert and tracked-transient findings into explicit implementation constraints for the future `C++23 + SDL3` port.
7. Do a fresh evidence pass on post-game-over audio tails versus the high-score bootstrap so SFX continuity is as precise as music continuity.
8. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
9. Tighten whether any alert refresh paths can visibly coexist with tracked transient overwrites in the same 8x4 dirty cells.
10. Consider a fresh session-log handoff after the next gameplay-transition pass, because the gameplay frame-boundary model is now noticeably more precise.
