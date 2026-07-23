# Frontend Menu Resolution Pass

This pass tightens the frontend-menu map using direct string extraction from the flat relocated executable, a raw disassembly recovery of the truncated `0x49bc` handler, and a focused pulse-correlation pass that isolates row-level highlight changes from the static title art.

## Files and Artifacts

- Scripts:
  - [render_frontend_menu_pulse.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/render_frontend_menu_pulse.py)
  - [correlate_frontend_menu_pulse.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/correlate_frontend_menu_pulse.py)
  - [correlate_frontend_menu_pulse_delta.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/correlate_frontend_menu_pulse_delta.py)
- Correlation outputs:
  - [frontend-menu-pulse manifest](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-menu-pulse/manifest.json)
  - [frontend-menu-pulse correlation](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-menu-pulse-correlation/manifest.json)
  - [frontend-menu-pulse delta](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-menu-pulse-delta/manifest.json)

## Main Menu Is Fully Resolved

The flat binary string table confirms that `0x40c0` formats the exact main-menu rows:

- `New Game`
- `Options`
- `Music:\`Continuum'`
- `Music:\`Tearing up Spacetime'`
- `Music:\`Inner Walls Released'`
- `Music:\`Costumed'`
- `Music:\`I See It Now'`
- `Music:\`Simple Song'`
- `Level %d`
- `High Scores`
- `Credits`
- `Exit Game`
- `Return to Game`

That makes `0x40c0` a very high-confidence `run_main_menu` handler rather than a generic frontend state.

## Options Menu Is Fully Resolved

`0x4584` formats these four rows:

- `Music Volume:%d`
- `Sound FX Volume:%d`
- `Keyboard Setup`
- `Back To Main Menu`

The handler directly edits the stored music and sound-FX volume values at `0x2c6eb` and `0x2c6e7`, clears only the affected row band through `0x61c8`, and rewrites the text in place. That matches the captured options screen and confirms that the options menu is not just a static screen redraw.

## Keyboard Setup Is Now Recovered From Raw Disassembly

The existing Ghidra export for `0x49bc` was truncated, so the loop was finished from raw disassembly of the flat relocated payload.

The handler formats these six rows:

- `Down:%s`
- `Left:%s`
- `Right:%s`
- `Rotate Left:%s`
- `Rotate Right:%s`
- `Back to Options Menu`

The first five rows are backed by binding bytes at:

- `0x2c73b` = Down
- `0x2c73c` = Left
- `0x2c73d` = Right
- `0x2c73e` = Rotate Left
- `0x2c73f` = Rotate Right

The `%s` substitution source is a fixed-width key-name table at `0x18c8b`, with `5`-byte entries like:

- `Esc `
- `BkSp`
- `Entr`
- `Tab `
- `CtLt`
- `ShLt`
- letter keys like ` Q  `, ` W  `, ` Z  `

The input side of this path is now resolved too:

- `0x94c` clears the current `0x20`-byte input latch at `0x2c227`
- `0x964` returns the single byte at offset `0x1f` from that same input-state block

That is the exact byte compared against `0x1c` for Enter, `0xc8/0x48` for Up, and `0xd0/0x50` for Down, and it is also the exact byte written back into the five key-binding slots during keyboard capture.

Behaviorally, `Enter` on rows `0..4` does not immediately return. Instead it:

1. clears the selected row band through `0x61c8`
2. rewrites the row as `Down:_`, `Left:_`, `Right:_`, `Rotate Left:_`, or `Rotate Right:_`
3. waits for one raw key code from `0x964`
4. stores that byte back into the selected binding slot
5. rebuilds the normal `%s` row text from the key-name table

Selecting row `5` sets exit state `6`, which returns control to the options menu.

## Sound Setup Loop Is Confirmed As A Sibling Frontend Menu

`0x5a94` is now firmly established as the first-run sound setup menu. Its rows are:

- `%s` for device name
- `Mixing Rate:%d`
- `Stereo` or `Mono`
- `16 Bit` or `8 Bit`
- `Play Game`
- `Exit to Dos`

Like the main menu, options menu, and keyboard setup menu, it uses:

- `0x3df4` for menu entry
- `0x39c4` for the steady frontend pointfield update
- `0x3fd8` for row highlight stepping / gating
- `0x3ee4` for menu exit

This makes the frontend architecture much cleaner than it first appeared: these menus are sibling state loops sharing the same presentation framework, not unrelated one-off screens.

## Pulse Correlation Result

The original full-frame pulse correlation was dominated by static title art and menu text, so a second pass scored only the pixels that change between each pulsed render and its own static menu baseline.

That focused pass still did **not** find support for the highlight pulse in the aligned captures.

For all six aligned `title-options` captures:

- the best non-degenerate match stayed on the `main-menu`
- the least-bad changed state was row `2` (`Level 0`) at phase `0x440`
- even that candidate was worse than the static baseline by `18.5` masked RGB points

That means the currently aligned capture set is better explained as a settled static main-menu presentation than as a visible selected-row pulse. The practical implication is:

- the capture files are still useful for menu layout confirmation
- but they should not currently be treated as evidence for the live `0x3fd8` highlight phase

## What This Changes For The Port

We now have enough frontend certainty to model the menu system as:

- shared entry / steady / exit animation framework
- per-menu row tables and action dispatch
- row-band redraw helpers rather than full-screen text redraw for every small change
- real editable keyboard bindings backed by five stored binding bytes

That is a stronger basis for a faithful port than the earlier “static title screen plus guessed menu overlay” model.
