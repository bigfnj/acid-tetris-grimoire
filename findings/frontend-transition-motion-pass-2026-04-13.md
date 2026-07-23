# Frontend Transition/Motion Pass

Date: 2026-04-13

## Summary

This pass tightened the frontend-animation map in two ways:

- it resolved the menu-side transition helpers around the chunk-7 frame pump
- it measured the real spatial change in the aligned title/options captures and compared that footprint against the current chunk-7 render set

The key additions were:

- [analyze_frontend_motion.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/analyze_frontend_motion.py)
- [title-options motion manifest](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-motion/manifest.json)

The function-role map was also updated in:

- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)

## Transition Helper Findings

Raw disassembly confirms the menu-side wrapper around the chunk-7 pointfield:

- `0x3d94` clears any currently drawn frontend points by calling `0x17613` for records whose active flag at `+0x18` is set
- `0x3dc0` redraws the current projected frontend points by calling `0x175c5` and storing the result back into `+0x18`
- `0x3df4` is a real menu-entry transition, not a generic helper
- `0x3ee4` is the matching menu-exit transition
- `0x3fd8` is a menu-highlight pulse helper driven by the shared phase at `0x1990f`

The important behavior in `0x3df4` / `0x3ee4` is:

- both run for `0x30` frames
- both wrap repeated `0x39c4` calls inside a clear -> redraw-text -> project -> draw-points -> flush -> present loop
- `0x3df4` feeds an increasing parameter into `0x60cc`
- `0x3ee4` feeds `max(0x2f - frame_index, 0)` into `0x60cc`

So the menu captures are not just observing the steady `0x39c4` state. They sit inside a larger transition framework that redraws the title-menu text each frame.

## Capture Delta/Motion Results

The new motion analysis writes:

- per-capture delta masks against the recovered title screen
- a union mask of all title/options capture deltas
- a temporal motion mask showing only pixels that actually change across the aligned capture set

Useful output files:

- [capture delta union mask](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-motion/title-options.capture-delta-union.png)
- [temporal motion union mask](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-motion/title-options.temporal-motion-union.png)

The real aligned-capture footprint is:

- capture delta union: `12571` pixels, bbox `(0,3) -> (297,173)`
- temporal motion union: `7452` pixels, bbox `(0,59) -> (124,173)`

That is a much larger and lower-on-screen footprint than the current best chunk-7 render candidates.

## Most Important Result

The best current chunk-7 render by temporal-motion overlap is:

- dynamic bank `2 -> 0`
- progress `109`
- render bbox `(0,0) -> (29,60)`
- overlap with capture motion mask: `27` pixels
- motion Jaccard: `0.003586`

The next few winners are the neighboring dynamic states at progress `108..110`, and they are all just as weak spatially.

This is the strongest current evidence that:

- the aligned title/options captures are not spatially explained by the current chunk-7 render model
- the earlier full-frame RGB rankings were being dominated by the static title art and menu text, not by a validated pointfield match

## Interpretation

This does not mean the chunk-7 interpretation is wrong in principle.

The executable-side evidence for chunk 7 as a morphing frontend record bank is still strong.

What it does mean is that the current reconstruction is missing at least one important piece of frontend presentation behavior, such as:

- a different startup state than the one seeded by our current `0x5ef4` model
- menu-specific masking or clipping
- a different screen region being compared in the captures
- interaction between the pointfield and the menu text/highlight layers that the current renderer does not model

There is also a concrete reason to be cautious with the motion mask itself:

- `0x3fd8` continuously redraws a highlighted menu row through `0x60cc`
- both `0x3df4` and `0x3ee4` redraw the full seven-row menu text block every frame during transitions

So the capture-side motion footprint may be dominated by menu text and highlight pulsing, not by the chunk-7 pointfield alone.

## Recommended Next Move

The next best move is no longer “keep ranking still frames harder.”

The best move is to keep following the executable-side frontend state path and resolve what the title/options handlers do around the pointfield before and after `0x39c4`, especially:

- when and how the frontend state is reinitialized
- whether the title/options path applies menu-specific masking or region clipping
- how `0x3fd8` and the `0x60cc` text path interact with the visible zero-valued title-screen regions

At this point, the motion evidence says the remaining gap is in the model, not just the scoring.
