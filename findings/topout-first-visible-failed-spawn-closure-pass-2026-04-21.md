# Top-Out First-Visible Failed-Spawn Closure Pass

Date: 2026-04-21

## Summary

This pass closes one remaining gameplay-to-top-out seam:

- what the first visible failed-spawn frame can actually contain before the row-by-row dissolve takes over

Current best closure:

- the failed-spawn frame itself is still a gameplay-owned presented frame
- it can already show the updated preview/HUD work from the spawn path
- it still draws the colliding spawned live piece
- the first visible alert-`6` reveal lands through the outer `0x206c` step in that same frame
- the dissolve does **not** begin until the following gameplay step

That replaces the older loose shorthand that made top-out sound like an immediate dissolve-only takeover, or like the spawned-piece draw was skipped as soon as game-over was armed.

## Why This Pass Was Needed

The project already had two strong pieces:

- [gameplay-edge-paths-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-edge-paths-pass-2026-04-14.md)
- [topout-and-gameover-presentation-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/topout-and-gameover-presentation-pass-2026-04-14.md)

Together they already said:

- the spawn path can redraw HUD and preview state before the outer transient pass
- top-out is staged as failed spawn, then dissolve, then late overlay, then state `9`

But one visible boundary was still underspecified:

- does the failed-spawn frame already present the colliding new piece and first alert reveal
- or does the game arm top-out and immediately switch to dissolve-only behavior before that frame is shown

The raw branch now answers that cleanly.

## Findings

### 1. `0x1348` Finishes Preview/Counter Work Before The Failed-Spawn Check

The owned raw spawn slice still shows the same important order:

- `0x1011` calls `0x1348`
- `0x1348` promotes the old next piece into the live slot
- erases the old preview
- draws the new preview
- increments and redraws the per-piece statistics counter
- resets the live spawned piece to `X = 4`, `Y = 0`, `rotation = 0`

So by the time the failed-spawn collision test runs, the visible supporting gameplay state is already updated for the newly selected piece family.

### 2. The Failed-Spawn Branch Arms Top-Out But Does Not Start Dissolve Yet

The raw `0x09c8` branch shows:

- `0x102d` checks spawn collision through `0x12ac`
- on failure, `0x1046` calls `0x2008` for alert `6`
- `0x1055` calls `0x1f8c` with `EAX = 2`
- `0x1065` plays slot `5`

The important detail comes from raw `0x1f8c`:

- the `EAX = 2` entry clears only the logical board block at `0x2c23b`
- stores `0x184db = 0`
- and returns immediately

It does **not** fall through into:

- the later `0x1edc` call
- row increment
- or overlay handoff logic

So the failed-spawn frame arms the dissolve state, but is not itself a dissolve step yet.

### 3. The Spawned Live Piece Still Draws On That Failed-Spawn Frame

After the failed-spawn block, raw `0x09c8` continues into:

- `0x106a .. 0x10a1`

The only gate before the later `0x10a1` live-piece draw is:

- `0x1070` load `0x184df`
- `0x107c` branch on pending line-clear state

That branch does **not** test:

- `0x184db`
- top-out-active state
- or the just-armed `0x1f8c(EAX = 2)` path

So on an ordinary failed spawn with no pending line clear:

- the colliding newly spawned live piece is still drawn through `0x10ec`

This is the key closure that was still too loose in the older wording.

### 4. The First Visible Alert Reveal Arrives In The Same Presented Frame

The owned gameplay present order is still:

1. `0x17875`
2. `0x2e18`
3. `0x09c8`
4. `0x206c`
5. `0x2f24`
6. `0x17719`
7. `0x24d0`

And `0x2008` fresh idle-start is now closed as:

- seed effect ID
- seed lifetime
- seed reveal counter `6`
- no immediate draw

That means on a failed-spawn frame:

- `0x09c8` only stages alert `6`
- then the outer `0x206c` step performs the first visible reveal work
- then the shared flush/present boundary shows it

So the first visible failed-spawn frame can already contain:

- updated preview state
- updated piece statistics counter
- any earlier spawn-path HUD redraws
- the colliding spawned piece
- the first visible alert reveal

all before the first dissolve-only frame ever appears.

### 5. The First Dissolve-Only Frame Starts On The Following Gameplay Step

On the next gameplay step, `0x09c8` reaches its early top-out check:

- if `0x184db >= 0`, it short-circuits directly into `0x1f8c`

That is where:

- `0x1edc` finally runs for row `0x184db + 0x15`
- `0x184db` increments
- and the staged row-teardown presentation truly begins

So the visual chain is now best modeled as:

1. failed-spawn frame still presents gameplay-side spawn artifacts plus alert start
2. following gameplay frames become dissolve-only
3. late saved-under `GAME OVER` overlay appears after dissolve completion
4. state `9` bootstrap still happens only after overlay removal

## Closure

The first visible failed-spawn frame is now best treated as:

- the last gameplay-like presented frame of the run
- already carrying the new preview/HUD state
- already carrying the colliding spawned live piece
- already carrying the first visible alert reveal
- but not yet carrying any row-dissolve teardown

That makes the next frame boundary much sharper:

- the dissolve begins on the following gameplay step, not on the failed-spawn trigger frame itself

## Port Implication

For a faithful port, do **not** collapse top-out into:

- failed spawn
- instant dissolve-only frame

or into:

- failed spawn
- no visible spawned-piece draw

The better preserved sequence is:

1. finish the spawn-path redraw work
2. detect failed spawn
3. seed alert `6`, seed top-out state, play slot `5`
4. still draw the colliding spawned piece
5. let the outer alert update produce the first visible reveal in that presented frame
6. start dissolve-only frames on the next gameplay step

## Artifact

This pass adds:

- [topout-first-visible-failed-spawn-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/topout-first-visible-failed-spawn-closure.json)

## What I Now Treat As Resolved

- top-out arming and first visible dissolve are no longer treated as the same frame
- the failed-spawn frame is now specific enough to preserve directly
- the older “draw new piece only when top-out is not active” shorthand can be retired

## Next Ordered Step

- pause this failed-spawn first-visible-frame branch unless a later capture or port milestone needs exact scene choreography against real frame-time video
