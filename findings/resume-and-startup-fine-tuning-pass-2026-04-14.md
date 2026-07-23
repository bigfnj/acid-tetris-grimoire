# Resume And Startup Fine-Tuning Pass

Date: 2026-04-14

## Summary

This pass tightens five neighboring fidelity seams together:

- resume-frame transient cleanup
- splash-to-title handoff timing
- early sound-setup return visibility
- resume input carry behavior
- a field-by-field `New Game` vs `Return to Game` preservation matrix

The biggest practical result is that the remaining ambiguity is now smaller and more local than before.

- the first resumed gameplay frame can legitimately include transient-pixel cleanup before any new live gameplay work
- the splash screens do not crossfade directly into the title/menu presentation
- early startup sound-setup return now reads as effectively black, not just "blank-ish"
- carried input behavior now splits more cleanly between `New Game` and `Return to Game`

## 1. Resume-Frame Transient Cleanup Is Real And Comes First

The outer gameplay loop begins at `0x048f` with:

1. `0x17875`
2. inherited catch-up-step count setup
3. first-step `Esc` release check
4. live gameplay steps through `0x09c8`
5. particle draw through `0x2f24`
6. dirty flush through `0x17719`
7. present pacing through `0x24d0`

That ordering matters.

`0x17875` is not a late cleanup helper.
It runs before:

- the first resumed gameplay logic step
- the first resumed `Esc` release check
- the first resumed gameplay-side present

So if the tracked transient restore queue is non-empty when the frontend returns, the first resumed frame can visibly include that cleanup.

The queue ownership is also stronger now.

A direct flat-binary reference scan shows that:

- `0x17821` appends restore records at `0x1ad8b`
- `0x17875` restores those records and clears `0x1ad8b`
- no other current references to `0x1ad8b`, `0x201f7`, `0x201fb`, or `0x201ff` were found

That means there is no current evidence that menu entry, dispatcher exit, or resume sanitizes this queue separately.

So the best current fidelity model is:

- transient tracked pixels survive menu entry if they were already queued
- the first resumed gameplay frame restores them before new gameplay work lands
- those restored bytes are marked dirty and can be flushed in that first resumed frame

This is a small detail, but it is exactly the kind of small detail that affects whether the port feels like the DOS original.

## 2. The Splash-To-Title Handoff Is Sequential, Not A Direct Crossfade

`0x2998` is now tight enough to describe as a complete splash presenter:

- black palette upload first
- chunk-local palette fade-in
- held full-bright screen
- chunk-local fade-out
- full VRAM clear at the end

That last point matters here.

After `0x2998(0)` and `0x2998(2)`, startup does **not** immediately hand the visible image over to the title/menu system.
Instead, it continues through non-trivial resource work:

1. load chunk `6`
2. load chunk `1`, including the gameplay palette into `0x2c303`
3. load chunk `3`
4. load chunk `8`
5. start the selected music track through `0x6544`
6. only then enter `0x3830(1)`

And `0x3830(1)` itself then does:

1. save the current working screen into `0x2c69f`
2. run `0x64f0(0x2c303)`
3. copy the preloaded title/logo base from `0x2c6af` into the working and visible pages
4. run `0x63b8(0x2cdd3)` for the chunk-7 object fade-in
5. only afterward dispatch into main menu state `1`, where `0x3df4` reveals the eight rows

So the cold-boot visual rhythm is best modeled as:

1. splash `0`
2. splash `2`
3. black/cleared post-splash stage while startup loads gameplay-side resources
4. music starts
5. title/logo base appears with floating chunk-7 objects fading in
6. eight main-menu rows reveal afterward

That is more accurate than either of the earlier simpler mental models:

- "splash goes straight into title"
- "title is already fully formed when music starts"

Neither is quite right.

## 3. Early Sound-Setup Return Visibility Is Now Stronger Than "Probably Blank"

The early sound-setup detour happens after:

- buffer allocation through `0x0770`
- chunk-7 preload through `0x5fd8`
- title/logo base preload through `0x5f78`
- `SETUP.DAT` load/seed through `0x36dc`

But it happens before:

- concrete audio backend init through `0x668c`
- SFX slot loading
- both splash presenters
- gameplay chunk `1` load
- gameplay palette load into `0x2c303`

The important dispatcher detail is this:

- `0x3830` entry always snapshots the current working screen `0x2c727` into `0x2c69f`
- at this early point, `0x2c727` is still the zero-filled startup working buffer
- state `2` exit restores `0x2c69f` back into `0x2c727` and the visible pages
- state `2` exit then runs `0x6498(0x2c303)` and `0x2574(0x2c303)`

And `0x2c303` is not populated until later startup code loads chunk `1`, well after the sound-setup detour returns.

So the stronger current reading is:

- early sound-setup `Play Game` / `Esc` does **not** return to the title screen
- it restores a zero-filled pre-resource gameplay snapshot
- it reveals that screen through a gameplay-palette path whose real palette has not been loaded yet
- the practical result is effectively black or near-black until startup continuation resumes

That is now stronger than the earlier cautious wording.
The caution that remains is not about the control flow or buffers.
It is only about whether a specific monitor capture would show "pure black" versus "black with trivial DAC residue."

For faithful porting purposes, the correct model is:

- early sound-setup return -> black prelude screen -> startup continues

## 4. Resume Input Carry Matrix Is Now Cleaner

The stable input rule remains:

- frontend exit clears only the release latch at `0x2c227`
- frontend exit does not clear the live pressed-state table at `0x2c22b`

But the consequences are different for `New Game` and `Return to Game`.

### Discrete Menu Events

These do **not** carry as events:

- `Enter`
- released `Up`
- released `Down`

Why:

- menus consume them through `0x964`
- the keyboard ISR clears the live pressed bit on release
- the dispatcher tail clears the release latch again

### `Esc`

`Esc` is still special:

- main-menu fast resume checks live pressed state
- gameplay-side menu entry watches released `Esc` through the latch

So leaving the frontend with `Esc` held does **not** immediately bounce back into the menu.

That stays true for both:

- `Return to Game`
- the early sound-setup detour

### Physically Held Gameplay Keys

These can carry if they are still physically down when the frontend exits:

- `Left`
- `Right`
- `Down`
- rotate left `A`
- rotate right `S`

But the first-frame behavior differs by path.

#### On `New Game`

`0x05e0` calls `0x09c8(EAX = 1)`, which resets:

- left repeat timer
- right repeat timer
- rotate-left repeat timer
- rotate-right repeat timer
- Down lockout

and `0x1348` resets:

- piece X / Y
- rotation
- gravity accumulator

So the carried-input result is:

- held `Left` / `Right` / `A` / `S` can act immediately on the first live gameplay step
- held `Down` is seen, but normally cannot cause a first-step row descent from a fresh spawn because the gravity accumulator starts at `0` and the held-Down increment is only `0x8000` against a `0x10000` threshold

#### On `Return to Game`

State `2` does **not** call `0x05e0`.
So resume preserves:

- left/right/rotate repeat timers
- Down lockout
- gravity accumulator
- current piece position and rotation

That means the first resumed gameplay step can react in several different ways depending on the preserved live state:

- held `Left` / `Right` / rotation may act immediately or may still be waiting on a preserved repeat timer
- held `Down` may do nothing, may soft-drop normally, or may immediately reach a descent threshold if the preserved accumulator was already near `0x10000`
- if the Down lockout is still active, held `Down` can remain suppressed across the resume

That split is very important for a faithful port.

`New Game` should feel like a clean-but-not-input-sanitized fresh spawn.
`Return to Game` should feel like the exact live run kept breathing while the menu overlaid it.

## 5. The Preservation Matrix Is Now Concrete Enough To Record

I wrote a machine-readable matrix here:

- [preservation-matrix-newgame-vs-resume.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/preservation-matrix-newgame-vs-resume.json)

The most important practical rows are:

- screen snapshot buffer `0x2c69f`
- working screen / visible pages
- dirty-region map
- tracked transient restore queue
- logical board
- current/next piece state
- gravity accumulator and gravity rate
- repeat timers and Down lockout
- score / lines / level
- release latch vs live pressed state

In short form:

- startup sound-setup return:
  restores the zero-filled pre-resource snapshot and returns to startup continuation
- `New Game`:
  restores the prior snapshot, then deliberately resets gameplay run state and stages a fresh bootstrap
- `Return to Game`:
  restores the prior snapshot and preserves live gameplay transient state

That matrix should save us time later when we begin the C++23 port, because it isolates exactly what must be reset, preserved, or treated as startup-only.

## Porting Impact

This pass gives the future source port a sharper set of fidelity rules:

- do not model splash-to-title as one generic fade
- preserve the black/near-black early setup return rather than snapping back to the title
- preserve transient tracked-pixel cleanup at the start of resumed gameplay frames
- keep `New Game` and `Return to Game` semantically separate in state reset behavior
- preserve carried live pressed-state behavior, but do not preserve stale release-latch events

## Bottom Line

The remaining seams are getting smaller and more trustworthy.

We now have stronger evidence that:

- resume can visibly begin with transient-pixel restoration
- splash and title/menu presentation are sequential stages with a real load gap between them
- early sound setup returns to an effectively black prelude, not the title backdrop
- input carry is path-dependent in a way that matters for feel, not just correctness

That is the kind of fine-grain truth we want before we start the real port.

## 5 Next Strongest Moves

1. Tighten the exact first visible gameplay-present delta on `Return to Game`, especially whether the resumed frame more often shows only transient cleanup or also a live movement/alert change.
2. Resolve whether any menu-entry path deliberately snapshots over active transient particles, or whether the saved gameplay image always excludes those frame-local overlays.
3. Tighten the exact music-state continuity across `New Game`, `Return to Game`, and post-game-over handoff, so the port can preserve the original soundtrack behavior without guessing.
4. Resolve the remaining small gameplay presentation helpers around HUD redraw and alert-tile dirty marking, so the first visible gameplay frame can be reproduced more deterministically.
5. Start turning the now-stable reset/preserve rules into a formal C++23 preservation spec for scene transitions, while staying in reverse-engineering mode for any helper that still affects first-frame feel.
