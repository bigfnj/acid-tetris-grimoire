## Top-Out And Game-Over Presentation Pass - 2026-04-14

This pass stayed on one narrow but important gameplay-transition seam:

- what the player actually sees after a failed spawn, before the later high-score handoff ever becomes relevant

### New Owned Artifacts

- `research/ghidra/exports/decompilations/topout-pass/raw-1edc-23ff.asm`
- `research/ghidra/exports/decompilations/topout-pass/topout-pass.index.json`
- `research/ghidra/topout-gameover-presentation.json`

### Main Result

Top-out is now much cleaner as a staged presentation chain.

The original does **not** do this:

- failed spawn
- show `GAME OVER`
- jump into high scores

The stronger current reading is:

1. failed spawn immediately triggers alert face `6` and slot `5`
2. game-over dissolve state is armed
3. the gameplay image then dissolves row by row through `0x1edc`
4. only after that dissolve completes does the game save the covered region and draw the chunk-8 `GAME OVER` overlay
5. only after the finished-game-over handoff does `0x23ac` restore the underlay and enter frontend state `9`

That is a very good fidelity win for the eventual port.

### Failed Spawn Does Not Jump Directly To The Overlay

We already knew the spawn-failure path in `0x09c8` could:

- trigger alert effect `6`
- stage the game-over helper through `0x1f8c`
- play slot `5`

This pass adds the missing middle visual sequence.

The raw `0x1f8c` helper shows:

- `EAX = 2` clears the logical board block at `0x2c23b`
- resets `0x184db` to `0`

Then the normal update path of `0x1f8c`:

- calls `0x1edc` for row `0x184db + 0x15`
- increments `0x184db`
- continues until `0x184db == 0xa0`

So after top-out, the player first gets a staged gameplay-area dissolve, not an immediate overlay replacement.

### `0x1edc` Is The Visible Dissolve Worker

The raw row worker makes that dissolve feel much more concrete.

For each row it:

- scans every other pixel across the gameplay row
- skips zero pixels
- samples the source color byte directly from the current working screen
- queues polar particles through `0x2f78`
- clears the source row bytes afterward through `0xe6b0`
- marks that gameplay row dirty through `0x2d30`

That means the dissolve is a real visual teardown of the live gameplay image:

- source colors become transient debris
- the row itself is cleared afterward
- the next presented frame shows the progressively erased playfield

### The Overlay Is A Late Separate Phase

Only when the row walk completes does `0x1f8c`:

- write `-2` into `0x184db`
- clear the live-game flag at `0x2c72b`
- call the helper currently mapped as `show_game_over_overlay`

The raw `0x22fc .. 0x23a8` helper then does two separate things:

1. copy the background under the overlay into the save buffer at `0x2c603`
2. blit the non-zero chunk-8 overlay bytes back onto the gameplay image

Then it marks that overlay region dirty.

So the `GAME OVER` overlay is explicitly a late saved-under overlay phase, not part of the initial failed-spawn reaction.

### State `9` Begins Only After The Overlay Is Removed

This pass also reinforces the later handoff chain:

- finished game-over leaves `0x184db == -2`
- the session-loop released-`Esc` handoff sees that state
- `0x23ac` restores the saved-under region from `0x2c603`
- only then does frontend state `9` begin

So the visible sequence is:

- top-out alert and SFX
- staged row dissolve
- `GAME OVER` overlay
- overlay removed
- shared frontend bootstrap
- high-score reveal or entry

That is much more specific, and much more faithful, than collapsing all of that into one generic “game over screen.”

### Practical Porting Consequence

The future port should preserve top-out as a multi-phase presentation:

1. immediate gameplay-side failure feedback
2. staged dissolve of the gameplay image
3. saved-under `GAME OVER` overlay
4. explicit overlay removal before frontend/high-score bootstrap

If we skip that middle dissolve or fold the overlay directly into the high-score path, the result will feel noticeably less like the original.

### Next 10 Strongest Moves

1. Revisit the first visible `state 9` bootstrap against captures using the newer gameplay-transition and frontend-layering models together.
2. Expand the owned raw artifact set around options, keyboard setup, and sound setup edge paths so the secondary frontend states are grounded like the main menu.
3. Keep searching for indirect or computed dirty-map writes that could weaken or refine the current mark-`3` propagation model.
4. Keep the `0x1765a` question open, but narrow it to reachability confirmation rather than broad behavior analysis.
5. Continue converting these top-out, alert, and tracked-transient findings into explicit implementation constraints for the future `C++23 + SDL3` port.
6. Do a fresh evidence pass on post-game-over audio tails versus the high-score bootstrap so SFX continuity is as precise as music continuity.
7. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
8. Tighten whether any alert refresh paths can visibly coexist with tracked transient overwrites in the same 8x4 dirty cells.
9. Refresh the root session log once the next frontend or post-game-over pass lands, because the transition model is now substantially sharper.
10. If we stay in reverse engineering mode another long stretch, start a port-facing renderer contract document that groups these frame-boundary findings into direct implementation rules.
