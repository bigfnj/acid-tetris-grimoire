# Frontend Secondary State Pass

Date: 2026-04-14

## Summary

This pass tightens four frontend-side handlers that matter for a faithful port:

- `0x49bc` keyboard setup
- `0x5a94` first-run sound setup
- `0x4f38` credits sequence
- `0x50b0` high-score display and qualification/name entry

The practical result is that these screens are no longer just identified by strings and dispatcher slots.
We can now describe their phase structure, input model, and transition behavior closely enough to recreate them as named frontend state machines in the future source port.

## `0x49bc` Keyboard Setup Uses The Same Menu Loop Family

The keyboard setup screen is a true sibling of the main and options menus rather than a special one-off flow.

Confirmed structure:

1. format the six visible rows from the current binding bytes
2. run `0x3df4` entry transition
3. clear previously drawn chunk-7 points with `0x17613`
4. process one or more fixed logic steps using the catch-up count at `0x184cb`
5. redraw chunk-7 points through `0x175c5`
6. flush dirty cells through `0x17719`
7. present through `0x24d0`
8. if exiting, run `0x3ee4` and return the staged frontend state

Resolved local roles:

- `EBP`
  selected row `0..5`
- `[esp + 0x8]`
  action mode
- `[esp]`
  exit-ready flag
- `[esp + 0x10]`
  return frontend state

Action-mode meanings:

- `0`
  idle
- `1`
  pending exit
- `2`
  pending move up
- `3`
  pending move down
- `4`
  waiting for a replacement key

Important input behavior:

- `Up` and `Down` use the release latch from `0x964`
- row changes are gated by `0x3fd8(selected_row, 1)` exactly like the main and options menus
- `Esc` stages return state `1` and exits through the same gated path
- `Enter` on rows `0..4` enters action mode `4`
- `Enter` on row `5` stages return state `6` and exits back to options

The capture path is now clear:

- the selected row band is cleared through `0x61c8`
- an underscore prompt variant such as `Down:_` is written in place
- the code waits for one byte from `0x964`
- values `0` and `0x01` are ignored
- accepted bytes are written back to:
  - `0x2c73b` down
  - `0x2c73c` left
  - `0x2c73d` right
  - `0x2c73e` rotate left
  - `0x2c73f` rotate right
- the visible row is immediately reformatted with the selected key name from the table at `0x18c8b`

Porting implication:

- keyboard setup should be modeled as the same fixed-step animated menu family as the main/options screens
- rebinding is not a separate modal overlay
- the active row pulses continuously while idle and uses the one-shot `0x3fd8` gate for row changes and exit

## `0x5a94` Sound Setup Is Also A Full Frontend Menu Loop

The first-run sound setup screen is built on the same presentation scaffold as the rest of the frontend.

Confirmed structure:

1. copy the seven-entry mixing-rate table from `0x3570` into local storage
2. resolve the current mixing-rate index from the stored value at `0x2c70f`
3. format all six visible rows
4. run `0x3df4`
5. enter the same clear -> catch-up -> redraw -> flush -> present loop
6. run `0x3ee4` when exit-ready becomes true

Resolved local roles:

- `EBP`
  selected row `0..5`
- `[esp + 0x24]`
  action mode
- `[esp + 0x20]`
  exit-ready flag
- `[esp + 0x28]`
  return frontend state
- `[esp + 0x1c]`
  selected mixing-rate-table index

Row semantics:

- row `0`
  cycle sound device index at `0x2c6d7` through the names at `0x18878`
- row `1`
  cycle mixing rate through the copied seven-entry table and store the chosen value back to `0x2c70f`
- row `2`
  toggle stereo/mono at `0x2c723`
- row `3`
  toggle 16-bit/8-bit at `0x2c6e3`
- row `4`
  stage return state `2` `Play Game`
- row `5`
  stage return state `3` `Exit to Dos`

Input behavior matches the other menu-family handlers:

- `Up` and `Down` use the release latch and are gated by `0x3fd8(selected_row, 1)`
- `Esc` stages return state `2` and exits through the same animated path
- after each logic burst the handler redraws the chunk-7 points once, flushes once, and presents once

Porting implication:

- this screen should not be treated as a setup wizard separate from the menu framework
- it is the same animated frontend system with different row-local side effects

## `0x4f38` Credits Is A Three-Phase Credits Pager

The credits handler is no longer just "show some text pages."
It is a small pager state machine with automatic page timing and an interrupt path.

Resolved local roles:

- `[esp + 0x4]`
  phase
- `[esp]`
  current credits page index
- `EBX`
  per-page timer
- `[esp + 0x8]`
  staged return frontend state
- `[esp + 0xc]`
  exit-ready flag
- `[esp + 0x10]`
  input-gate flag

Phases:

- phase `1`
  render the current eight-line page from `0x1918b + page * 0x140`, then run `0x3df4`
- phase `0`
  steady display loop for the current page
- phase `2`
  run `0x3ee4`, advance the page index, wrap after page `5`, then return to phase `0`

Important timing behavior:

- the steady page display counts up in `EBX`
- when the timer reaches `0x12c`, the handler stages phase `2`
- that gives each credits page a fixed hold period before the next page transition begins

Interrupt behavior:

- `Esc` checks the live key-state table at `0x2c22b + 1`
- that path stages return state `1`
- the handler then exits through the same outer draw/flush/present rhythm and returns to the main menu

Presentation behavior:

- like the other frontend handlers, credits clears the prior pointfield pixels, advances `0x39c4` for one or more catch-up steps, redraws the active points through `0x175c5`, flushes through `0x17719`, and presents through `0x24d0`

Porting implication:

- credits should be implemented as an auto-advancing paged sequence with a page timer and an interruptable exit path
- it should reuse the shared frontend animation/update loop instead of a bespoke full-screen redraw

## `0x50b0` High Scores Has Distinct Display, Entry, And Exit Phases

The high-score handler is richer than the earlier state-map note suggested.
It has:

- insertion/qualification logic
- a 48-frame reveal phase
- a live name-entry phase with animated row highlight
- a footer-exit confirmation loop
- a 48-frame conceal phase

### Qualification And Insert Logic

When called with `EAX = 1`:

- compare the current run score at `0x2c6cb` against the five saved scores at `0x2c623`
- if no slot qualifies, fall back to plain display mode
- otherwise shift lower entries down when needed
- clear the inserted name field
- copy the current run score into the record score field
- copy total cleared lines from `0x2c72f` into the record secondary field

That confirms the post-game path preserves score and total lines directly into the stored table before the player types a name.

### Shared 48-Frame Reveal

Before either plain display mode or name-entry mode settles, the handler runs a custom 48-frame reveal loop.

Each logical step:

- redraws the table columns through `0x60cc`
- clamps the reveal amount to `0x30`
- advances `0x39c4`
- redraws the chunk-7 pointfield once
- flushes dirty cells once
- presents once

This is structurally similar to the menu-entry transition, but customized for the three-column high-score table and footer text rather than reusing `0x3df4` directly.

### Name Entry Mode

When a score qualifies:

- the active row index is the inserted table slot
- the typed name is stored in a temporary stack buffer
- visible length is capped at `10`
- every logic step clears the active name band, redraws the current buffer, advances `0x39c4`, and animates the selected row through `0x5868`

Resolved name-entry behavior:

- `Enter` stages a commit action
- `Backspace` is scan code `0x0e`
- scan code `0x3a` toggles the local case/shift selection flag
- the current modifiers at `0x2c22b + 0x2a` and `0x2c22b + 0x36` steer which character-translation table is used
- the translation tables are:
  - `0x18acb`
  - `0x18bcb`
  - `0x18c0b`
  - `0x18c4b`

The row highlight helper is now more precise in context:

- `0x5868(row, 0)` is the continuous animated redraw while the player is still typing
- `0x5868(row, 1)` is the one-shot completion gate when the name is being committed

### Footer Exit Loop And Conceal

After the row-commit gate completes:

- the handler enters a separate footer-exit loop
- that loop uses row `6` with `0x3fd8` exactly like the menu-family handlers
- either `Esc` live-state or `Enter` release can stage the final exit
- once exit-ready, the handler runs a 48-frame conceal loop that mirrors the earlier reveal with `max(0x2f - frame, 0)`

So the high-score screen is not just:

- show table
- accept characters
- return

It is a full presentation state with its own reveal, live entry, footer confirmation, and conceal phases.

## Porting Impact

This pass makes the frontend source-port plan more concrete:

- keyboard setup and sound setup should share the same frontend loop abstraction as the main and options menus
- credits should be modeled as an auto-paged state with timer-driven transitions and interrupt handling
- high scores should preserve the original reveal/entry/footer/conceal rhythm rather than being reduced to a static dialog

## Recommended Next Move

The strongest adjacent target now is the frontend dispatcher context around `0x3830` and the return values from:

- `0x4f38`
- `0x50b0`
- `0x49bc`
- `0x5a94`

That should let us write a tighter state-transition table for the source port instead of only naming individual handlers.
