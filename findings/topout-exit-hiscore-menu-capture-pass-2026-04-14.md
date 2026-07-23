# Top-Out Exit Hi-Score Menu Capture Pass

Date: 2026-04-14

## Summary

This pass uses the new owned runtime capture:

- [topout-exit-hiscore-menu.mkv](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/topout-exit-hiscore-menu.mkv)

It turned out to be a very high-value clip because it visibly covers all of these in one sequence:

- active gameplay before top-out
- late top-out / row-dissolve presentation
- the `GAME OVER` overlay
- the later fade-down into frontend state `9`
- a directly perceptible shared title/logo bootstrap before the high-score table fully appears
- high-score steady display
- footer exit / conceal
- return to the main menu

That means this pass materially upgrades two older areas from "executable-strong, capture-light" to "executable-strong, capture-supported".

## New Owned Artifacts

- [capture analysis JSON](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/analysis.json)
- [contact-sheet.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/contact-sheet.png)
- [audio-levels-by-second.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/audio-levels-by-second.json)

## 1. The Capture Directly Supports The Staged Top-Out Chain

The early and middle portions of the clip line up well with the current executable model.

Useful extracted frames:

- [frame-011.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-011.png)
- [frame-014.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-014.png)
- [frame-021.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-021.png)

What those frames support:

- the smiley alert face is present before the late overlay phase
- the `GAME OVER` text appears as a later overlay rather than as the first visible top-out response
- the overlay persists for multiple seconds before the frontend handoff begins

That is exactly the shape we already recovered from the executable:

1. immediate top-out feedback
2. dissolve / teardown phase
3. late saved-under `GAME OVER` overlay
4. later handoff into frontend state `9`

So this clip now gives us direct visual reinforcement for a chain that used to be mostly executable-driven.

## 2. The Shared `State 9` Bootstrap Is Now Directly Perceptible

This is the biggest capture-side upgrade.

Useful extracted frames:

- [frame-022.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-022.png)
- [frame-023.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-023.png)
- [frame-024.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-024.png)

What they show:

- a darkened late game-over frame before the frontend is visible
- then a frame where the shared `ACiD TETRIS` title/logo base is already visible without the full high-score table yet dominating
- then a later frame where the high-score table is clearly present under that same title/logo base

That matters a lot, because it upgrades this older claim:

- "the first visible `state 9` stage is shared frontend bootstrap"

from:

- executable-strong, capture-light

to:

- executable-strong, directly capture-supported

The clip now shows that the shared title/logo bootstrap is not just a theoretical internal staging step.
It is actually perceptible in the observed runtime path before the high-score table fully settles.

## 3. The High-Score Family Still Preserves The Shared Floating-Object Layer

The steady high-score frames in this clip also reinforce what the earlier screenshot work suggested.

Useful extracted frames:

- [frame-026.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-026.png)
- [frame-033.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-033.png)
- [frame-035.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-035.png)

What remains true in the runtime clip:

- the shared title/logo base is still present
- the chunk-7 floating object remains active behind the table text
- the high-score family does not switch to a bespoke table-only backdrop

That strengthens the porting rule that state `9` should stay inside the shared frontend presentation family rather than being implemented as a separate isolated screen style.

## 4. The Clip Also Shows The Later Footer Exit And Return To Main Menu

This was an extra payoff from the capture.

Useful extracted frames:

- [frame-036.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-036.png)
- [frame-037.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-037.png)
- [frame-038.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-038.png)

What they support:

- the high-score screen visibly enters a conceal/transition phase
- the main menu then reappears with the same shared title/logo family and chunk-7 floating-object layer

That lines up well with the current `0x50b0` model:

- steady display / name-entry family
- footer exit gate
- shared conceal
- return to main menu

So the later half of the clip now gives us runtime support for the previously executable-derived footer-exit structure too.

## 5. Audio-Side Support From The Clip

This clip also helps, but more modestly, on audio.

The second-by-second envelope in:

- [audio-levels-by-second.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/audio-levels-by-second.json)

shows the sequence remains audibly active throughout the whole `38.2` seconds.
There is no broad silent dead zone at:

- the late `GAME OVER` overlay
- the fade-down into frontend
- the high-score screen
- the later return to the main menu

That supports the current continuity model:

- music is staying alive through the whole path
- there is no obvious global mute / reset at the `state 9` seam

What the clip does **not** cleanly isolate by itself is:

- the exact audible end of the short slot-`5` top-out one-shot

So the best wording remains:

- clip-level audio continuity is now capture-supported
- exact one-shot tail perceptibility is still not isolated with certainty

## 6. What This Changes In The Research Model

This pass sharpens two previously cautious areas.

### Earlier Wording We Can Now Strengthen

- the shared `state 9` bootstrap is perceptible before the high-score table fully dominates
- the post-game-over path is now direct capture-supported, not only endpoint-supported

### Wording That Should Still Stay Modest

- the exact user-input trigger timing for the handoff is still not recoverable from these 1-fps extracted frames alone
- the exact audible end of slot `5` is still not isolated cleanly from the continuing music bed

That is a good balance:

- stronger where the clip really proves something
- still careful where it does not

## Porting Impact

For the future `C++23 + SDL3` port, this clip strengthens these concrete rules:

- preserve the staged top-out chain instead of jumping straight to a score screen
- preserve a visibly perceptible shared title/logo bootstrap before the high-score table fully settles
- preserve the chunk-7 floating-object layer during the high-score screen family
- preserve the later conceal/return path from high scores back to the main menu
- preserve uninterrupted music continuity across the whole observed sequence

## Bottom Line

This was one of the most useful captures so far.

- it directly supports the staged top-out model
- it directly supports the perceptible shared `state 9` bootstrap
- it directly supports the high-score footer-exit return to main menu
- it modestly strengthens the capture-side audio continuity story too

That is a big fidelity win for the eventual port.

## Update (2026-04-15)

The remaining slot-`5` overlap uncertainty is now much tighter via direct fit ranking against the same capture audio:

- [topout-slot5-fit-vs-state9-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/topout-slot5-fit-vs-state9-pass-2026-04-15.md)

Updated bounded reading:

- slot `5` is the strongest top-out-window fit
- fitted slot-`5` end is around `9.309s`
- first state-`9` bootstrap support frame is around `22.0s`
- gap is about `12.69s`

So overlap with perceptible state-`9` bootstrap is not expected in the owned transition capture.

The state-`9` bootstrap timing itself is now also tighter than the old 1-fps frame labels:

- [state9-bootstrap-onset-timing-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/state9-bootstrap-onset-timing-pass-2026-04-15.md)

New bounded timing from sub-second similarity scoring:

- state-`9` crossover onset: `~21.083s`
- stronger state-`9` dominance onset: `~21.333s`
- state-`9` confidence peak: `~22.5s`
- first high-score crossover: `~23.167s`

This keeps the same staged transition model, but with better timing precision for faithful port pacing.

## 10 Next Strongest Moves

1. Expand the owned raw artifact set around options, keyboard setup, and sound setup edge paths so the secondary frontend states are grounded like the main menu.
2. Keep searching for indirect or computed dirty-map writes that could weaken or refine the current mark-`3` propagation model.
3. Keep the `0x1765a` question open, but narrow it to reachability confirmation rather than broad behavior analysis.
4. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
5. Tighten whether any alert refresh paths can visibly coexist with tracked transient overwrites in the same `8x4` dirty cells.
6. Keep converting these top-out, alert, transient, and high-score-family findings into explicit implementation constraints for the future `C++23 + SDL3` port.
7. Refresh the root session log now that the post-game-over transition model is materially stronger than before.
8. Start a renderer-contract note that groups first-frame, scene-bootstrap, dirty-flush, overlay-layer, and text-vs-object priority rules into one port-facing implementation reference.
9. If you want one more very high-value capture, the next best target would be a direct `Return to Game` resume clip showing held-input edge cases and the first resumed gameplay frame.
10. After one or two more fidelity passes, consider starting the first formal port scaffolding documents from the now-stable preservation rules.
