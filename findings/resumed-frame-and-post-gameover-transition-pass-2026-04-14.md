# Resumed Frame And Post-Game-Over Transition Pass

Date: 2026-04-14

## Summary

This pass closes five neighboring fidelity questions together:

- what can actually change on the first resumed gameplay frame
- whether any non-tracked transient systems still diverge from the saved snapshot
- what the finished game-over handoff really gives to frontend state `9`
- how much audio continuity survives into the high-score flow
- how these rules should be captured for the future source port

The biggest practical result is that this seam is now bounded much more tightly than before.

- the first resumed gameplay present can only differ from the saved gameplay snapshot through a small, known set of systems
- the `GAME OVER` overlay is not part of the state-`9` snapshot
- music continuity stays intact through the post-game-over handoff
- there is no current evidence of a hidden extra overlay system appearing between snapshot restore and the first resumed or post-game-over frontend frame

## 1. The First Resumed Gameplay Frame Now Has A Bounded Delta Set

The outer gameplay loop and per-step order now let us describe the first resumed frame more precisely.

The relevant order is:

1. `0x17875`
   restore tracked transient pixels from the prior rendered frame
2. first-step input and `Esc` handoff check
3. `0x2e18`
   update particle/object simulation state
4. `0x09c8`
   run one gameplay step
5. `0x206c`
   update the alert-tile animation
6. `0x2f24`
   draw particle/object pixels for this rendered frame
7. `0x17719`
   flush dirty cells
8. `0x24d0(0)`
   present and pace

Because the frontend return path resets the timing baseline through `0x24d0(AL = 1)`, the first post-frontend gameplay present does not receive a long catch-up burst for time spent in the menu.
It gets one fresh gameplay-step budget.

That means the first resumed gameplay image can differ from the saved snapshot only through:

- tracked-pixel cleanup at the start of the frame
- one particle/object simulation update
- one live gameplay step
- one alert-tile animation update
- one fresh particle/object draw for the new rendered frame

That is a much better fidelity boundary than a vague "resume and let the game catch up" model.

## 2. Snapshot Divergence Is Limited To Persistent Live Systems, Not Hidden Overlays

We already knew the saved gameplay snapshot excluded frame-local tracked particle overlays:

- `0x17875` runs before the `Esc` handoff can enter `0x3830`
- no new `0x2f24` draw occurs between that restore and the dispatcher snapshot

This pass tightens the remaining non-tracked question.

Current evidence supports only two persistent non-tracked systems that can still diverge between the saved snapshot and the first resumed gameplay present:

- the particle/object pool advanced by `0x2e18`
- the board-alert subsystem advanced by `0x206c`

Both are true live-state systems, not stale one-frame overlays.

### `0x2e18` Particle/Object Update

`0x2e18` walks the active linked list rooted at `0x2caa3`, advances positions by velocity, ages objects, and recycles dead or offscreen nodes.
It does not directly draw pixels.
But because it runs before `0x2f24`, it can change which objects are still alive and where their new transient pixels will be drawn on that first resumed rendered frame.

So resumed-frame divergence from the saved snapshot can legitimately include:

- a particle vanishing because it expired during the first resumed step
- a particle moving before it is redrawn
- no particle difference at all if the active list was empty

### `0x206c` Alert-Tile Update

`0x206c` is a true visible first-frame delta source, not just internal bookkeeping.
Its live update path writes directly into the working screen and dirty map through the alert-tile helpers.
So if an alert face was already active when the menu opened, the first resumed frame can legitimately show one more alert reveal step, hold step, or cleanup step before the first gameplay-side present.

### No Current Evidence For A Third Hidden Divergence Layer

Within the current outer-loop model, we do not have evidence for another separate transient presentation system between:

- saved gameplay snapshot restore
- first resumed gameplay present

So the best current fidelity model is:

- tracked one-frame overlays are restored before snapshotting
- persistent live systems continue evolving on resume
- the first resumed frame is the saved gameplay image plus one clean live update pass

That is a much smaller and more faithful gap than earlier.

## 3. The Finished Game-Over Handoff Restores The Underlay Before State `9`

The finished game-over seam is now substantially cleaner.

### `0x2300` Draws The Overlay And Saves Its Background

When the game-over dissolve completes, the runtime calls `0x2300`.
That helper:

- copies the background under the `GAME OVER` region into `0x2c603`
- blits the non-zero bytes of chunk `8` over the gameplay view
- marks that overlay region dirty

So the `GAME OVER` text is a real temporary overlay with its own saved-under background, not a destructive rewrite of the gameplay screen.

### `0x23ac` Removes The Overlay Before Entering State `9`

At the outer session-loop handoff, the `0x184db == -2` path calls `0x23ac` before entering frontend state `9`.
`0x23ac` restores the saved background from `0x2c603` and marks the same region dirty again.

The important implication is:

- the state-`9` frontend entry does not inherit a gameplay snapshot with the `GAME OVER` text still painted on top

It inherits the gameplay underlay after the overlay has already been removed.

That is a strong fidelity detail for the port.

If we naively carried the overlay into the high-score path, the transition would be wrong.

## 4. State `9` Uses The Shared Frontend Bootstrap, Then The High-Score Reveal

The game-over to high-score handoff is not a bespoke direct jump into a table renderer.

After `0x23ac` restores the gameplay underlay, `0x3830` still uses its normal shared entry sequence:

1. save the current working screen into `0x2c69f`
2. fade out the gameplay palette through `0x64f0`
3. copy the title/logo base screen into the working and visible pages
4. run the chunk-7 frontend object fade-in through `0x63b8`
5. dispatch to state `9`

State `9` then reaches the high-score handler, where `0x50b0` runs:

- qualification insert if the score qualifies
- the custom 48-frame high-score reveal
- name entry or plain display

So the first visible post-game-over frontend stage is best modeled as:

- cleaned gameplay underlay saved away
- standard frontend bootstrap
- then high-score-specific reveal

That is more faithful than a simplified "GAME OVER goes straight into the high-score table" interpretation.

## 5. Post-Game-Over Audio Continuity Is Stronger Than The Remaining SFX Ambiguity

Music continuity stays clean here.

We already know:

- direct `0x6544` call sites currently resolve only to cold startup and the main-menu `Music` row
- the shared state-`2` and state-`4` dispatcher tails do not reseed music
- the hard-exit state `3` is the only direct frontend-side fade-out path

This pass tightens the state-`9` side of that story.

Current direct call evidence does **not** show the high-score handler `0x50b0` calling:

- `0x6544`
- `0x6817`
- `0x67aa`
- `0x699e`
- `0x69b0`

So the state-`9` flow does not currently look like an audio reset point.

### What This Means For Music

The safest current model is:

- the active music track continues into the post-game-over high-score flow
- state `9` does not reseed, stop, or fade music by itself

### What This Means For SFX

The SFX story is slightly narrower but still useful.

We do not currently have evidence that the state-`9` handoff explicitly stops warning or game-over SFX.
So any still-active one-shot effect would naturally continue until completion.

But we also now know the game-over sound itself is triggered at the spawn-failure point, long before the row-by-row dissolve finishes.
Because the dissolve then walks `0xa0` rows before `0x184db` reaches `-2`, the most likely practical outcome is:

- the game-over sound has already finished naturally before the state-`9` handoff

So the honest current reading is:

- music continuity is strong and should be preserved
- SFX are not explicitly cut at the handoff
- in practice, the main game-over SFX likely ends before the high-score frontend becomes active because the dissolve is long

That is accurate enough for the port without overstating certainty we do not have.

## Porting Impact

For the future C++23 port, the safest faithful transition rules here are:

- restore tracked transient pixels before saving or resuming gameplay imagery
- treat the first resumed gameplay frame as exactly one fresh live step on top of the restored gameplay image
- preserve the particle/object pool and alert-tile subsystem across menu sessions
- remove the `GAME OVER` overlay before entering the high-score frontend flow
- treat state `9` as a normal frontend bootstrap plus high-score reveal, not a bespoke overlay-to-table jump
- preserve music continuity across the game-over to high-score handoff
- do not invent an explicit SFX cutoff at that handoff unless later runtime tracing proves one

## Bottom Line

This pass closes a useful fidelity seam.

- the first resumed gameplay frame now has a bounded and believable delta set
- the state-`9` handoff is cleaner than before because the `GAME OVER` overlay is removed before frontend snapshotting
- the high-score path behaves like part of the normal frontend presentation family
- the soundtrack remains more continuous than a naive scene-based model would suggest

That is exactly the kind of detail that will keep the eventual port feeling true instead of merely similar.

## 5 Next Strongest Moves

1. Tighten the exact trigger semantics for the finished game-over to state-`9` handoff, especially whether it is always `Esc`-release gated in practice or whether another path auto-advances under some conditions.
2. Tighten the first visible state-`9` frame against runtime capture evidence so we can confirm how much of the shared frontend bootstrap is actually perceptible before the high-score reveal dominates.
3. Resolve the remaining `0x17719` dirty-cell flush behavior deeply enough to preserve transient layering and first-frame presentation with less guesswork.
4. Tighten the `0x2938` page-upload and `0x24d0` baseline-reset interaction across all transition tails so the future port reproduces the same first-visible-frame cadence.
5. Keep building the transition-preservation spec into a true implementation contract for the C++23 port while we finish the few remaining frame-boundary helpers.
