# `0x26D000` State-4 First Gameplay Flush Pass

Date: 2026-04-20

## Summary

This pass tightened the first gameplay-side localized flush that follows the unique state-`4` new-game fork.

Main result:

- the first gameplay-side `0x17719 -> 0x24d0` after state `4` is a composite frame
- it does **not** just present `0x05e0` as-is
- it does **not** just present an ordinary shared gameplay step either
- it combines:
  - already-visible reset surfaces from the early `0x2938` upload
  - deferred new-game bootstrap redraws staged by `0x05e0`
  - shared first-loop gameplay writers that run before the next flush

The useful closure is that the state-`4`-unique contribution inside that first flush is now smaller and cleaner:

- staged new-game HUD / preview / alert bootstrap

while the rest of the first flush belongs to the shared gameplay frame driver.

## New Owned Artifact

- [26d000-state4-first-gameplay-flush.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-state4-first-gameplay-flush.json)

This artifact records:

- what is already visible before the first gameplay flush
- which `0x05e0` writes remain staged for that flush
- which shared first-loop writers join the same presented frame
- which visible effects are state-`4`-unique versus shared gameplay cadence

## Key Artifacts Reused

- [26d000-state4-surface-vs-flat-0868-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-surface-vs-flat-0868-pass-2026-04-20.md)
- [new-game-visible-bootstrap-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/new-game-visible-bootstrap-pass-2026-04-14.md)
- [startup-to-first-gameplay-present-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/startup-to-first-gameplay-present-pass-2026-04-14.md)
- [gameplay-edge-paths-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-edge-paths-pass-2026-04-14.md)
- [first-spawn-step-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/first-spawn-step-pass-2026-04-14.md)
- [cold-boot-menu-and-held-input-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/cold-boot-menu-and-held-input-pass-2026-04-14.md)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)

## Findings

### 1. The First Gameplay Flush Starts From A Partially Visible State-`4` Screen

Before the first gameplay-side localized flush happens at all, the state-`4` path has already made one important thing visible:

- the restored gameplay snapshot with:
  - board interior cleared
  - preview box cleared

That image is already on all three VGA pages because `0x05e0` used:

- `0x6274`
- `0x6274`
- `0x2938 x3`

So when the first gameplay-side `0x17719` runs, it is **not** creating the whole restart scene from scratch.
It is completing a scene that was already partially revealed by the state-`4` upload.

### 2. Only Part Of `0x05e0` Is Still Waiting For The First Flush

After the early upload, `0x05e0` continues with both visible and non-visible work.

Visible deferred work still waiting for the first gameplay-side flush:

- decimal HUD counter redraws through `0x2c58`
- preview / per-piece-stat redraw work staged by `0x1348`
- alert-region reset through `0x206c(EAX = 1)`

Non-visible run-state work that affects logic but is not itself a direct screen contribution:

- `0x32b0`
  refill random table
- `0x1330`
  clear logical board cells
- `0x09c8(EAX = 1)`
  reset gameplay repeat state
- gravity write to `0x2c717`
- live-game flag write to `0x2c72b`

So the state-`4`-unique visible portion that remains staged for the first flush is narrower than the full body of `0x05e0`.

### 3. The First Flush Also Includes Shared First-Loop Writers

After `0x3830` returns, the session loop resumes its normal gameplay cadence.

The already-owned order is:

1. `0x17875`
   tracked-pixel restore
2. `0x2e18`
   particle/object state update
3. `0x09c8`
   first live gameplay step
4. `0x206c`
   alert-tile update
5. `0x2f24`
   tracked transient draw
6. `0x17719 -> 0x24d0`
   localized flush and present

That means the first gameplay-side present after state `4` is not a pure bootstrap completion frame.
It is also the next ordinary gameplay-owned frame.

### 4. The Shared First Live Step Usually Adds Less Visible Change Than The Staged Bootstrap

This is the most useful refinement from bringing the earlier gameplay-edge findings back into the `0x26D000` branch.

On a fresh run:

- `0x1348` has already reset:
  - `X = 4`
  - `Y = 0`
  - `rotation = 0`
  - gravity accumulator = `0`
- one ordinary gravity step from that state does not reach the `0x10000` row-drop threshold

So the first live `0x09c8` step is real, but it usually does **not** make the spawned piece visibly descend.

What it can still add before the first flush:

- the first live current-piece draw
- preview / counter edge work on the spawn path
- spawn-collision or game-over edge behavior if relevant
- movement or rotation if the player is still physically holding a gameplay key

But in the ordinary fresh-spawn case, the first gameplay-side present is still visually dominated by the staged new-game bootstrap rather than by a visible gravity drop.

### 5. The State-`4`-Unique Portion Of The First Flush Is Now Well Bounded

The first gameplay-side localized flush after state `4` contains three classes of change:

Already visible before the flush:

- cleared board interior
- cleared preview box

State-`4`-unique staged bootstrap completed by the flush:

- new-run HUD digits
- preview / per-piece-stat bootstrap
- alert reset surface

Shared gameplay-frame contributions joining the same flush:

- tracked-pixel cleanup
- first live gameplay step output
- transient particle/object overlay

That is the sharpest current separation on this branch.

The first gameplay-side flush is not wholly unique to state `4`.
Only the staged bootstrap slice inside it is.

## Practical Porting Impact

For a faithful port, the restart path should preserve three layers:

1. immediate visible reset image from the early upload
2. deferred new-game bootstrap that completes on the first gameplay flush
3. shared first gameplay-frame effects that join that same presented frame

Collapsing those into one opaque "new game screen" would lose a real sequencing property of the original.

## Next Strongest Move

Do a focused static pass on the staged-bootstrap contributors only:

1. isolate exactly which `0x2c58` counter sites in `0x05e0` are part of the state-`4`-unique first-flush surface
2. separate preview / per-piece bootstrap from the first live-piece draw added later by `0x09c8`
3. keep narrowing the state-`4`-unique slice away from the shared gameplay frame driver

## Bottom Line

The important closure is:

- the first gameplay-side `0x17719` after state `4` is a mixed frame, but the state-`4`-unique part of that frame is now bounded to the staged new-game bootstrap work left dirty by `0x05e0`
