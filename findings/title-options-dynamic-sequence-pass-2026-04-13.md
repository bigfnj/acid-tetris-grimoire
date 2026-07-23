# Title/Options Dynamic Sequence Pass

Date: 2026-04-13

## Summary

This pass extended the chunk-7 renderer with a real per-frame state update model and then reran the title/options frame correlation with two views:

- raw best matches
- visible-pointfield-only best matches

The key script changes were:

- [render_chunk7_pointfield.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/render_chunk7_pointfield.py)
- [correlate_frontend_captures.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/correlate_frontend_captures.py)

The updated title/options alignment results are in:

- [title-options alignment manifest](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-alignment/manifest.json)

## Most Important Correction

Once the dynamic sequence renders were added, the raw best matches changed completely.

The raw winners are now:

- dynamic bank `2 -> 0`
- progress `0..4`
- with `drawn_count = 0`

Those frames are effectively just the base title screen, because the current dynamic pointfield model has all projected points offscreen there.

So the raw best-match result is not a strong positive signal for the pointfield itself.

It is mostly a signal that the recovered title screen is a good base match for the captured title/options frames.

## Why The Visible-Only Ranking Matters

To keep the comparison informative, the correlation script now also records:

- `top_visible_matches`

using a `drawn_count >= 64` threshold.

That filtered view removes the trivial "pure title screen" winners and surfaces the best candidates that still contain a visibly present pointfield.

## Visible-Only Result

For the six title/options frames, the visible-only best matches cluster tightly around:

- bank `3 -> 0`, progress `21..23`
- bank `2 -> 0`, progress `103`

More specifically:

- `title-options-01` -> bank `3`, progress `23`
- `title-options-02` -> bank `3`, progress `23`
- `title-options-03` -> bank `2`, progress `103`
- `title-options-t135` -> bank `3`, progress `23`
- `title-options-t15` -> bank `3`, progress `23`
- `title-options-t60` -> bank `2`, progress `103`

So the informative dynamic candidates are now much narrower than the earlier static-frame pass suggested.

## What This Likely Means

This result suggests two things at once:

1. The title/options captures are dominated visually by the static title-screen base layer.
2. When we force the comparison to consider only renders with a visibly present pointfield, the best dynamic candidates come from a narrow band of the sequence rather than from arbitrary progress values.

That is useful, but it also means the earlier "bank 3 progress 64" interpretation is no longer the whole story.

It remains a good still-frame heuristic when we ignore the degenerate base-title frames.

But once the dynamic sequence and visibility filtering are included, the evidence becomes:

- raw capture match: base-title-like frames win
- informative pointfield match: narrow dynamic windows win

## Important Caveat

The current dynamic sequence model drives many frames into tiny top-left visible clusters or fully offscreen states.

That may mean one of two things:

- the model is still incomplete or slightly wrong
- or the captured title/options frames genuinely do not show much of the pointfield effect

At this stage, we should treat this as a useful narrowing result, not final proof.

## Recommended Next Move

The best next move is to inspect the menu-side frame pump and any state-reset or pre-draw helpers around the frontend loop to see whether the title/options path is reinitializing, clipping, or masking the pointfield before `0x39c4` runs.
