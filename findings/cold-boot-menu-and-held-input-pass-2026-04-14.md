# Cold-Boot Menu And Held-Input Pass

Date: 2026-04-14

## Summary

This pass closes two related fidelity questions in one place:

- what the player actually sees during the first cold-boot main-menu presentation
- which input states can carry from the frontend into the first gameplay frame after `New Game`

The main result is that the original does **not** jump straight from startup splashes into a fully formed menu.
It first reveals the title/logo backdrop and floating chunk-7 objects, then reveals the eight menu rows.

On the input side, the frontend clears only the release latch before leaving.
It does **not** clear the live pressed-state table.
So the first gameplay frame after `New Game` can see keys that are still physically held down, but it does **not** inherit the release events that drove the menu itself.

## Cold-Boot Main Menu Presentation Is Two-Stage

The startup order is already established:

1. optional early sound setup through `0x3830(5)`
2. load SFX
3. show splash `0`
4. show splash `2`
5. load gameplay-side chunks
6. start the selected music track through `0x6544`
7. enter `0x3830(1)` for the normal cold-start main-menu path

The new useful part is the exact visible structure of that state-`1` entry.

## What `0x3830(1)` Shows Before The Main Menu Rows Exist

At entry, `0x3830` does this before it ever dispatches to `0x40c0`:

1. save the current gameplay-side working screen into `0x2c69f`
2. run the simple palette fade-out through `0x64f0`
3. copy the preloaded title/logo base screen from `0x2c6af` into the working and visible pages
4. run the frontend fade-in through `0x63b8` using palette `0x2cdd3`

That means the first cold-start frontend phase is:

- the ACiD Tetris title/logo base already present on the visible pages
- the chunk-7 floating menu-object system fading in over it
- no menu rows yet

Because startup already called `0x6544` before `0x3830(1)`, music is already running during this first title/logo presentation phase.

## What `0x40c0` Adds After The Backdrop Is Already Visible

Only after `0x63b8` finishes does `0x3830` dispatch state `1` into the main-menu handler `0x40c0`.

`0x40c0` then:

- formats the eight main-menu rows
- calls `0x3df4`

`0x3df4` is the menu-entry text transition, not the title/logo reveal.
It redraws the eight rows with the `0x60cc` reveal parameter increasing up to `0x30`, which means the rows vertically reveal/resample into place across the already visible title/logo plus chunk-7 backdrop.

So the cold-boot menu presentation is best modeled as:

1. title/logo backdrop copied in
2. floating frontend objects fade in
3. eight menu rows reveal over that backdrop
4. steady-state selected-row pulse begins afterward

## Initial Selected Row Behavior

`0x5f78` seeds:

- `0x1886f = 0`

So the first main-menu selection is row `0`:

- `New Game`

That row is not yet doing its steady pulse during the `0x3df4` reveal itself.
The continuous selected-row pulse comes later from `0x3fd8` inside the steady loop once `0x3df4` has already completed.

That is a small but useful fidelity detail:

- first the rows appear
- then the selected row settles into its steady squash/stretch pulse

## Main Menu Input Is Mixed, Not Purely One Model

The main-menu loop in `0x40c0` uses two different input paths:

- release latch via `0x964` for:
  - `Enter`
  - `Up`
  - `Down`
- live pressed-state table at `0x2c22b` for:
  - `Esc`

And even `Esc` is special-cased:

- it only exits through state `2` when the live-game flag `0x2c72b` is set

So on a normal cold-boot main menu:

- `Enter`, `Up`, and `Down` are release-driven
- `Esc` does not act like a universal back/quit key

That is more specific than the earlier simplified "menus are release-driven" summary.

## What Gets Cleared Before Leaving The Frontend

When `0x3830` breaks out of the frontend loop, its shared tail starts with:

- `0x094c`

`0x094c` clears only the `0x20`-byte release latch at `0x2c227`.
It does **not** clear the live pressed-state table at `0x2c22b`.

That means:

- the menu-triggering release events are discarded
- but any keys still physically held remain live

This is the key rule for the `New Game` handoff.

## What Can Carry Into The First Gameplay Frame After `New Game`

Because `0x05e0` resets gameplay repeat timers through `0x09c8(EAX = 1)` but does not clear the live pressed-state table, the first gameplay step can still see currently held gameplay-bound keys.

That produces a very specific split:

- menu activation by `Enter` does **not** carry into gameplay, because the menu sees `Enter` on release and the ISR clears the live pressed bit on release
- menu navigation by released `Up` or `Down` also does **not** carry by itself for the same reason
- keys that are still physically held when the frontend exits **can** carry into the first gameplay step

With the shipped default bindings, the practical overlap from the main-menu path is:

- held `Down`
- held `Left`
- held `Right`
- held `A`
- held `S`

But only `Down` overlaps with the main menu's own arrow-navigation family.

So the most realistic accidental carry from ordinary menu use is:

- the player keeps `Down` physically held while triggering `New Game`

## What That Carry Actually Does On The First Gameplay Step

This interacts with the already resolved startup timing in an important way.

The first gameplay step after `New Game`:

- is live
- can see currently held keys
- but normally does **not** gravity-drop the new piece yet

So a carried held key can:

- move left
- move right
- rotate left
- rotate right
- arm soft-drop behavior

But soft-drop still does not produce an immediate first-frame descent from a fresh spawn, because:

- the spawn gravity accumulator starts at `0`
- even held Down only contributes `0x8000`
- the descent threshold is `0x10000`

So the most faithful input-edge reading is:

- movement or rotation carry can visibly affect the very first gameplay present
- soft-drop carry usually cannot produce a row drop on that first present

## Porting Impact

For the future source port, the safe fidelity model is:

- cold boot should show:
  - music already playing
  - title/logo backdrop and floating objects first
  - menu-row reveal second
  - steady row pulse third
- frontend exit should clear release-latch state but not forcibly zero all pressed keys
- first gameplay-step carry-over should be based on physical held state, not stale menu release events

If we later decide to add a modernization option that sanitizes all held input on scene changes, that should be an explicit opt-in behavior, not the default faithful mode.

## Bottom Line

This pass sharpens both startup and input fidelity:

- the first cold-boot menu presentation is a backdrop-first, text-second sequence
- the first post-`New Game` gameplay frame can inherit currently held keys, but not the release events that drove the menu

That is exactly the kind of small sequencing truth that will help the port feel like the original game rather than just resemble it.
