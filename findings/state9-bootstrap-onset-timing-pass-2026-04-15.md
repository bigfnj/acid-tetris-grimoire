# State-9 Bootstrap Onset Timing Pass

Date: 2026-04-15

## Summary

This pass tightens one remaining visual timing seam:

- when the post-game-over presentation first becomes state-`9`-bootstrap-like in the owned transition capture

Using frame-similarity scoring at `12 fps`, we now have a sub-second bound instead of only 1-fps frame labels.

## New Owned Artifacts

- [state9-bootstrap-onset-analysis.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/state9-bootstrap-onset-analysis.json)
- [analyze_state9_bootstrap_onset.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/analyze_state9_bootstrap_onset.py)

## Method

Inputs:

- transition clip:
  [topout-exit-hiscore-menu.mkv](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/topout-exit-hiscore-menu.mkv)
- reference stills from the same capture sequence:
  - game-over phase: [frame-021.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-021.png)
  - state-`9` bootstrap phase: [frame-023.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-023.png)
  - high-score steady phase: [frame-026.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/frame-026.png)

Process:

1. sample `18s..27s` at `12 fps`
2. scale to `320x180`
3. compute normalized mean-absolute-difference against each reference
4. solve for persistent crossover and stronger dominance windows

## Key Results

From [state9-bootstrap-onset-analysis.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/state9-bootstrap-onset-analysis.json):

- `state9_crossover_onset` (state-`9` ref closer than game-over ref, persistent):
  `21.083333s` (`frame_index 37`)
- `state9_bootstrap_onset` (stronger dominance margin `0.01`, persistent):
  `21.333333s` (`frame_index 40`)
- max confidence row for state-`9` vs game-over separation:
  `22.5s` (`frame_index 54`)

Also useful:

- first persistent high-score crossover vs state-`9` bootstrap ref:
  `23.166667s` (`frame_index 62`)
- strong high-score dominance (same `0.01` margin) was not reached in this window

## Fidelity Impact

The old wording “first visible state-`9` stage is capture-light” can now be tightened.

Current best reading:

- sub-second onset of state-`9`-bootstrap-like presentation is bounded to about `21.08s..21.33s`
- this is consistent with the earlier 1-fps frame support around `frame-022` (`~22s`)

For the `C++23 + SDL3` port, this improves transition pacing fidelity:

- preserve a short but perceptible pre-high-score bootstrap interval
- avoid jumping directly from late game-over overlay to fully settled high-score table

## Scope Note

This method is still similarity-based, not semantic per-pixel labeling.

So the bounded onset is strong enough for port timing decisions, while still being reported as approximate rather than sample-exact scene classification.
