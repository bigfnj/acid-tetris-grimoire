# State 9 Bootstrap Capture Support Pass

Date: 2026-04-14

## Summary

This pass stayed on one narrow fidelity question:

- how much the owned capture set actually supports the current executable-side `state 9` bootstrap model

The main result is useful and a little more conservative than the earlier wording.

- the captures do **not** directly show the post-game-over `state 9` handoff
- they **do** strengthen the shared-bootstrap model for the high-score screen family
- the strongest support comes from the fact that the owned high-score screenshot still carries the shared title/logo base and the chunk-7 floating-object layer rather than behaving like a bespoke table-only backdrop

That means our current preservation wording can now be a little sharper:

- the exact first visible post-game-over `state 9` frame is still primarily executable-derived
- the broader visual claim that high-score-family screens ride on the shared frontend title/object presentation is now capture-supported

## New Owned Artifact

- [state9-bootstrap-capture-support.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/state9-bootstrap-capture-support.json)

## 1. `04-hiscores.png` Supports The Shared High-Score Family Presentation

The owned high-score screenshot is:

- [04-hiscores.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/04-hiscores.png)

What it visibly preserves:

- the shared `ACiD TETRIS` title/logo base at the top
- the same black-backed frontend layout family as the menu screens
- a floating green chunk-7 object behind the score-table text

That is important because the executable-side model already says `0x3830` performs a shared frontend bootstrap before handing off to `0x50b0`.
The screenshot is not direct proof of the first visible `state 9` frame, but it is strong support for the claim that the high-score screen family is built on the same title/object presentation system rather than on a separate dedicated backdrop.

## 2. The Capture Correlation Result Supports That Reading Too

The earlier capture-alignment pass already aligned `04-hiscores.png` back to the recovered `320x240` title screen and scored it against the chunk-7 overlay renders.

The strongest current aligned result for [04-hiscores.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/04-hiscores.png) is:

- aligned capture:
  [04-hiscores.aligned-320x240.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-alignment/04-hiscores.aligned-320x240.png)
- alignment score:
  `5.446667`
- best current chunk-7 match:
  [bank03-to-bank00.progress064.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank03-to-bank00.progress064.title-overlay.png)
- best-match mean absolute RGB difference:
  `11.705712`

That matters because the ordinary title screenshot behaves almost the same way:

- [01-title.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/01-title.png)
- best current chunk-7 match:
  [bank03-to-bank00.progress064.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank03-to-bank00.progress064.title-overlay.png)

So the high-score screenshot is not drifting into some separate visual family.
It lands inside the same title-base plus chunk-7-object family as the ordinary menu captures.

## 3. This Is Still Not A Direct `State 9` Transition Capture

This distinction matters enough to say plainly.

The owned screenshot [04-hiscores.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/04-hiscores.png) is best treated as a high-score-family steady-state screen, not as a direct recorded post-game-over transition frame.

Why that matters:

- the executable shows `0x50b0` serves both plain high-score display and qualifying-name-entry modes
- the capture itself does not prove whether it came from state `8` or state `9`
- the capture does not show the moment where the cleaned gameplay underlay fades out and the shared frontend bootstrap becomes visible

So the accurate confidence split is:

- **capture-supported**
  high-score-family screens preserve the shared title/logo base and chunk-7 object layer
- **still primarily executable-derived**
  the exact first visible post-game-over `state 9` bootstrap cadence

## 4. What This Changes In Practice

This pass does not overturn the current executable-side model.
It sharpens it.

The safest preserved wording is now:

- finished game-over still enters frontend state `9` through the confirmed gameplay/frontend handoff seam
- the first visible `state 9` phase is still best modeled as the shared frontend bootstrap
- steady owned high-score-family capture evidence now supports that those screens retain the shared title/logo base and chunk-7 floating-object layer
- what we still do **not** have is a direct owned transition sequence showing exactly how much of that bootstrap is perceptible before the high-score reveal dominates

That is a better research position than either extreme:

- weaker than "captures prove the whole transition"
- stronger than "captures tell us nothing about the bootstrap"

## Porting Impact

For the future `C++23 + SDL3` port, the safest faithful rule here is:

- do not treat the high-score family as a standalone table screen with a custom background
- preserve the shared title/logo base and chunk-7 floating-object layer behind high-score display and entry screens
- still keep the exact first visible post-game-over `state 9` bootstrap timing marked as executable-led rather than fully capture-proven

## Bottom Line

This pass improved the evidence boundary more than the behavior model.

- the captures still do not directly show the post-game-over `state 9` handoff
- they now support the shared-bootstrap claim better than before
- that gives us a safer, more honest basis for the eventual port

## Update (2026-04-15)

A dedicated sub-second timing follow-up now tightens the first visible bootstrap window:

- [state9-bootstrap-onset-timing-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/state9-bootstrap-onset-timing-pass-2026-04-15.md)

Current bounded result from same-capture reference scoring:

- state-`9` crossover onset around `21.083s`
- stronger state-`9` dominance around `21.333s`

So the old “exact first visible frame is still primarily executable-derived” caveat can now be narrowed to:

- still approximate (similarity-based),
- but no longer only 1-fps coarse capture evidence.

## 10 Next Strongest Moves

1. Do a fresh evidence pass on post-game-over audio tails versus the high-score bootstrap so SFX continuity is as precise as music continuity.
2. Expand the owned raw artifact set around options, keyboard setup, and sound setup edge paths so the secondary frontend states are grounded like the main menu.
3. Keep searching for indirect or computed dirty-map writes that could weaken or refine the current mark-`3` propagation model.
4. Keep the `0x1765a` question open, but narrow it to reachability confirmation rather than broad behavior analysis.
5. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
6. Tighten whether any alert refresh paths can visibly coexist with tracked transient overwrites in the same `8x4` dirty cells.
7. Revisit the first visible `state 9` bootstrap again if we later recover or record a direct transition sequence.
8. Keep converting these top-out, alert, transient, and high-score-family findings into explicit implementation constraints for the future `C++23 + SDL3` port.
9. Refresh the root session log once the next frontend or post-game-over cluster lands, because the transition model is now materially tighter.
10. Start a renderer-contract note that groups first-frame, scene-bootstrap, dirty-flush, and overlay-layer rules into one port-facing implementation reference.
