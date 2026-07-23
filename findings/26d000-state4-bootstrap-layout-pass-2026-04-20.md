# `0x26D000` State-4 Bootstrap Layout Pass

Date: 2026-04-20

## Summary

This pass tied the fully named state-`4` bootstrap image back to the fixed gameplay-screen layout and the owned gameplay captures.

Main result:

- the state-`4` bootstrap image now maps cleanly onto the shipped gameplay layout
- every named bootstrap contributor lands on a stable panel already visible in owned captures
- the remaining uncertainty is no longer "what is on the bootstrap image?"
- it is "do we own a frame-perfect capture of the moment before the first live-piece draw?"

That means the `0x26D000` static branch is now specific enough to pause without losing the thread.

## New Owned Artifact

- [26d000-state4-bootstrap-layout.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-state4-bootstrap-layout.json)

This artifact records:

- the fixed gameplay-screen regions used by the bootstrap path
- the named state-`4` contributors that land in each region
- the owned capture files that visually confirm those regions
- the remaining evidence limit on the exact pre-live-piece moment

## Key Artifacts Reused

- [26d000-state4-alert-split-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-alert-split-pass-2026-04-20.md)
- [26d000-state4-bootstrap-contributor-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-contributor-pass-2026-04-20.md)
- [new-game-visible-bootstrap-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/new-game-visible-bootstrap-pass-2026-04-14.md)
- [behavior-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/behavior-spec.md)
- [06-newgame.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/06-newgame.png)
- [07-gameplay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/07-gameplay.png)
- [08-gameover.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/08-gameover.png)

## Findings

### 1. The Bootstrap Regions Match The Fixed Gameplay Layout Exactly

The state-`4` bootstrap contributors now land on fixed gameplay-screen regions that are stable across owned captures:

- playfield interior clear
  - `x = 109`
  - `y = 21`
  - `width = 80`
  - `height = 160`
- next-piece preview interior
  - `x = 30`
  - `y = 56`
  - `width = 32`
  - `height = 16`
- alert tile region
  - `x = 20`
  - `y = 140`
  - `width = 50`
  - `height = 50`
- piece-stat digits
  - `x = 248`
  - `y = 47, 58, 69, 80, 91, 102, 113`
- named gameplay HUD counters
  - `HIGH-SCORE`
    - `x = 36`
    - `y = 89`
  - `SCORE`
    - `x = 36`
    - `y = 101`
  - `LEVEL`
    - `x = 60`
    - `y = 112`
  - `LINES`
    - `x = 144`
    - `y = 195`

Those placements match the left panel, right statistics panel, bottom `LINES` band, and lower-left alert area visible in the owned screenshots.

### 2. The Owned `New Game` Capture Matches The Bootstrap Model After The Shared First-Step Piece Draw

[06-newgame.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/06-newgame.png) shows:

- left panel populated with:
  - `NEXT`
  - `HIGH-SCORE`
  - `SCORE`
  - `LEVEL`
- bottom `LINES` band
- right panel populated with seven per-piece counters
- no alert icon in the lower-left alert region
- one live piece already visible near the top of the well

That is exactly what the current branch predicts for a post-bootstrap frame:

- state-`4` staged counters / preview / piece-stat panel are present
- clean alert background is present
- the live piece has already been added by the first shared `0x09c8` step

So the `New Game` capture supports the current split rather than blurring it.

### 3. The Ordinary Gameplay Capture Confirms These Regions Are Stable, Not One-Off Restart Artifacts

[07-gameplay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/07-gameplay.png) shows the same fixed layout with:

- the same left counter panel
- the same preview box
- the same right seven-piece statistics panel
- the same bottom `LINES` band

That matters because it confirms the state-`4` bootstrap image is not inventing a special alternate layout.
It is populating the normal gameplay scene in its ordinary fixed regions.

### 4. The Game-Over Capture Confirms The Alert Region And Panel Map Remain Spatially Stable

[08-gameover.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/08-gameover.png) keeps the same basic gameplay panel map while adding:

- the `GAME OVER` overlay over the well
- a live alert-face graphic in the lower-left alert region

That strengthens the alert-side closure from the prior pass:

- the state-`4` bootstrap should leave this region clean
- alert art appears here only when later gameplay-side logic activates it

### 5. The Remaining Gap Is Now A Frame-Selection Problem, Not A Static-Identity Problem

After this pass, the missing evidence is narrow.

We do **not** still lack:

- the identity of the counters
- the identity of the right-panel piece stats
- the identity of the preview box
- the location or role of the alert region
- the split between staged bootstrap and later first-step live-piece draw

What we still lack is only:

- a frame-perfect owned capture of the instant after the staged bootstrap is ready but before the first shared live-piece draw becomes visible

That is a capture-timing gap, not a remaining static RE ambiguity.

### 6. The `0x26D000` Static Branch Is Specific Enough To Pause

This branch started as a broad late-lane question.
It is now narrow enough to preserve safely:

- flat `0868` lane is not the state-`4` all-pages upload
- the first gameplay-side flush is a mixed frame
- the state-`4`-unique visible slice is:
  - named left-panel counters
  - seven-piece statistics panel
  - next-piece preview
  - clean alert-region restore
- the live piece belongs to the shared first gameplay step

That is already precise enough to guide future porting work and future capture verification.

## Practical Porting Impact

The restart image can now be described in concrete UI terms instead of helper names:

- blank well interior and preview first
- then named left-panel counters, bottom `LINES`, right piece-stat panel, next preview, and clean alert background
- then first live-piece draw joins on the first gameplay-owned frame

That is strong enough to serve as a direct parity target.

## Next Strongest Move

Pause the `0x26D000` static branch here unless one of these becomes available:

1. a frame-perfect restart capture that can isolate the exact pre-live-piece bootstrap image
2. a new need to implement or test restart-scene parity in the port skeleton
3. a reason to compare this bootstrap image against another unresolved subsystem

If we keep decompilation moving now, the stronger use of time is another unresolved subsystem rather than more refinement on this branch.

## Bottom Line

The important closure is:

- the state-`4` bootstrap image is now fully named, spatially grounded in owned captures, and specific enough to pause
