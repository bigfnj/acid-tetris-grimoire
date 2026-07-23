# Secondary Frontend Scaffold And Live Audio Pass - 2026-04-14

This pass stayed inside the secondary frontend menu family and tightened a set of fidelity details that matter more than they look at first:

- the visual row scaffold used by options, keyboard setup, and sound setup
- the difference between release-gated row movement and held-state value editing
- the asymmetric live-audio behavior in the options screen

## New Owned Artifacts

- [raw-4584-49bb.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/frontend-menu-primitive-pass/raw-4584-49bb.asm)
- updated [frontend-menu-primitive-pass.index.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/frontend-menu-primitive-pass/frontend-menu-primitive-pass.index.json)
- updated [frontend-mutable-row-behavior.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/frontend-mutable-row-behavior.json)
- updated [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- updated [transition-preservation-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/transition-preservation-spec.md)

## Main Result

The secondary frontend states do not just share the same broad loop shape as the main menu.
They also preserve the same eight-row reveal scaffold even when fewer rows are actually actionable.

Current best model:

- options:
  - `4` active rows
  - `4` blank padding rows
- keyboard setup:
  - `6` active rows
  - `2` blank padding rows
- sound setup:
  - `6` active rows
  - `2` blank padding rows

That means the future port should not rebuild these screens as smaller bespoke dialogs with their own reveal counts.
They sit on the same shared frontend row framework as the main menu.

## Options Uses A More Specific Input Model Than Our Earlier Summary

The new raw options artifact closes two useful behavior details.

### 1. Row movement and value editing are not the same kind of input

The options handler splits input by purpose:

- `Up` and `Down`
  - release-gated through `0x964`
  - one-shot gated through `0x3fd8`
- volume editing
  - live held-state checks from the key table at `0x2c22b`

So the menu selection behaves like a discrete animated menu, but the volume values behave like held controls.
That is a good fidelity detail to preserve.

### 2. Music and SFX volume are intentionally asymmetric

Row `0` `Music Volume`:

- clears only the row band through `0x61c8`
- clamps the stored value at `0x2c6eb` to `0..100`
- rewrites the visible string immediately
- calls `0x68bb`

That last helper is the important part.
The music-volume change is applied immediately to the already playing track.

Row `1` `Sound FX Volume`:

- clears only the row band through `0x61c8`
- clamps the stored value at `0x2c6e7` to `0..100`
- rewrites the visible string immediately
- does **not** call the live music-volume helper

So SFX volume changes are stored immediately, but their audible effect is indirect through later sound playback rather than a dedicated preview helper.

## Options Row Navigation Uses The Current SFX Volume

One more small but useful fidelity detail came out of the raw options loop.

When the player changes rows with `Up` or `Down`, the handler plays slot `1` through `0x6817` using the current SFX volume value at `0x2c6e7`.

That means:

- the navigation pulse is part of the normal options feel
- if the player edits SFX volume first, later row changes immediately reflect the new loudness

That is exactly the kind of detail a faithful port can preserve easily once it is written down.

## Keyboard Setup And Sound Setup Share The Same Eight-Row Scaffold

The earlier notes already established that keyboard setup and sound setup are menu-family siblings.
This pass tightens the visible scaffold rule.

Keyboard setup:

- fills six named rows
- pads rows `6` and `7` with blank strings
- still uses the same reveal/steady/exit family

Sound setup:

- fills six named rows
- pads rows `6` and `7` with blank strings
- still uses the same reveal/steady/exit family

So the blank rows are not accidental leftover buffer space.
They are part of the visible row system the frontend is built around.

## Why This Matters For The Port

If we rebuilt these screens as ordinary modern option panels, the result would probably work, but it would lose several parts of the original feel:

- the shared eight-row reveal rhythm
- the blank padding rows that preserve the reveal geometry
- the difference between discrete row moves and held value edits
- immediate live music-volume response
- indirect but immediate-in-effect SFX-volume changes through later UI sounds

The faithful port should preserve those rules even if the implementation underneath becomes much cleaner than the DOS original.

## 10 Strongest Next Moves

1. Do a similar edge-focused raw pass on the high-score and name-entry flow so prompt, blink, row pulse, and commit sequencing are as sharp as keyboard capture now is.
2. Tighten alert-tile lifetime edge cases further, especially expiry and immediate reset behavior across consecutive gameplay-owned frames.
3. Tighten tracked-particle edge cases where cleanup occurs but the subsequent draw set is empty or materially smaller than the prior frame.
4. Keep the `0x1765a` question open, but narrow it strictly to reachability confirmation.
5. Expand the owned raw artifact set around credits presentation so the secondary frontend family has the same direct raw coverage throughout.
6. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
7. Tighten whether any alert refresh paths can visibly coexist with tracked transient overwrites in the same `8x4` dirty cells.
8. Keep searching for indirect or computed dirty-map writes that could weaken or refine the current mark-`3` propagation model.
9. Continue converting these frontend and gameplay edge findings into direct implementation constraints for the future `C++23 + SDL3` port.
10. Refresh the root session log once the next edge-path cluster lands, because the frontend fidelity model is now noticeably tighter again.
