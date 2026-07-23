# Gameplay Resume Early-Return Closure Pass

Date: 2026-04-21

## Summary

This pass closes the last machine-readable open question in the gameplay-present-order artifacts:

- whether any owned `0x09c8` edge-path can defer all visible ordinary block-blit work past the first presented resumed frame

Current best closure:

- **yes**, but only in two real resumable cases:
  - pending line-clear collapse while `0x184df > 0`
  - active top-out dissolve while `0x184db >= 0`
- **no** for the finished game-over sentinel:
  - `0x184db == -2` is not a real `Return to Game` case

So the earlier "one fresh gameplay-owned frame" contract remains right in spirit, but it now needs one explicit qualifier:

- the first resumed frame can legitimately be a collapse-only or dissolve-only gameplay step instead of an ordinary erase / move / spawn / redraw step

## Why This Pass Was Needed

The owned `0x09c8` export already showed the structural fork:

- `0x184df > 0`
  - call `0x1d04`
  - return immediately
- `0x184db >= 0`
  - call `0x1f8c`
  - return immediately
- `0x184db == -2`
  - jump to the late tail at `0x10e1`

What was still unclear was not the gameplay helper itself.
It was the resume applicability:

- can a real `Return to Game` preserve those edge-state fields into the first resumed outer frame
- or are they only theoretical `0x09c8` branches that never matter on resume

That distinction matters for both preservation wording and the future port.

## Findings

### 1. State `2` Resume Does Not Sanitize `0x184df` Or `0x184db`

The owned dispatcher export at `0x3830` now gives the practical boundary directly.

State-`2` resume:

- restores the saved screen snapshot
- fades the frontend out
- saves setup
- restores the gameplay image back into working / visible pages
- runs the simple palette reveal
- returns

Only state `4` adds:

- `0x2d88`
- `0x05e0`

That matters because `0x05e0` is the gameplay-side bootstrap that explicitly resets:

- `0x184db = -1`
- `0x184df = 0`
- `0x2c72b = 1`

State `2` does **not** call `0x05e0`, so the resume path does not perform that sanitization.

### 2. Pending Line-Clear Collapse Is A Real Resume-Preserved Exception

At the top of `0x09c8`:

- if `0x184df > 0`
  - `0x09c8` calls `0x1d04`
  - returns before the ordinary erase / input / gravity / spawn band

The line-clear passes already closed the gameplay-side behavior:

- `0x09c8` seeds pending clears into `0x184df`
- subsequent steps with `0x184df > 0` short-circuit into `0x1d04`
- the outer gameplay loop still runs:
  - `0x17875`
  - `0x2e18`
  - `0x09c8`
  - `0x206c`
  - `0x2f24`
  - `0x17719`
  - `0x24d0`

The important resume-side closure is:

- row `7` `Return to Game` is gated only by live-game flag `0x2c72b`
- no owned line-clear path writes `0x2c72b`
- the state-`2` resume tail does not reset `0x184df`

So a pause taken during line-clear collapse can legitimately resume into:

- tracked cleanup
- object update
- `0x1d04` collapse-only work
- alert update
- tracked redraw
- present

with no ordinary `0x11e0` / `0x10ec` live-piece band on that first resumed frame.

### 3. Active Top-Out Dissolve Is Also A Real Resume-Preserved Exception

At the top of `0x09c8`:

- if `0x184db >= 0`
  - `0x09c8` calls `0x1f8c`
  - returns before the ordinary erase / input / gravity / spawn band

The owned top-out chain is now concrete:

- failed spawn arms `0x1f8c(EAX = 2)` and sets `0x184db = 0`
- later `0x1f8c` update calls:
  - `0x1edc(row = 0x184db + 0x15)`
  - increments `0x184db`
- only when the dissolve finishes does `0x1f8c`:
  - store `0x184db = -2`
  - clear live-game flag `0x2c72b = 0`
  - draw the late `GAME OVER` overlay

That gives the resume answer cleanly:

- while `0x184db` is still in the active dissolve range `0 .. 0x9f`
  - live-game flag is still set
  - row `7` `Return to Game` remains actionable
  - state `2` resume preserves that dissolve state

So a pause taken during active top-out dissolve can legitimately resume into:

- tracked cleanup
- object update
- `0x1f8c` dissolve-only work
- alert update
- tracked redraw
- present

again without the ordinary live-piece erase / redraw band.

### 4. `0x184db == -2` Is Not A Real `Return to Game` Resume Case

The finished game-over sentinel now closes differently.

Owned evidence already shows:

- when `0x1f8c` finishes, it stores:
  - `0x184db = -2`
  - `0x2c72b = 0`
- the session-loop released-`Esc` handoff then:
  - sees `0x184db == -2`
  - restores the `GAME OVER` underlay through `0x23ac`
  - enters frontend state `9`

And the main-menu row-`7` gate is still only:

- `0x2c72b == 1`

So by the time `0x184db == -2` exists as a stable finished game-over state:

- `Return to Game` is no longer actionable

That means the `0x09c8` top check:

- `0x184db == -2 -> jump 0x10e1`

is a real gameplay helper path, but **not** a real first-resumed-frame exception for `Return to Game`.

### 5. The Resume Contract Needs A Qualified Wording, Not A Rewrite

The older closure was directionally right:

- state `2` restores the clean saved gameplay snapshot
- resets the timing baseline
- then gives the run one fresh gameplay-owned frame before the next presented result

What changes now is only the allowed form of that first gameplay-owned frame.

It is not always:

- ordinary live-piece erase / move / spawn / redraw

It can instead be one of three gameplay-owned first-step families:

1. ordinary live gameplay step
2. collapse-only line-clear step through `0x1d04`
3. dissolve-only top-out step through `0x1f8c`

The finished `-2` sentinel is excluded because it is no longer resumable gameplay.

## Closure

The open question in the gameplay-present-order artifacts is now closed as:

- **yes**, a real `Return to Game` can defer all ordinary direct block-blit work inside `0x09c8`
- but only when resume preserves one of two live edge states:
  - pending line-clear collapse `0x184df > 0`
  - active top-out dissolve `0x184db >= 0`
- **no**, the finished game-over sentinel `0x184db == -2` is not a real resume case because it has already cleared the live-game flag

So the future port should preserve this qualified rule:

- the first resumed frame is still exactly one fresh gameplay-owned frame after timing-baseline reset
- but that gameplay-owned frame may be:
  - ordinary
  - collapse-only
  - or dissolve-only

## Artifact

This pass adds:

- [gameplay-resume-early-return-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-resume-early-return-closure.json)

## What I Now Treat As Resolved

- the last open question in [gameplay-present-order.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-present-order.json:1) is closed
- the first resumed gameplay frame now has explicit real exception families instead of one implicit ordinary-step assumption
- the finished game-over `-2` branch is no longer mixed into the `Return to Game` fidelity model

## Next Ordered Step

- pivot to another bounded unresolved subsystem rather than keep squeezing this now-closed resume seam
- current best small static target:
  - high-score name-entry ownership / helper-boundary cleanup around `0x50b0`
