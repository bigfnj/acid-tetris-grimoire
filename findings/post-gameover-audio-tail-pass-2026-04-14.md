# Post-Game-Over Audio Tail Pass

Date: 2026-04-14

## Summary

This pass stayed on one narrow fidelity seam:

- what actually happens to audio after top-out, through the dissolve, `GAME OVER` overlay, and later `state 9` bootstrap

The most useful result is that the executable-side model is now tighter than the old wording.

- the top-out/game-over SFX is a one-shot slot-`5` trigger at spawn failure
- the later dissolve, overlay, state-`9` bootstrap, and high-score handler do **not** currently show any explicit audio stop, fade, reseed, or replay behavior
- so the post-top-out audio story is now best modeled as **passive continuity only**

That means the remaining uncertainty is no longer "what logic path owns this audio?"
It is just:

- how much of the natural tail is still perceptible in real runtime audio before the later frontend presentation dominates

## New Owned Artifact

- [post-gameover-audio-tails.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/post-gameover-audio-tails.json)

## 1. The Top-Out / Game-Over Sound Fires Once At Spawn Failure

The spawn-failure slice already showed the direct SFX call:

- [raw-0ec0-10b7.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-edge-pass/raw-0ec0-10b7.asm)

The relevant chain is:

1. failed spawn check succeeds
2. alert `6` is staged through `0x2008`
3. game-over helper is staged through `0x1f8c(EAX = 2)`
4. slot `5` is played through `0x6817` at `0x1065`

That matches the current sound-event map:

- slot `5`
  [top_out_game_over](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/sound-event-map.json)
- converted asset duration:
  `0.87` seconds

So the top-out/game-over sound is not a later overlay sound or a state-`9` sound.
It belongs to the immediate spawn-failure edge.

## 2. The Later Game-Over Presentation Path Is Audio-Passive

The next important question was whether the later gameplay and frontend path does anything else to audio.

The owned raw slices now support a stronger negative reading.

### Row-By-Row Dissolve And Overlay Helpers

In:

- [raw-1edc-23ff.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/topout-pass/raw-1edc-23ff.asm)

we already have the visual game-over chain:

- `0x1edc`
  row dissolve worker
- `0x1f8c`
  dissolve driver
- `0x22fc`
  saved-under `GAME OVER` overlay draw
- `0x23ac`
  overlay restore before frontend

What matters here is what is **not** present in that later path:

- no `0x6544`
- no `0x67aa`
- no `0x699e`
- no `0x69b0`

The same exported slice does include the known alert-warning SFX calls at:

- `0x2244`
- `0x2294`
- `0x22e3`

but those belong to the alert subsystem neighborhood, not to the dissolve or overlay handoff itself.

So the stronger reading is:

- the later game-over presentation path does not actively intervene in audio

## 3. The High-Score Handler Is Also Audio-Passive

The owned high-score raw slice is:

- [raw-50b0-5a93.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/highscore-edge-pass/raw-50b0-5a93.asm)

A direct call scan of that slice now supports a strong negative result:

- no `0x6544`
- no `0x67aa`
- no `0x67cf`
- no `0x676a`
- no `0x6817`
- no `0x699e`
- no `0x69b0`

That means the high-score reveal, live name entry, footer exit, and conceal phases do not currently look like audio owners at all.

So once the game reaches `0x50b0`, the audio model is still:

- music continuity only
- no explicit SFX reseed
- no explicit SFX cutoff

## 4. The Shared Frontend Entry Is Not A Hidden Audio Cutoff Here

The state-`9` handoff still runs through:

- `0x23ac`
- `0x3830`
- `0x50b0`

and the current dispatcher-side model already showed:

- hard exit state `3`
  uses `0x69b0` and `0x699e`
- main-menu music row
  uses `0x6544`

What this pass adds is that none of the post-game-over path pieces we actually use for `state 9` show a different audio intervention.

So the safest preserved wording is now:

- `state 9` is **not** an audio-reset point
- `state 9` is **not** an SFX-cutoff point
- `state 9` inherits whatever audio state survives naturally from the earlier gameplay path

## 5. What This Means For The Actual Tail Of Slot `5`

This is where the wording should stay honest.

What we can now say strongly from the executable:

- slot `5` is fired once at spawn failure
- the later path does not replay it
- the later path does not explicitly stop it
- the later path does not fade audio because of game over or state `9`

So the remaining practical question is only:

- does the natural slot-`5` tail still overlap audibly with later visual phases?

Current best reading:

- probably not by the time `state 9` becomes active in a perceptible way

Why that reading is reasonable:

- slot `5` is only `0.87` seconds long
- the game then performs a `0xa0`-step row dissolve before even drawing the late `GAME OVER` overlay
- after that it still waits for the finished-game-over handoff seam before frontend state `9` begins

So even without direct transition audio capture, the executable strongly suggests:

- the top-out/game-over SFX is a short one-shot attached to failure
- by the time the later high-score-family frontend presentation is visible, any remaining slot-`5` audio should usually have ended naturally

The part I would still keep modest is:

- exact runtime perceptibility remains capture-light

## Porting Impact

For the future `C++23 + SDL3` port, the safest faithful rule is:

- fire the top-out/game-over SFX once at spawn failure
- do not replay it during the later dissolve, overlay, or high-score bootstrap
- do not invent an explicit SFX cutoff at the state-`9` handoff
- let the one-shot tail complete naturally
- preserve music continuity independently across the same handoff

That is cleaner and more faithful than tying the sound to the `GAME OVER` overlay or to the high-score screen.

## Bottom Line

This pass tightened the audio seam in a useful way.

- the top-out/game-over sound is now clearly an immediate spawn-failure one-shot
- the later game-over and high-score path is audio-passive
- the remaining uncertainty is only about audible overlap, not ownership or control flow

That is a very good place to be for the port.

## Update (2026-04-15)

That remaining overlap question is now substantially closed by owned capture-fit analysis:

- [topout-slot5-fit-vs-state9-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/topout-slot5-fit-vs-state9-pass-2026-04-15.md)

Current bounded result:

- slot `5` fits strongest in the top-out window
- fitted slot-`5` end is around `9.309s`
- first state-`9` bootstrap support frame is around `22.0s`
- gap is roughly `12.69s`

So overlap with perceptible state-`9` bootstrap is not expected in the owned transition capture.

## 10 Next Strongest Moves

1. Expand the owned raw artifact set around options, keyboard setup, and sound setup edge paths so the secondary frontend states are grounded like the main menu.
2. Keep searching for indirect or computed dirty-map writes that could weaken or refine the current mark-`3` propagation model.
3. Keep the `0x1765a` question open, but narrow it to reachability confirmation rather than broad behavior analysis.
4. Tighten line-clear particle overlap cases now that the tracked overwrite semantics are corrected.
5. Tighten whether any alert refresh paths can visibly coexist with tracked transient overwrites in the same `8x4` dirty cells.
6. Revisit the first visible `state 9` bootstrap again if we later recover or record a direct transition sequence.
7. Keep converting these top-out, alert, transient, and high-score-family findings into explicit implementation constraints for the future `C++23 + SDL3` port.
8. Refresh the root session log once the next frontend or post-game-over cluster lands, because the transition model is now materially tighter.
9. Start a renderer-contract note that groups first-frame, scene-bootstrap, dirty-flush, and overlay-layer rules into one port-facing implementation reference.
10. If you want to tighten the final runtime uncertainty here, a direct top-out-to-high-score video capture with audio would be the highest-value new evidence.
