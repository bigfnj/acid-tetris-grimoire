## Frontend Mutable Rows And Key Capture Pass - 2026-04-14

This pass stayed inside the frontend menu family and tightened one specific presentation question:

- how do mutable rows actually update inside the steady loop?

That matters for fidelity because these are the places where the user sees immediate in-menu changes:

- music volume
- sound effects volume
- sound setup device/rate/bit-depth/stereo rows
- keyboard rebinding rows

### New Owned Artifacts

- `research/ghidra/exports/decompilations/frontend-menu-primitive-pass/raw-49bc-4f37.asm`
- `research/ghidra/exports/decompilations/frontend-menu-primitive-pass/raw-5a94-5ef3.asm`
- updated `research/ghidra/exports/decompilations/frontend-menu-primitive-pass/frontend-menu-primitive-pass.index.json`
- `research/ghidra/frontend-mutable-row-behavior.json`

### Main Result

Mutable menu rows are not deferred to some later redraw pass.

The stronger current reading is:

1. detect the row-local action during the steady loop
2. clear only the affected row band through `0x61c8`
3. rewrite the backing row string immediately
4. continue the catch-up loop
5. later redraw floating chunk-7 object pixels once for the presented frame
6. flush and present

So the visible row update is part of the same steady menu cadence, not a separate overlay system.

### Options Menu

The raw loop now shows the two mutable rows directly:

- `Music Volume`
- `Sound FX Volume`

For both:

- the row band is cleared through `0x61c8`
- the stored value is changed in memory
- the row string is reformatted immediately
- only later does the frame reach the chunk-7 object redraw

That keeps the earlier layering rule intact:

- text/value updates are foreground
- floating objects are background-like point redraws

### Keyboard Setup

This was the most useful part of the pass.

The keybinding capture path is now sharper than before.

When Enter is pressed on rows `0..4`:

1. only the selected row band is cleared through `0x61c8`
2. a row-local capture prompt is written into the backing row string
3. the handler enters a dedicated capture substate
4. the selected-row pulse still redraws while waiting
5. one raw key byte is polled from `0x964`
6. `0` and `0x01` are ignored
7. on acceptance, the release latch is cleared through `0x94c`
8. the row band is cleared again
9. the accepted byte is stored into the selected binding slot
10. the normal row string is rewritten using the fixed-width key-name table at `0x18c8b`

That is a good preservation detail because it means rebinding is not just "wait for key, then replace text."
There is a real two-stage visual rhythm:

- prompt state
- accepted-key restore state

### Sound Setup

The sound-setup loop now has the same sharper reading.

Each mutable row updates in place:

- device row cycles `0x2c6d7`
- rate row cycles the local 7-entry rate table and stores to `0x2c70f`
- stereo row toggles `0x2c723`
- bit-depth row toggles `0x2c6e3`

For each of those:

- the affected row band is cleared through `0x61c8`
- the backing row string is rewritten immediately
- only later does the frame reach the floating-object redraw and present boundary

So the sound setup UI is not a generic full-screen refresh; it is a localized row-update system inside the steady loop.

### Layering Consequence

This pass strengthens the earlier frontend layering result.

Not only do text and highlight pixels sit in front of the floating objects, but **row-local value changes and capture prompts do too**.

That means the future port should preserve:

- row-band-local clear/rewrite behavior
- prompt/accepted-key sequencing for keyboard capture
- object redraw occurring later and non-destructively

### Why This Matters For The Port

This is exactly the kind of behavior that would disappear if we rebuilt the frontend as:

- one big widget tree
- one generalized menu redraw
- one decorative background layer

The original is more specific than that.
It behaves like a fixed-step menu loop with localized row rewrites and a later non-destructive object pass.

### Next 5 Strongest Moves

1. Tighten the remaining ownership question around `0x1765a`, especially whether it is dead, indirect-only, or reached from a not-yet-exported frontend or gameplay helper.
2. Tighten specific resumed gameplay edge paths: active alert tile, active tracked particles, and spawn/next-piece transition.
3. Keep expanding the owned raw artifact set around the frontend menu family so options, keyboard setup, and sound setup have direct raw support for more edge paths, not just their core loops.
4. Do a similar edge-focused pass on the high-score/name-entry flow so prompt, blink, row pulse, and commit sequencing are as sharp as keyboard capture now is.
5. Continue converting these low-level frontend findings into direct renderer and UI constraints for the future port so localized row rewrites are preserved without guesswork.
