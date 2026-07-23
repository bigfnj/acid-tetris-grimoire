## Frontend Menu Layering Pass - 2026-04-14

This pass gave the steady frontend menu loop the same low-level treatment we already gave gameplay.

The key question was:

- in a presented menu frame, what gets redrawn first?

The answer matters because it determines whether the floating chunk-7 objects should visually sit on top of menu text, behind it, or interleave in some more complicated way.

### New Owned Artifacts

- `research/ghidra/exports/decompilations/frontend-menu-primitive-pass/raw-3fd8-40bf.asm`
- `research/ghidra/exports/decompilations/frontend-menu-primitive-pass/raw-40c0-4583.asm`
- `research/ghidra/exports/decompilations/frontend-menu-primitive-pass/raw-60cc-625f.asm`
- `research/ghidra/exports/decompilations/frontend-menu-primitive-pass/frontend-menu-primitive-pass.index.json`
- `research/ghidra/frontend-menu-present-order.json`

### Main Result

The steady main-menu frame is now best modeled as:

1. clear previously drawn chunk-7 object pixels
2. process one or more catch-up logic steps
3. redraw text/value/highlight state during those logic steps
4. redraw the current chunk-7 object points once for the presented frame
5. flush dirty cells
6. present

That is already a useful fidelity improvement.

### Text And Highlight Win The Layering Fight

The strongest new rule from this pass is:

- menu text and highlight redraw have visual priority over the floating frontend objects

Why:

- `0x3fd8` redraws the highlighted row through `0x60cc`
- `0x60cc` and `0x61c8` rewrite text bands directly into the working screen during the steady loop
- the later object draw pass uses `0x175c5`
- `0x175c5` only plots into pixels that are still zero

So if text or a pulse redraw has already written non-zero pixels into the menu area, the floating object points do not overwrite them.

That means the original presentation is not "objects and text blended equally."
Text is a true foreground layer with respect to the object pixels.

### `0x3fd8`

This helper is now stronger in context than before.

It is not just a conceptual pulse.
In the steady menu loop it is a real redraw stage that happens before the object pass.

That means the selected-row pulse is part of the visible text layer and inherits the same foreground priority over chunk-7 object pixels.

### `0x60cc` And `0x61c8`

These helpers are also sharper now.

- `0x60cc` measures, centers if requested, optionally clears the row band when reveal is zero, then draws glyphs and marks dirty rectangles.
- `0x61c8` clears a layout-consistent 16-pixel-tall string band and marks it dirty.

In the menu handlers, these are used for real in-place row updates:

- music title changes
- level value changes
- options value changes
- keyboard binding redraw
- sound setup row refreshes

Those updates happen inside the steady loop before the later object redraw.

### `0x40c0` Steady Loop

The raw main-menu loop now makes the sequence quite concrete:

- clear prior object pixels
- run catch-up steps
- process input and staged state changes
- redraw menu text/highlight state
- advance the frontend object simulation through `0x39c4`
- after catch-up is exhausted, draw current object pixels once
- flush and present

That means the frontend object system is visually subordinate to the menu text layer during steady presentation, even though both are active in the same frame.

### Generalization

The strongest evidence is from the main menu loop itself, but the call patterns and already-mapped handlers support the same family model across:

- main menu
- options menu
- keyboard setup
- sound setup

So this is a good rule to preserve for the general frontend menu family, not just one screen.

### Why This Matters For The Port

If the future `C++23 + SDL3` port redraws the floating menu objects after text without preserving the original plot-if-empty rule, the frontend will look subtly wrong even if all the right art is present.

The crisp preservation target is:

- redraw or update text/highlight first
- draw floating object pixels afterward
- but keep object plotting non-destructive with respect to existing non-zero text pixels

That should preserve the original visual hierarchy without needing to emulate VGA literally.

### Next 5 Strongest Moves

1. Tighten the remaining ownership question around `0x1765a`, especially whether it is dead, indirect-only, or reached from a not-yet-exported frontend or gameplay helper.
2. Do a focused edge-path pass on mutable frontend rows so music/title/value updates and keybinding capture are described frame-accurately inside the steady loop.
3. Tighten the first resumed gameplay frame in specific edge paths: active alert tile, active tracked particles, and spawn/next-piece transition.
4. Keep expanding the owned raw artifact set around the frontend menu family so options, keyboard setup, and sound setup have the same direct raw support as the main menu now does.
5. Continue converting these frontend layering findings into direct renderer constraints for the future port so text/object priority is preserved without guesswork.
