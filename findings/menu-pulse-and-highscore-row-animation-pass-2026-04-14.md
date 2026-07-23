# Menu Pulse And High-Score Row Animation Pass

Date: 2026-04-14

## Summary

This pass resolves two visible-behavior helpers together:

- `0x3fd8`
- `0x5868`

The important result is that both helpers use the same broad animation idea:

- sample a sine-driven phase
- convert it into the reveal term expected by `0x60cc`
- redraw fixed-position text
- advance phase

But they are not identical.
The high-score row helper has its own phase variable and a slightly different completion behavior from the generic menu pulse gate.

## `0x3fd8` Is The Generic Menu Highlight Pulse

`0x3fd8` takes:

- `EAX = row index`
- `EDX = mode`

It redraws one row from the current menu string table:

- string base: `0x2d0d3 + row * 40`
- x center: `0xa0`
- y: `0x64 + row * 17`

It computes the reveal term from:

- sine table: `0x2c6cf`
- phase variable: `0x1990f`

Then it passes that reveal term into `0x60cc`.

So the menu highlight is not a special sprite or separate overlay.
It is a repeated text redraw of the selected row with a sine-driven reveal amount.

## `0x3fd8` Continuous And One-Shot Modes

### Continuous Mode

With `EDX = 0`, the helper:

- redraws the selected row
- advances the shared phase by `0x20`
- wraps phase with `& 0x7ff`
- returns `0`

This is the idle selected-row pulse used while a menu is just sitting there.

### One-Shot Gate Mode

With `EDX = 1`, the helper behaves differently:

- if phase is already at boundary `0` or `0x400`, it returns `1` immediately
- otherwise it draws the pulse frame, advances phase by `0x20`, then advances by an additional `0x20`, and returns `0`

So one-shot mode is not just "same pulse, but report done."
It is a slightly accelerated gate that callers can poll until the highlight reaches a clean phase boundary.

That explains why menu row changes and exits feel synchronized rather than abrupt.

## `0x5868` Is The High-Score Row Pulse

`0x5868` uses the same sine table idea, but it redraws a whole high-score row across three fixed text columns.

Per row:

- y: `0x64 + row * 17`
- name column center: `0x69`
- score column center: `0xdc`
- lines column center: `0x118`

The three row-string bases are:

- `0x2d0d3 + row * 40`
- `0x2cc93 + row * 40`
- `0x2cb53 + row * 40`

It uses:

- sine table: `0x2c6cf`
- phase variable: `0x19913`

That separate phase variable is important:
the high-score row effect is independent of the generic menu pulse timing.

## Important Correction: The High-Score Helper Does Not Move The Row

An earlier interpretation suggested animated row offsets.
This pass corrects that.

`0x5868` keeps the row at fixed x/y positions and re-renders the three text columns with a changing reveal term.

So the effect is:

- fixed-position pulsing text

not:

- a row sliding horizontally
- a row wobbling in screen position

That is a meaningful correction for the future port.

## `0x5868` Completion Behavior

Like `0x3fd8`, `0x5868` has:

- continuous mode
- one-shot gate mode

But the one-shot success case differs slightly.

If the high-score phase is already at boundary `0` or `0x400`, `0x5868`:

- draws one final stable boundary frame
- returns `1`

If not at boundary, it behaves like the generic gate:

- draw
- advance by `0x20`
- advance by another `0x20` in gate mode
- return `0`

That means the high-score commit gate is visually a little more explicit than the generic menu gate.
It gives the player one clean settled frame of the highlighted score row before the screen moves on.

## Why This Matters

These helpers are small, but they are exactly the sort of details that make a port feel faithful.

We now know:

- selected menu rows are pulsed by re-rendered text, not a separate cursor object
- row changes and exits are gated to clean phase boundaries
- the high-score row uses its own phase and redraws all three visible columns together
- the high-score commit gate intentionally lands on a stable final frame

## Porting Impact

For the C++23/SDL3 port, the faithful model should be:

- keep a phase-driven text pulse helper for generic menu rows
- keep a separate phase-driven helper for high-score row emphasis
- use the gate variants when changing rows or committing transitions
- preserve the "stable boundary frame before success" behavior on the high-score row path

We do not need to preserve the exact integer math instruction-for-instruction, but we do want the same timing and presentation semantics.

## Bottom Line

This pass resolves both helpers into something practical:

- `0x3fd8`
  generic menu-row pulse and gate
- `0x5868`
  three-column high-score row pulse and commit gate

That gives us a much sharper model for two pieces of frontend polish that the eventual port should absolutely keep.
