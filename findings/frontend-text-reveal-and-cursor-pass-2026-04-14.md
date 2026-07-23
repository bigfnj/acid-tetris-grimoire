# Frontend Text Reveal And Cursor Pass

Date: 2026-04-14

## Summary

This pass tightens the frontend text system around:

- `0x60cc`
- `0x61c8`
- `0x178b0`
- `0x3fd8`
- `0x5868`

The important result is that the menu and high-score text effects are now better understood as:

- vertical text reveal and resample behavior
- not horizontal motion
- not palette pulsing
- not a separate cursor sprite system

It also closes the underscore cursor behavior cleanly.

## What The Reveal Parameter Really Does

`0x60cc` passes its fifth argument down into `0x178b0` for every visible glyph.

`0x178b0` then uses that value as a vertical resample factor while drawing a `12`-row source glyph into a `16`-row destination band.

Current best model:

- source glyph height: `12`
- destination band height: `16`
- source row chosen per destination row:
  - `floor(dest_row * 48 / reveal)`

Rows that map outside `0..11` are written as zero.

So the reveal term is not a vague style token.
It directly controls how much of the glyph appears and how stretched it looks inside the fixed `16`-pixel menu band.

## Why Menu Entry And Exit Look The Way They Do

This now explains the transition helpers much more concretely.

### Entry `0x3df4`

The entry transition feeds an increasing value up to `0x30` into `0x60cc`.

That means the rows are not simply alpha-fading in.
They are being redrawn with a progressively taller visible text form inside the same band.

### Exit `0x3ee4`

The exit transition feeds:

- `max(0x2f - frame_index, 0)`

into `0x60cc`.

So the rows are progressively collapsing back down through the same reveal path.

## What The Selected-Row Pulse Really Is

`0x3fd8` computes:

- `reveal = (sine(phase) >> 11) + 0x30`

and passes that into `0x60cc`.

That means the selected-row effect is a text squash/stretch pulse.

It is not:

- a moving cursor
- a row offset
- a palette shimmer

The row stays fixed in place and is simply re-rendered with a changing vertical reveal scale.

## High-Score Row Pulse Uses The Same Mechanism

`0x5868` does the same basic thing for the highlighted high-score row, but with its own phase variable and three separate column strings.

So the qualification-row and high-score emphasis effect is also:

- fixed-position
- text re-rendered in place
- driven by the same reveal/resample idea

not a sliding-row effect.

## The Underscore Cursor Is Built Into The Text Renderer

`0x60cc` treats glyph code `0x42` specially.

That is the underscore glyph from the chunk-5 font mapping.

When bit `0x10` of the global tick counter at:

- `0x2d2a3`

is set, `0x60cc` skips drawing the underscore while still advancing the layout.

That means:

- the underscore blink is built into the text renderer
- the blink cadence is tied to the global engine tick
- there is no separate cursor sprite or overlay object for that prompt

This fits the keyboard-setup and name-entry prompt behavior much better than the earlier generic "underscore prompt" reading.

## Reveal Zero Means Clear The Whole String Band

When `0x60cc` is called with reveal `0`, it does not attempt partial glyph drawing.

Instead it:

- measures the full string width
- clears the corresponding `16`-pixel-tall band through `0x6274`
- marks the cleared area dirty through `0x2d30`

That makes the reveal path structurally clean:

- reveal `0`
  clear band only
- reveal `> 0`
  draw glyphs with vertical reveal/resample

This matches the explicit band-clear helper `0x61c8`, which uses the same layout measurement and clear geometry.

## Porting Impact

For a faithful port, we should preserve these behaviors conceptually:

- frontend strings draw into fixed `16`-pixel bands
- entry/exit transitions use reveal-scale text redraws, not alpha fades
- selected-row and high-score pulses are vertical text pulses
- underscore cursor blinking is part of text rendering and tied to the global frontend/game tick

We do not need to preserve the original integer math literally, but we do want the same visible effect.

## Bottom Line

This pass sharpens several visible frontend behaviors at once.

The menu and high-score text effects are now best modeled as:

- fixed-position text
- vertically revealed and pulsed by resampled glyph rendering
- with underscore blinking handled inside the same renderer

That is a much better fidelity target for the future port than a generic "draw some menu text and highlight the selected row" model.
