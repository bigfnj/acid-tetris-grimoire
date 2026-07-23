## High-Score Name Entry And Helper Boundary Pass - 2026-04-14

This pass paired one visible frontend seam with one remaining low-level engine seam:

1. the exact runtime sequencing of high-score qualification, live name entry, commit, footer exit, and conceal
2. the remaining ownership and reachability boundary around `0x1765a`

That combination worked well because both questions live near the same renderer and present boundary, and both matter for a faithful port.

### New Owned Artifacts

- `research/ghidra/exports/decompilations/highscore-edge-pass/raw-50b0-5a93.asm`
- `research/ghidra/exports/decompilations/highscore-edge-pass/raw-175c5-178af.asm`
- `research/ghidra/exports/decompilations/highscore-edge-pass/highscore-edge-pass.index.json`
- `research/ghidra/highscore-name-entry-behavior.json`

### Main Result

The high-score screen is now sharper at the per-frame interaction level, and `0x1765a` is now better fenced even though it is still not directly owned by a recovered caller.

The strongest visible result is:

- high-score name entry is a two-input, row-local redraw loop rather than a generic text field

The strongest engine-structure result is:

- `0x1765a` still has no direct branch references in the full flat-binary sweep, and its lack of bounds checks now makes it look more like a local fast primitive than a public subsystem entry point

### High-Score Qualification And Reveal

The front half of `0x50b0` holds up well against our earlier model, but the raw slice makes it more concrete.

When called with `EAX = 1`:

- it scans the five saved score records at `0x2c623`
- finds the inserted row if the current run score qualifies
- shifts lower rows down when needed
- clears the inserted name field through the formatting helper
- writes the current run score into the saved score field
- writes total cleared lines from `0x2c72f` into the secondary field

Then it enters the shared 48-frame high-score reveal:

- three table columns are redrawn row-by-row through `0x60cc`
- the footer is redrawn through `0x60cc`
- `0x39c4` advances the shared frontend object system
- `0x175c5` redraws the current chunk-7 object pixels
- `0x17719 -> 0x24d0` flushes and presents

That preserves the earlier conclusion that the score table is not an instant static dialog, but now it is better grounded in the raw export itself.

### Name Entry Is A Two-Input Local Rewrite Loop

This is the biggest visible-behavior improvement from the pass.

Once the reveal completes and qualification mode is active, the live name-entry loop is not just:

- read one key
- append or delete
- redraw everything

The tighter current reading is:

1. clear previously drawn chunk-7 object pixels through `0x17613`
2. optionally poll `0x964` for released `Enter` to stage a commit request
3. clear only the active row band through `0x61c8`
4. rebuild that row’s visible left-column string from the temporary stack buffer through `0xf787`
5. advance the shared frontend object system through `0x39c4`
6. pulse the active row through `0x5868(row, 0)` while still typing
7. if commit is pending, run `0x5868(row, 1)` as a one-shot completion gate
8. poll `0x964` again for editing input
9. apply backspace, case-toggle, and translated character insertion into the temporary name buffer
10. redraw chunk-7 objects once
11. flush and present once

That is much crisper than the older shorthand and is a good implementation target for the port.

### Specific Name-Entry Behaviors

The raw handler confirms these details:

- the temporary name buffer lives on the stack
- the visible name is NUL-terminated after each accepted edit
- visible length is capped at `10`
- `Backspace` is raw code `0x0e`
- raw code `0x3a` toggles the local case-selection flag
- the raw release-poll helper `0x964` is used both for commit staging and for edit polling
- `0x94c` clears the release latch after accepted actions

Character translation is more structured than a single lookup table:

- `0x18acb`
- `0x18bcb`
- `0x18c0b`
- `0x18c4b`

and the current keyboard modifier bytes at `0x2c22b + 0x2a` and `0x2c22b + 0x36` still participate in choosing which table result becomes the inserted glyph.

So the future port should preserve:

- raw key-to-character translation as a separate step
- local case-toggle behavior
- a staged commit action distinct from ordinary character edits

### Commit And Footer Exit Are Separate Gates

The raw slice also sharpens the transition out of name entry.

When commit is staged:

- the handler switches from continuous `0x5868(row, 0)` to one-shot `0x5868(row, 1)`
- only when that one-shot gate returns success does it leave the live edit loop

After that it does **not** return immediately.
It enters the separate footer-exit loop we had identified before, but now with better raw grounding:

- row `6` is pulsed through `0x3fd8`
- live `Esc` state through the input block and released `Enter` through `0x964` can stage exit
- once footer exit is staged, the handler enters the matching 48-frame conceal loop

So the high-score screen has three distinct user-visible interaction phases after qualification:

- live typing with continuous row pulse
- row-commit gate
- footer-exit gate

That should absolutely be preserved in the port instead of collapsed into a single confirm action.

### `0x1765a` Boundary Is Stronger, Not Broader

This pass did not discover a direct owning caller for `0x1765a`, but it did improve the confidence boundary.

What is now firmer:

- a full flat-binary disassembly sweep still shows no direct `call` or `jmp` to `0x1765a`
- the same sweep shows no direct branch to the mid-body label at `0x17660`
- unlike `0x175c5` and `0x17613`, `0x1765a` has no bounds checks at all
- it still uses the same one-pixel working-screen math family
- it still writes `CL` and marks the dirty cell as `3`
- it still returns the previously stored pixel byte in `AL`

That combination now points a little more strongly toward this reading:

- useful low-level fast primitive
- behavior understood
- not yet safe to model as a public subsystem entry point

So the best current preservation stance is still:

- keep it documented
- do not invent an owning high-level system for it yet
- do not fold its semantics into the tracked transient path

### Practical Porting Impact

For the future port, this pass adds two concrete constraints:

1. High-score qualification and name entry should preserve their original phased presentation:
   - reveal
   - live row-local edit loop
   - row-commit gate
   - footer-exit gate
   - conceal

2. The low-level renderer should preserve the distinction between:
   - tracked transient pixel plotting and restoration
   - untracked direct pixel swap helpers

even if `0x1765a` itself stays abstracted until we know its owning caller.

### Next 10 Strongest Moves

1. Tighten the alert-tile lifetime edge cases further, especially expiry and immediate reset behavior across consecutive gameplay-owned frames.
2. Tighten tracked-particle edge cases where cleanup occurs but the subsequent draw set is empty or materially smaller than the prior frame.
3. Expand the owned raw artifact set around options, keyboard setup, and sound setup edge paths so the secondary frontend states are grounded like the main menu.
4. Tighten the spawn-collision and top-out presentation path one more step, especially the first visible game-over frame before frontend state `9`.
5. Revisit the first visible `state 9` bootstrap against captures using the newer frontend layering and gameplay-edge model together.
6. Keep searching for indirect or computed dirty-map writes that could weaken or refine the current mark-`3` propagation model.
7. Keep the `0x1765a` question open, but downgrade it to a narrower reachability-confirmation task rather than a broad behavior task.
8. Continue converting these high-score and gameplay-edge findings into explicit implementation constraints for the future `C++23 + SDL3` port.
9. Do a fresh evidence pass on post-game-over audio tails versus the high-score bootstrap so SFX continuity is as precise as music continuity.
10. Consider a fresh session-log handoff once the next cluster of frontend edge-path notes lands, because the fine-grained frontend fidelity model is now materially richer than the last root-level log.
