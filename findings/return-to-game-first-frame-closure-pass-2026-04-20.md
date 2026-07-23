# Return-to-Game First-Frame Closure Pass

Date: 2026-04-20

## Summary

This pass closes one practical port-facing question that had been spread across several older gameplay-transition findings:

- what exactly can the first visible frame after `Return to Game` contain

Current best closure:

- `Return to Game` does **not** run the `New Game` bootstrap
- it restores the clean saved gameplay snapshot
- resets the timing baseline
- then presents exactly one fresh gameplay-owned frame on preserved live run state

So the first resumed frame is now specific enough to preserve directly.

## Why This Pass Was Needed

The project already had the important ingredients:

- [transition-snapshot-and-music-continuity-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/transition-snapshot-and-music-continuity-pass-2026-04-14.md)
- [gameplay-present-order-and-resume-delta-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-present-order-and-resume-delta-pass-2026-04-14.md)
- [gameplay-edge-paths-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-edge-paths-pass-2026-04-14.md)
- [resumed-frame-and-post-gameover-transition-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/resumed-frame-and-post-gameover-transition-pass-2026-04-14.md)
- [26d000-state2-state4-caller-context-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state2-state4-caller-context-pass-2026-04-20.md)

What was missing was one closure-grade statement that isolates the `Return to Game` path itself and says:

- what state is restored
- what is preserved
- what can visibly change before the first resumed present
- what definitely does **not** happen on that path

## Findings

### 1. `Return to Game` Uses Dispatcher State `2`, Not The State-`4` New-Game Fork

The caller-context split is now specific enough to reuse safely:

- state `2`
  - live session-loop caller
  - resume existing gameplay snapshot
- state `4`
  - live session-loop caller
  - clear dirty map through `0x2d88`
  - run `0x05e0`
  - re-seed a fresh run

So the `Return to Game` path is already cleanly separated from the `New Game` path.

Important negative consequence:

- `Return to Game` does **not** call `0x05e0`
- it does **not** clear the logical board
- it does **not** reseed score / lines / level
- it does **not** promote a new current piece / next piece pair

### 2. The Restored Snapshot Is The Clean Gameplay Image, Not A Frame-Local Particle Composite

The saved gameplay image at `0x2c69f` is already tighter than a generic "whatever was on screen" interpretation.

On the gameplay-to-frontend handoff:

- `0x17875` restores tracked transient pixels first
- the `Esc` handoff check happens after that restore
- no new `0x2f24` transient draw lands before `0x3830` snapshots the working screen

So the resumed path restores:

- the clean gameplay image for that moment

not:

- a one-frame tracked-particle overlay composite

That matters because the resumed frame should be compared against a stable gameplay snapshot layer, not against a transient-heavy screenshot.

### 3. The Timing Baseline Resets, So Menu Time Does Not Become A Catch-Up Burst

After state `2` returns to the session loop, the outer loop does:

- write `0x184cb = 1`
- call `0x24d0` with `AL = 1`

That `AL = 1` path is the timing-baseline reset, not an ordinary page flip or catch-up computation.

Practical consequence:

- time spent in the frontend does **not** accumulate into a long resumed gameplay catch-up burst
- the resumed path gets one fresh gameplay-step budget before the first gameplay-side present

So the first resumed frame is not:

- snapshot restore plus many delayed gameplay steps

It is:

- snapshot restore
- one fresh gameplay-owned frame

### 4. The First Visible Resumed Frame Has A Small, Known Delta Set

The gameplay-side presentation order remains:

1. `0x17875`
   - tracked transient cleanup
2. `0x2e18`
   - persistent transient simulation update
3. `0x09c8`
   - one gameplay step with direct block-writer activity
4. `0x206c`
   - alert-tile restore / reveal work
5. `0x2f24`
   - tracked transient redraw
6. `0x17719`
   - dirty flush
7. `0x24d0(0)`
   - present

That means the first resumed visible frame can legitimately differ from the restored snapshot through only these families:

- tracked cleanup from the previous rendered frame
- one gameplay-step worth of direct block writes
- one alert-tile update
- one tracked transient overlay draw

That is now the practical delta contract for `Return to Game`.

### 5. The Preserved State Is Gameplay-Stateful, Not Freshly Bootstrapped

The preservation matrix already gives the clean split.

What `Return to Game` preserves:

- logical board contents
- current and next piece state
- gravity accumulator and rate
- score, lines, and level totals
- live-game flag
- current music track
- live pressed-key table
- tracked transient restore queue until outer-loop cleanup reaches it

What `Return to Game` does **not** preserve:

- stale frontend release events

So the resume path behaves like:

- continue the same run
- on the same cleaned gameplay snapshot
- with preserved live state
- but without leaking menu-driving release events into gameplay

### 6. Held Keys Can Carry, But They Act On Preserved Runtime State

The dispatcher tail clears:

- the release latch at `0x2c227`

It does **not** clear:

- the live pressed-state table at `0x2c22b`

So physically held gameplay keys can still affect the first resumed gameplay step.

Because `Return to Game` preserves repeat timers, lockouts, and accumulator state, that carry is qualitatively different from `New Game`:

- on `New Game`
  - held keys act on freshly reset gameplay state
- on `Return to Game`
  - held keys act on the preserved run state that was already live when the menu was opened

That is an important fidelity distinction for the future port.

### 7. Music Continuity Is Part Of The Resume Contract

The state-`2` resume path does not call:

- `0x6544`
- music stop helpers
- explicit fade-out helpers

So the current music track continues through `Return to Game`.

If the player changed the `Music:` row while paused, gameplay resumes under the newly selected track.

That makes soundtrack continuity a real part of the resume behavior, not just an incidental side effect.

## Closure

The current best reading is now:

- `Return to Game` restores the clean saved gameplay snapshot
- preserves the live run state
- clears the frontend release latch
- keeps the live pressed-key table
- resets the timing baseline
- then presents exactly one fresh gameplay-owned frame whose visible delta is limited to:
  - tracked cleanup
  - one gameplay step
  - one alert update
  - one tracked transient redraw

That is now specific enough to preserve directly in the port and specific enough to pause as a decompilation sub-branch.

## Port Implication

For a faithful port, `Return to Game` should be modeled as:

1. restore the clean saved gameplay snapshot
2. do not run `New Game` rebootstrap helpers
3. clear menu release-latch state only
4. keep physically held gameplay keys live
5. reset timing baseline instead of catching up for all menu dwell time
6. run one fresh gameplay-owned frame on preserved run state
7. keep the current music track playing

That is a much better target than either:

- immediate static resume with no live step
- or a long catch-up burst based on time spent in the menu

## Artifact

This pass adds:

- [return-to-game-first-frame-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/return-to-game-first-frame-closure.json)

## What I Now Treat As Resolved

- the first visible `Return to Game` frame is no longer a vague resume seam
- the resume-side first-frame delta set is bounded and small
- the state split between `Return to Game` and `New Game` is now concrete enough to preserve without guesswork

## Next Ordered Step

- pause the `Return to Game` first-frame branch unless one of these becomes newly useful:
  - a direct capture of the resume moment with visible held-input edge cases
  - a port milestone that needs an implementation contract for resume behavior
  - a later static pass that needs one of the bounded delta families split even further
