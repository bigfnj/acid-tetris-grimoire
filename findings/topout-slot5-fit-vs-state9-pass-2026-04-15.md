# Top-Out Slot-5 Fit Vs State-9 Pass

Date: 2026-04-15

## Summary

This pass closes the remaining practical uncertainty around the top-out SFX tail.

Using the owned top-out capture audio and the extracted SFX slots, slot `5` (`top_out_game_over`) is the strongest fit in the `7s..12s` window and its fitted end lands far before the first perceptible state-`9` bootstrap frame.

Best current wording:

- slot `5` is the right top-out one-shot
- it naturally ends well before visible state-`9` bootstrap in the owned transition capture
- no explicit game-over/state-`9` SFX cutoff is needed for faithful port behavior

## New Owned Artifacts

- [audio-sfx-fit-analysis.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/audio-sfx-fit-analysis.json)
- [analyze_topout_audio_sfx_fit.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/analyze_topout_audio_sfx_fit.py)

## Method

The script:

1. reads the capture mono WAV and all converted slot WAVs from [sound-event-map.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/sound-event-map.json)
2. resamples all inputs to `11025 Hz` mono 16-bit
3. ranks slot fits against:
   - full clip
   - focused `7s..12s` window where top-out one-shot should dominate
4. derives slot-`5` fitted end time from `fit_seconds + duration_seconds`
5. compares that to the first state-`9` support frame from [analysis.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/analysis.json) (`frame-022 -> ~22s`)

## Key Results

From [audio-sfx-fit-analysis.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/topout-exit-hiscore-menu/audio-sfx-fit-analysis.json):

- focus-window best slot: `5` (`top_out_game_over`)
- slot-`5` fit time: `8.439456s`
- slot-`5` duration: `0.87s`
- slot-`5` fitted end: `9.309456s`
- second-best focus score is slot `0`, with score delta `+0.06778956` in favor of slot `5`
- first state-`9` support frame: `frame-022` (`~22.0s`)
- fitted gap from slot-`5` end to state-`9` bootstrap: `12.690544s`

The artifact interpretation therefore resolves:

- `slot5_is_best_in_focus_window = true`
- `slot5_tail_reaches_state9_bootstrap = false`

## Fidelity Impact

For the `C++23 + SDL3` port:

- keep slot `5` as immediate spawn-failure one-shot
- do not replay slot `5` during dissolve/overlay/high-score bootstrap
- do not add a synthetic SFX cutoff at state `9`
- allow natural one-shot completion while music continuity remains independent

## Bottom Line

The remaining “does the slot-5 tail overlap perceptible state-9?” seam is now effectively closed by owned evidence.

The executable model and capture-fit model now agree:

- top-out SFX is early and short
- state-`9` bootstrap is much later
- overlap is not expected in normal runtime presentation
