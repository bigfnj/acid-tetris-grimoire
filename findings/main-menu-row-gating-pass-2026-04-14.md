# Main Menu Row Gating Pass

Date: 2026-04-14

## Summary

This pass corrects and sharpens an important visible-behavior detail in the main menu.

The key results are:

- the main-menu transition helpers redraw `8` rows, not `7`
- `Return to Game` is always present as the visible eighth row
- that row is only actionable when the live-game flag is set

That is a better fit for both the executable and the captures.

## Main Menu Really Has Eight Visible Rows

`0x40c0` formats eight row buffers into the runtime menu table:

- `New Game`
- `Options`
- `Music:...`
- `Level %d`
- `High Scores`
- `Credits`
- `Exit Game`
- `Return to Game`

The earlier shorthand that described the title/menu transition as a seven-row redraw was too small.

Raw disassembly of `0x3df4` shows the row loop starts at:

- `y = 0x64`

and advances by:

- `0x11`

until it reaches:

- `0xdb`

That is an eight-row sequence, not seven.

So the entry and exit transition helpers are redrawing the full visible main menu.

## `Return to Game` Is Not Hidden

The main-menu row index at:

- `0x1886f`

is treated as a normal `0..7` selection throughout the loop.

The row-wrap logic also confirms that:

- moving up from row `0` wraps to row `7`
- moving down from row `7` wraps to row `0`

So the eighth row is not a hidden mode or conditional insertion.
It is part of the real visible menu structure.

## What Is Actually Gated

The gating happens at activation time, not draw time.

When `Enter` lands on row `7`, `0x40c0` checks:

- live-game flag `0x2c72b`

If that flag is `1`, activation stages dispatcher state `2` and exits through the normal gated menu path.

If that flag is not set, the row remains visible and selectable, but activation is ignored.

So the original menu behavior is:

- always show `Return to Game`
- only honor it when there is a resumable live game

## Why This Matters

This explains two things cleanly:

1. why the main menu still structurally behaves like an eight-row menu whenever the main-menu handler is entered
2. why captures can legitimately show `Return to Game` without implying that resume is always available

It also means a faithful port should not silently remove that row just because no live game is active.

## Porting Impact

For the future port, the main menu should preserve:

- eight visible rows
- normal selection wrap across all eight rows
- row `7` rendered and pulsed like the others
- activation gating on live-game state instead of row visibility

That is a more accurate model than:

- hiding the row when unavailable
- changing the row count between different frontend entry contexts

## Bottom Line

This pass corrects the menu model in a useful way.

The main menu is an eight-row visible menu, and `Return to Game` is a real rendered row whose action is conditionally enabled rather than conditionally drawn.
