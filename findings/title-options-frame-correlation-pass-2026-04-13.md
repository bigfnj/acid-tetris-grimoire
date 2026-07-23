# Title/Options Frame Correlation Pass

Date: 2026-04-13

## Summary

This pass reused the frontend-correlation pipeline against the extracted title/options video frames rather than the desktop still screenshots.

The generated artifacts are:

- [title-options alignment manifest](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-alignment/manifest.json)
- aligned frames under [title-options-alignment](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-alignment)

Input frames analyzed:

- `title-options-01`
- `title-options-02`
- `title-options-03`
- `title-options-t15`
- `title-options-t60`
- `title-options-t135`

## Strongest Result

All six title/options video frames prefer the same chunk-7 render family:

- bank `3`
- progress `64`
- title-overlay render

Second place is consistently:

- bank `2`
- progress `32`

And the best steady-state fallback is consistently:

- steady bank `2`

That is a very stable ranking pattern across multiple frames taken from different times in the same frontend video.

## Fine-Grained Sweep Confirmation

After the initial pass, the chunk-7 renderer was extended with an `8`-step progress sweep for:

- bank `2 -> 0`
- bank `3 -> 0`

That finer sweep did **not** dislodge the earlier winner.

For all six title/options frames, the best sweep result remained:

- bank `3`
- progress `64`

So the earlier `64` result was not just a coarse-step artifact from only checking `32/64/96`.

## What This Adds Beyond The Still-Screenshot Pass

The earlier still-screenshot correlation already suggested that the captures aligned better with mid-morph chunk-7 states than with a settled steady bank.

This video-frame pass strengthens that conclusion because:

- the source frames are more uniform than the desktop-window screenshots
- the alignment scores are lower and cleaner
- the same best match repeats across the whole title/options frame set

So the frontend-animation interpretation is no longer only "plausible."

It is now supported by both:

- still screenshot correlation
- video-frame correlation

## Practical Interpretation

The captured frontend look appears to be best approximated by:

- the compact bank family
- during a morph interval
- near the middle of that transition

The exact reason bank `3` progress `64` keeps winning is still open.

It might mean:

- the real captured frontend was genuinely near that state
- or that bank `3` progress `64` is simply the most representative compact-family midpoint under the current projection model

Either way, the evidence keeps pointing away from:

- the wide `5..7` family
- any literal bitmap-background interpretation
- a purely static frontend decoration

## Recommended Next Move

The best next move is to generate a short deterministic frame sequence around:

- bank `3` progress `32`
- bank `3` progress `64`
- bank `3` progress `96`

Then compare the sequence evolution, not just isolated frames, against the title/options video timeline.
