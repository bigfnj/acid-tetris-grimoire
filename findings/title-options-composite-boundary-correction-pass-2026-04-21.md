# Title/Options Composite Boundary Correction Pass

Date: 2026-04-21

## Summary

This pass corrected a real artifact bug in the title/options correlation support branch and then re-ran the affected menu-composite checks.

The bug:

- the reconstructed main-menu sample in [render_frontend_menu_text.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/render_frontend_menu_text.py)
  had been missing the executable-owned `Music:` row

That mattered because earlier pulse/menu-correlation artifacts were comparing the aligned `title-options` video frames against an incomplete eight-row frontend model.

After fixing that row and regenerating track-local artifacts, the most important result is:

- the current aligned `title-options` video frames are still **not** a trustworthy certification source for the exact title/options composite

The correction improved the artifact hygiene.
It did **not** produce a newly trusted pixel-certified title/options menu/object state.

## What Was Corrected

The flat-binary string table already owns the real main-menu row family:

- `New Game`
- `Options`
- `Music:\`Continuum'`
- `Level %d`
- `High Scores`
- `Credits`
- `Exit Game`
- `Return to Game`

That exact eight-row reading was already stated in:

- [frontend-menu-resolution-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-menu-resolution-pass-2026-04-13.md)

But the renderer-side sample used by the local menu/pulse correlation branch had only seven rows.

This pass corrected that mismatch and regenerated fresh local artifacts under new 2026-04-21 output paths instead of overwriting the older branch.

## New Artifact Set

- [frontend-text-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-text-2026-04-21/manifest.json)
- [frontend-menu-pulse-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-menu-pulse-2026-04-21/manifest.json)
- [frontend-menu-pulse-correlation-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-menu-pulse-correlation-2026-04-21/manifest.json)
- [frontend-menu-pulse-delta-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-menu-pulse-delta-2026-04-21/manifest.json)
- [title-options-composite-boundary-correction-2026-04-21.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-composite-boundary-correction-2026-04-21.json)

## Main Result

### 1. The Correction Closed A Real Artifact Bug

That part is straightforward:

- the local renderer now matches the owned eight-row main-menu string family

So any future menu-composite work should use the corrected 2026-04-21 artifact family rather than the older incomplete seven-row branch.

### 2. The Corrected Full-Frame Winners Became Even More Cautious, Not More Concrete

After the correction, all six aligned `title-options` video frames now prefer:

- `options-menu.static`

not:

- `main-menu.static`
- any pulsed-row candidate

That should **not** be read as "the captures are really the options menu."

The better reading is:

- the aligned frames are still so title-base-dominated that the lighter four-row options composite simply adds less error than the heavier eight-row main-menu composite

This is visible in the aggregate penalties recorded in the new JSON artifact:

- corrected main-menu static penalty over title base: `9.285234`
- corrected options-menu static penalty over title base: about `7.85`

So the scoring is still rewarding "fewest extra text pixels" more than "real recovered frontend state."

### 3. Pulse Support Is Still Negative

The corrected delta-only pulse pass still gives the same qualitative answer as before:

- every best pulse candidate remains **negative**

Current least-bad result:

- `main-menu.row03.phase-0440.reveal-42`
- masked improvement `-18.5`
- full-frame improvement `-0.010117`

So even after correcting the main-menu row set, the aligned `title-options` video frames still do **not** support treating the current capture set as evidence for a live selected-row pulse phase.

## What This Changes

This pass does **not** weaken the executable-side frontend model.

The owned executable-side truths still stand:

- preloaded title/logo base
- shared continuous chunk-7 decorative object cycle
- text/highlight layer in front of objects
- fixed-position row redraw and pulse behavior

What this pass changes is the boundary on the capture side.

The current `title-options` aligned video-frame set should now be treated as:

- useful title-base-dominated reference material

not:

- a certification source for exact menu-text placement
- a certification source for exact chunk-7 object placement
- a certification source for selected-row pulse phase

## Implementation-Safe Call

For port-facing work, I now treat this subsystem as:

- implementation-safe

with one explicit non-port gap:

- we still do not have a native-frame or materially better-aligned capture source that can certify the exact title/options composite against desktop-video alignment noise and title-base domination

That is a capture-certification gap, not a core executable-behavior gap.

## Recommended Next Move

Do **not** spend another pass trying to squeeze exact title/options composite truth out of the current aligned video frames.

If future work needs a stronger title/options visual certification target, the next useful input is:

- a native-resolution frontend frame grab
- or a materially better-cropped/aligned capture sequence

Until then, the safer rule is:

- preserve the executable-owned frontend presentation model
- treat the current title/options capture branch as bounded by source quality rather than missing chunk-7 semantics
