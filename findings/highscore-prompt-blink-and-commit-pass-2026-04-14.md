# Highscore Prompt Blink And Commit Pass - 2026-04-14

This pass stayed inside the high-score/name-entry path and tightened the part that is easiest to accidentally modernize away:

- what the visible prompt actually is
- how the cursor blink works
- when the typed name becomes persistent
- how commit differs from footer exit

## Updated Artifacts

- updated [highscore-name-entry-behavior.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/highscore-name-entry-behavior.json)
- updated [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- updated [transition-preservation-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/transition-preservation-spec.md)

Primary raw source:

- [raw-50b0-5a93.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/highscore-edge-pass/raw-50b0-5a93.asm)

## Main Result

The live high-score prompt is not a special cursor system.
It is a normal frontend string built with:

- `0x17cef = "%s_"`

That means the cursor is the underscore glyph, and its blink behavior comes from the already-mapped underscore handling inside `0x60cc`.

So the future port should not invent:

- a separate caret
- a text-box widget cursor
- a special high-score-only blink timer

The original is cleaner and more unified than that.

## The Cursor Blink Is The Shared Text-Renderer Blink

We already knew from the text-render pass that `0x60cc` treats underscore glyph code `0x42` specially and blinks it against the global tick.

This pass closes the loop for high-score entry specifically:

- the row-local prompt format is `"%s_"`
- the underscore is therefore drawn through the same text renderer as the rest of the row
- the visible typing cursor is the shared underscore blink, not a separate object

That is a very good fidelity detail to preserve because it keeps the high-score entry screen in the same visual language as keyboard setup and the rest of the frontend.

## Commit Persists The Name Before The One-Shot Row Gate Finishes

The raw commit path is sharper than a generic “press Enter, then save.”

Once released `Enter` has staged commit:

1. the temporary stack buffer is formatted into the persistent score-record name field with `0x17cd3 = "%s"`
2. the same buffer is formatted into the visible left-column row string with `0x17cd3 = "%s"`
3. only then does the handler rely on `0x5868(row, 1)` to finish the one-shot row gate

So commit is not just:

- wait for pulse to finish
- then save

It is:

- copy into record and visible row
- then finish the animated commit gate

That is a subtle but important sequencing rule for the port.

## Commit And Footer Exit Are Still Separate Phases

This pass also keeps the boundaries clean:

- released `Enter` first stages row commit
- row commit finishes through the one-shot `0x5868` gate
- only after that does the handler move into the footer-exit loop

Then the footer loop uses:

- live `Esc` state from the input block
- or released `Enter` from `0x964`

to stage the final leave-screen action on row `6`.

So the future port should preserve three distinct user-visible interactions:

1. typing with blinking underscore prompt
2. animated row commit
3. animated footer exit

## Why This Matters For The Port

If we rebuilt this screen as a simple name-entry dialog, we would lose several original behaviors at once:

- row-local prompt rendering
- shared underscore blink timing
- persistent-save before the row-gate completion
- separate commit and footer-exit phases

Those are all small details individually, but together they are exactly the sort of thing that makes an old game feel like itself.

## 10 Strongest Next Moves

1. Tighten alert-tile lifetime edge cases, especially expiry and immediate reset across consecutive gameplay-owned frames.
2. Tighten tracked-particle edge cases where cleanup occurs but the subsequent draw set is empty or materially smaller than the prior frame.
3. Expand the owned raw artifact set around credits presentation so the secondary frontend family has the same direct raw coverage throughout.
4. Keep the `0x1765a` question open, but narrow it strictly to reachability confirmation.
5. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
6. Tighten whether alert refresh paths can visibly coexist with tracked transient overwrites in the same `8x4` dirty cells.
7. Keep searching for indirect or computed dirty-map writes that could refine the current mark-`3` propagation model.
8. Revisit the high-score footer-exit-to-main-menu conceal rhythm against captures now that the prompt/commit side is tighter.
9. Continue converting these frontend and gameplay-edge findings into direct implementation constraints for the future `C++23 + SDL3` port.
10. Refresh the root session log once the next cluster lands, because the high-score interaction model is now materially tighter than the last handoff.
