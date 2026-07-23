# State-9 First-Visible-Frame Closure Pass

Date: 2026-04-20

## Summary

This pass closes one transition question that had remained split across several capture and executable findings:

- what the first visible `state 9` frame actually is

Current best closure:

- the first visible `state 9` phase is a short shared frontend-bootstrap interval
- it is **not** an instant jump from `GAME OVER` directly to a fully settled high-score table

The exact first visible frame is still approximate because the timing bound is similarity-based, but the behavior family is now specific enough to preserve directly.

## Why This Pass Was Needed

The project already had the important ingredients:

- [topout-exit-hiscore-menu-capture-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/topout-exit-hiscore-menu-capture-pass-2026-04-14.md)
- [state9-bootstrap-capture-support-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/state9-bootstrap-capture-support-pass-2026-04-14.md)
- [state9-bootstrap-onset-timing-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/state9-bootstrap-onset-timing-pass-2026-04-15.md)
- [topout-slot5-fit-vs-state9-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/topout-slot5-fit-vs-state9-pass-2026-04-15.md)
- [resumed-frame-and-post-gameover-transition-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/resumed-frame-and-post-gameover-transition-pass-2026-04-14.md)

What was missing was one closure-grade statement that combines those into a single answer:

- when the post-game-over presentation first becomes `state 9`-like
- what is visibly present in that first `state 9` stage
- what is definitely absent by then

## Findings

### 1. The `GAME OVER` Overlay Is Gone Before The First Visible `State 9` Phase

The executable-side transition seam already gives the important negative rule:

- the late `GAME OVER` overlay is drawn by the chunk-`8` overlay path
- `0x23ac` restores the saved-under gameplay region before entering frontend state `9`

So the first visible `state 9` frame should **not** be modeled as:

- `GAME OVER` text still painted over the gameplay image

That old overlay has already been removed before the frontend bootstrap becomes visible.

### 2. The First Visible `State 9` Stage Is The Shared Frontend Bootstrap

The owned transition capture directly supports that the post-game-over path does not jump straight into a fully settled table screen.

Most useful frames:

- [frame-021.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-021.png)
- [frame-022.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-022.png)
- [frame-023.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-023.png)
- [frame-024.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-024.png)
- [frame-026.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-026.png)

What those frames jointly support:

- a darkened late game-over frame still exists before the frontend reappears
- a title-dominant frontend stage becomes visible before the table fully settles
- only afterward does the high-score display clearly dominate the screen

That means the first visible `state 9` stage is now best modeled as:

- shared title/logo bootstrap first
- then the high-score-specific reveal grows dominant

### 3. The Timing Window Is Now Bounded Tightly Enough For Port Pacing

The similarity-timing artifact already gives a sub-second onset bound:

- [state9-bootstrap-onset-analysis.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/state9-bootstrap-onset-analysis.json)

Current best timing:

- `state9_crossover_onset`
  - `21.083333s`
- stronger persistent `state9_bootstrap_onset`
  - `21.333333s`
- strongest bootstrap-vs-game-over separation
  - `22.5s`
- first persistent high-score crossover vs bootstrap reference
  - `23.166667s`

So the first visible `state 9` interval is now best bounded as:

- bootstrap-like presentation becomes visible around `21.08s..21.33s`
- high-score-family dominance emerges later, around `23.17s`

That gives the port a useful pacing target without pretending we have semantic frame-perfect classification.

### 4. The First Visible `State 9` Frame Is Still Inside The Shared Frontend Visual Family

The capture-side evidence and earlier screenshot support still agree on the same broader rule:

- the shared `ACiD TETRIS` title/logo base remains present
- the chunk-7 floating-object layer remains part of the high-score family

So the first visible `state 9` phase should **not** be modeled as:

- a bespoke table-only screen
- a gameplay-derived background
- or a special one-off high-score backdrop

It is still inside the shared frontend family.

### 5. Audio Does Not Need A Special `State 9` Cutoff

The top-out SFX fit pass already tightened the remaining practical audio seam:

- slot `5` is the correct top-out one-shot
- its fitted end is about `9.309s`
- the first perceptible `state 9` bootstrap support arrives much later, around `21s+`

So the first visible `state 9` frame does not need a synthetic SFX cutoff rule.

The honest current reading is:

- music continuity carries through the handoff
- the top-out one-shot has already ended naturally well before the first visible `state 9` phase

## Closure

The current best reading is now:

- the first visible `state 9` frame is a short shared frontend-bootstrap interval after the `GAME OVER` overlay has already been removed
- it is title/bootstrap-dominant rather than table-dominant
- the high-score table fully settles later

That means the old weaker wording:

- "`state 9` bootstrap is mostly executable-derived"

can now be retired in favor of:

- executable-strong and capture-supported, with approximate but usefully bounded onset timing

## Port Implication

For a faithful port, the post-game-over path should preserve this sequence:

1. remove the saved-under `GAME OVER` overlay
2. run the shared frontend bootstrap
3. allow a short perceptible bootstrap interval before the table fully settles
4. keep the high-score family on the shared title/logo plus chunk-7-object presentation
5. keep music continuity intact without inventing a special state-`9` audio cutoff

That is a stronger and more faithful target than:

- direct jump from late game-over overlay to settled high-score table

## Artifact

This pass adds:

- [state9-first-visible-frame-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/state9-first-visible-frame-closure.json)

## What I Now Treat As Resolved

- the first visible `state 9` phase is now capture-supported strongly enough to preserve directly
- the post-game-over path is not a direct overlay-to-table jump
- the remaining uncertainty is timing precision, not visual family or stage identity

## Next Ordered Step

- pause the `state 9` first-visible-frame branch unless one of these becomes newly useful:
  - a tighter native-window transition capture
  - a port milestone that needs an explicit post-game-over scene contract
  - a later semantic frame-labeling pass that can refine the bootstrap-to-table crossover further
