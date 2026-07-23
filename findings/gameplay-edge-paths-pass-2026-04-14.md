## Gameplay Edge Paths Pass - 2026-04-14

This pass stayed on the gameplay-side first-frame seam, but widened it slightly so the visible edge cases live in one place.

The three questions for this pass were:

1. what an already-active tracked transient queue can change on the first gameplay-owned frame
2. what an already-active alert tile can change on that same frame
3. what the spawn/next-piece transition can already redraw before the outer transient pass runs

### New Owned Artifacts

- `research/ghidra/exports/decompilations/gameplay-edge-pass/raw-1348-14b3.asm`
- `research/ghidra/exports/decompilations/gameplay-edge-pass/raw-0ec0-10b7.asm`
- `research/ghidra/exports/decompilations/gameplay-edge-pass/raw-206c-21b0.asm`
- `research/ghidra/exports/decompilations/gameplay-edge-pass/gameplay-edge-pass.index.json`
- `research/ghidra/gameplay-edge-paths.json`

These artifacts complement the earlier session-loop and gameplay-present-order exports rather than replacing them.

### Main Result

The first gameplay-owned present can legitimately include more staged change than our earlier shorthand suggested.

The tighter current reading is:

1. `0x17875` can restore tracked pixels from the previous frame first
2. `0x2e18` can advance or recycle persistent particle/object state
3. `0x09c8` can already redraw counters, preview state, current-piece state, and spawn/game-over edge behavior
4. `0x206c` can already restore or reveal the alert tile region
5. `0x2f24` then plots the new tracked transient overlay
6. `0x17719 -> 0x24d0` presents the result

So the gameplay-side edge model is now:

- direct gameplay block work and alert-region work first
- tracked transient overlay second
- shared flush and page-ring present last

### Active Tracked Particles

This pass did not overturn the current tracked-particle model, but it did tighten its role relative to the other edge writers.

What stays true:

- `0x17875` can visibly alter the restored gameplay snapshot before new live gameplay work begins
- `0x2e18` updates object state but does not itself draw
- `0x2f24` remains the tracked transient draw stage for the current frame

The fidelity improvement here is mainly contextual:

- tracked cleanup is now clearly only one of several first-frame delta sources
- it should not be mistaken for the whole first-frame difference on resume or spawn-heavy frames

### Active Alert Tile

The raw `0x206c .. 0x21b0` export reinforces that the alert subsystem is a real screen-writing stage before the transient overlay pass.

Direct evidence:

- `0x20ff` calls `0x17983` during reveal
- `0x210b` calls `0x2d30` immediately after that reveal work
- `0x211f` calls `0x2138` on reset or expiry
- `0x217e` calls `0x2d30` immediately after the reset-path restore

So an already-active alert tile can change the first gameplay-owned frame through:

- one more reveal step
- one restore step
- or a reset-to-clean-region step

all before `0x2f24` draws tracked transient pixels.

### Spawn And Next-Piece Transition

This is the biggest new closure from the pass.

The raw `0x1348 .. 0x14b3` export now makes the spawn helper much more concrete:

- it promotes the previous next-piece ID from `0x2c6ef` into the live current-piece slot at `0x2c71f`
- it immediately erases the old preview piece through `0x11e0`
- it chooses a new next-piece ID modulo `7`
- it immediately draws that new preview piece through `0x10ec`
- it increments the per-piece statistics counter at `0x2c6f3 + current_piece * 4`
- it redraws that per-piece counter through `0x2c58`
- it resets the live spawned piece to `X = 4`, `Y = 0`, `rotation = 0`, and `gravity accumulator = 0`

That alone means the preview area and per-piece counter can already change before the outer transient pass.

The raw `0x0ec0 .. 0x10b7` slice then tightens what happens immediately around that helper inside `0x09c8`:

- `0xff2` redraws one gameplay HUD counter through `0x2c58`
- `0x100c` redraws another gameplay HUD counter through `0x2c58`
- `0x1011` calls `0x1348`
- `0x102d` immediately checks spawn collision through `0x12ac`
- `0x1046` can trigger alert effect `6` through `0x2008` on spawn collision
- `0x1055` can stage the game-over dissolve path through `0x1f8c`
- `0x1065` can play slot `5`
- `0x10a1` draws the newly spawned live piece through `0x10ec` when the game-over path is not active

That gives us a much better first-frame edge model:

- HUD counters can already redraw
- the preview box can already erase and redraw
- the piece statistics counter can already update
- spawn collision can already trigger alert and game-over staging
- the newly spawned live piece can already draw

all before `0x2f24` adds tracked transient pixels.

### Practical Fidelity Consequence

The future port should not treat the first gameplay-owned frame as:

- restore snapshot
- maybe update alerts
- then throw particles over the top

The stronger gameplay-preservation model is:

- tracked cleanup may land first
- direct gameplay block writers may already update HUD counters, preview state, piece counters, and the live piece
- alert-region restore or reveal may also land
- tracked transient pixels layer over those results

That ordering is especially important for:

- first resumed frames
- spawn/next-piece transition frames
- first new-game frames after the staged bootstrap
- top-out frames where spawn collision immediately turns into game-over staging

### Confidence

High for:

- preview erase/redraw inside `0x1348`
- per-piece counter redraw inside `0x1348`
- spawn-collision check immediately after `0x1348`
- alert or game-over staging immediately after failed spawn
- current-piece draw before outer transient draw when game-over is not active

Still medium on:

- the exact user-facing labels of the two `0x2c58` HUD counter redraws at `0xff2` and `0x100c`

That uncertainty does not materially weaken the main fidelity result, because both are still real direct HUD block writes before the transient overlay pass.

### Next 10 Strongest Moves

1. Tighten the remaining ownership question around `0x1765a`, especially whether it is dead, indirect-only, or reached from a not-yet-exported helper.
2. Do a similar edge-focused pass on the high-score and name-entry flow so prompt, blink, row pulse, and commit sequencing are as sharp as keyboard capture now is.
3. Tighten the alert-tile lifetime edge cases further, especially expiry and immediate reset behavior across consecutive gameplay-owned frames.
4. Tighten tracked-particle edge cases where cleanup occurs but the subsequent draw set is empty or materially smaller than the prior frame.
5. Expand the owned raw artifact set around options, keyboard setup, and sound setup edge paths so the secondary frontend states are grounded like the main menu.
6. Tighten the spawn-collision and top-out presentation path one more step, especially the first visible game-over frame before frontend state `9`.
7. Revisit the first visible `state 9` bootstrap against captures using the newer frontend layering and gameplay-edge model together.
8. Keep searching for indirect or computed dirty-map writes that could weaken or refine the current mark-`3` propagation model.
9. Continue converting these low-level gameplay-edge findings into explicit implementation constraints for the future C++23/SDL3 port.
10. Consider a fresh session-log handoff once the next cluster of edge-path notes lands, since the current fidelity model has grown materially sharper since the last root-level log.
