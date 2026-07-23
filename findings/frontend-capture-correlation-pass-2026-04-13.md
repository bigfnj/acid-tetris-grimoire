# Frontend Capture Correlation Pass

Date: 2026-04-13

## Summary

This pass aligned the captured frontend screenshots back to the recovered `320x240` title screen and then scored those aligned captures against the chunk-7 title-overlay renders.

The new script is:

- [correlate_frontend_captures.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/correlate_frontend_captures.py)

The generated artifacts are:

- [frontend-alignment manifest](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-alignment/manifest.json)
- aligned captures under [frontend-alignment](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-alignment)

This pass used the title-screen asset as the reference base for these captures:

- `01-title`
- `02-options`
- `03-keyboardsetup`
- `04-hiscores`
- `05-credits`
- `05.1-credits`
- `05.2-credits`

## Main Result

The captures do **not** prefer the wide steady banks `5..7`.

They also do not prefer the compact steady banks as strongly as the first quick pass suggested.

After rerunning the correlation sequentially against the full updated render set, the captures most often prefer **mid-morph title-overlay renders**, especially:

- bank `3` at progress `64`
- bank `2` at progress `32`

## Strongest Match Pattern

Best-ranked render by capture:

- `01-title` -> bank `3`, progress `64`
- `02-options` -> bank `2`, progress `32`
- `03-keyboardsetup` -> bank `3`, progress `64`
- `04-hiscores` -> bank `3`, progress `64`
- `05-credits` -> bank `3`, progress `64`
- `05.1-credits` -> bank `3`, progress `64`
- `05.2-credits` -> bank `3`, progress `64`

That is a very consistent pattern.

## What This Probably Means

The frontend captures appear to line up better with:

- the compact bank family
- during an active morph interval
- rather than a fully settled steady-state bank image

The most common winner, bank `3` at progress `64`, is exactly the midpoint of the `0x80`-step interpolation interval.

That does **not** prove that every capture was taken at the exact midpoint of a real transition.

But it does strongly suggest that:

- the real frontend look in the captures is better modeled by the bank-morph system than by any single steady bank alone
- the compact `0..4` family is the visually relevant family for these screens
- the wider `5..7` family is a poor fit for the captured menu and credits appearance

## Practical Decompilation Impact

This is a useful preservation result.

For the future source port, it suggests that the frontend effect should not be implemented as:

- one fixed decorative point bank

It should be implemented as:

- a banked pointfield
- with the live interpolation behavior preserved
- and with the compact-family look treated as the most capture-consistent reference

## Important Caution

This pass is still a heuristic correlation, not absolute proof.

Reasons to stay cautious:

- the source screenshots are desktop-window captures, not direct raw frame grabs
- alignment is inferred automatically from the title-screen asset
- menu text and other UI overlays contribute a constant error term across all chunk-7 candidate renders

Even with those caveats, the ranking stability across seven separate captures is strong enough to take seriously.

## Recommended Next Move

The best next move is to extend the renderer from single-frame stills into short deterministic frame sequences for:

- bank `3` progress `32`
- bank `3` progress `64`
- bank `3` progress `96`
- bank `2` progress `32`

Then we can compare those sequences against the captured menu video instead of only against still screenshots.
