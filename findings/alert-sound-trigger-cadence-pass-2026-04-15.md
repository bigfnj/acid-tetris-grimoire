# Alert Sound Trigger Cadence Pass

Date: 2026-04-15

## Summary

This pass tightens the smiley or alert sound behavior into an explicit cadence model.

Main result:

- warning sounds are cooldown-gated by three counters in `0x21c4`
- the cooldowns pause during line-clear collapse because `0x21c4` is bypassed
- alert lifetimes still age during collapse through `0x206c`

That combination explains why alert visuals and warning sounds can desynchronize under heavy clear activity without being a bug.

## Warning Band Model (`0x21c4`)

`0x21c4` decrements three per-band cooldown timers each call:

- `0x2c753` low band
- `0x2c757` mid band
- `0x2c76b` high band

Topmost occupied row comes from `0x1de8` and resolves into bands:

- high: `topmost_row <= 2`
- mid: `topmost_row in 3..4`
- low: `topmost_row in 5..7`
- `>= 8`: no warning trigger

Trigger behavior:

- high band:
  - alert effect `5`, lifetime `0xa0`
  - sound slot `4` (center pan) only when mid cooldown is also ready
  - resets all three cooldowns to `0x12c`
- mid band:
  - alert effect `4`, lifetime `0xa0`
  - sound slot `4` (center pan)
  - resets mid + low cooldowns to `0x12c`
- low band:
  - alert effect `3`, lifetime `0xa0`
  - sound slot `3` (center pan)
  - resets low cooldown to `0x12c`

## Collapse Interaction

When pending line clears are active (`0x184df > 0`):

- `0x09c8` short-circuits to `0x1d04`
- that bypasses `0x21c4`, so warning cooldown counters do not decrement
- but `0x206c` still runs, so active alert lifetime/reveal continues

Fidelity implication:

- do not freeze alert lifetime during collapse
- do preserve warning-cooldown pause during collapse

## Related Sound Paths (for context)

- line-clear sounds: `0x0f4f` slots `{7,8,9,10,11}`
- top-out sound: `0x1065` slot `5`
- sleepy timeout sound: `0x10dc` slot `6`

## New Machine-Readable Artifact

- `/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/alert-sound-trigger-model.json`
