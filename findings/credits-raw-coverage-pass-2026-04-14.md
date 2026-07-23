# Credits Raw Coverage Pass - 2026-04-14

This pass closes the remaining raw-coverage gap in the secondary frontend family by grounding the credits handler directly from the executable.

## New Artifacts

- [raw-4f38-50af.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/frontend-menu-primitive-pass/raw-4f38-50af.asm)
- updated [frontend-menu-primitive-pass.index.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/frontend-menu-primitive-pass/frontend-menu-primitive-pass.index.json)
- [frontend-credits-behavior.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/frontend-credits-behavior.json)
- updated [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)

## Main Result

The credits sequence is now grounded the same way the other frontend states are.

The important fidelity gain is this:

- credits text is drawn once per page
- then left alone
- while only the shared chunk-7 floating-object system and page timer keep advancing

So credits is not a per-frame text redraw loop like the menu-family states.

## Exact Page Layout

The raw export confirms:

- page base: `0x1918b`
- page stride: `0x140`
- line stride: `0x28`
- lines per page: `8`
- page count: `6`

The phase-1 draw path copies one full page of eight fixed-width lines into the frontend text buffers rooted at `0x2d0d3`, then runs `0x3df4`.

That means the credits really are a six-page fixed-text pager, not an arbitrarily long stream or a scrolling text system.

## Credits Uses A Three-Phase Pager, But With Static Text Persistence

The broad three-phase model still holds:

- phase `1`
  - draw current page
  - run `0x3df4`
- phase `0`
  - steady hold
- phase `2`
  - run `0x3ee4`
  - advance page
  - wrap after page `5`

The raw export sharpens the middle phase:

- it does not redraw page text every step
- it clears previously drawn chunk-7 object pixels
- advances the shared frontend object system through `0x39c4`
- increments the hold timer
- redraws chunk-7 objects
- flushes and presents

So the visible credits page persists in the working screen while the floating objects continue moving underneath it.

That is exactly the kind of subtle presentation rule a port could easily get wrong by rebuilding credits as a generic “redraw everything every frame” screen.

## Esc Behavior Is Live-State, Not Release-Latched

The raw handler also confirms a nice difference from the menu-family states:

- credits checks live `Esc` pressed-state at `[0x2c22b + 1]`
- it does not use the release latch path from `0x964`
- it guards against retrigger with a local flag
- then stages return state `1` and exits through the animated path

So credits should not be modeled as a normal row-selection menu with release-driven navigation semantics.
It is a timed pager with a live interrupt.

## Why This Matters For The Port

The faithful port should preserve:

- six fixed text pages
- one page draw per page-turn
- steady page persistence between turns
- shared floating-object animation continuing behind the page
- live-Esc interrupt behavior

That is a cleaner and more specific target than “show some credit text over the menu background.”

## 10 Strongest Next Moves

1. Keep the `0x1765a` question open, but narrow it strictly to reachability confirmation.
2. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
3. Tighten whether any indirect or computed dirty-map writes refine the current mark-`3` propagation model.
4. Revisit the high-score footer-exit-to-main-menu conceal rhythm against captures now that the prompt and commit side is tighter.
5. Tighten the spawn-collision and top-out presentation path one more step, especially the very first visible failed-spawn frame.
6. Continue converting these gameplay-layering and frontend-edge findings into direct implementation constraints for the future `C++23 + SDL3` port.
7. Refresh the root session log once the next cluster lands, because the secondary frontend family is now almost completely grounded from owned raw artifacts.
8. Start a port-facing renderer contract note that groups dirty-cell propagation, tracked overwrite semantics, alert coexistence, and page-ring presentation into one implementation reference.
9. If we stay in reverse-engineering mode, do a small focused pass on `0x1765a` reachability so that last low-level primitive either closes or stays explicitly provisional.
10. If you add more capture later, a clean credits-to-main-menu clip would be the most useful new evidence for tightening the credits exit feel against runtime presentation.
