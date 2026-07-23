# Gameplay Input Timing Pass

Date: 2026-04-14

## Summary

This pass resolves the held-input timing model inside the live gameplay loop and turns several previously vague piece-state fields into concrete gameplay semantics.

The most important result is that the game does not use one generic repeat counter. It uses four independent `8`-frame repeat timers for left, right, rotate-left, and rotate-right, plus a separate soft-drop lockout field that is reused across line-clear and spawn transitions.

## `0x05e0` Seeds Gravity In Fixed-Point Form

The new-game initializer at `0x05e0` computes the live gravity value stored at `0x2c717` as:

- `(current_level << 9) + 0x200`

Equivalent reading:

- `512 * (level + 1)` fixed-point units per gameplay frame

The gameplay loop compares that running gravity value against a threshold of `0x10000`, so the nominal natural descent interval is:

- `128 / (level + 1)` gameplay frames per row

That same startup path also calls `0x09c8` with `EAX = 1`, which resets all gameplay input-repeat state before the first active piece is played.

## `0x09c8` Uses Four Independent Action Repeat Timers

`0x09c8` maintains one repeat timer per held action:

- `0x2c77f` -> left
- `0x2c77b` -> right
- `0x2c777` -> rotate-left
- `0x2c773` -> rotate-right

Each timer behaves the same way:

- if the timer is greater than `0`, it is decremented once per gameplay frame
- if the bound key is held and the timer is `0`, the game attempts that action immediately
- after the attempt, the timer is loaded with `8`
- if the key is not held, the timer is reset to `0`

Practical source-port meaning:

- first action happens immediately on press
- held repeat cadence is once every `8` gameplay frames
- left, right, rotate-left, and rotate-right each repeat independently

## Soft Drop Uses A Separate Hold Lockout

The down binding at `0x2c73b` does not use the same repeat timer model.

Instead:

- the normal gravity increment comes from `0x2c717`
- if Down is held and `0x2c783 == 0`, the frame increment is overridden to `0x8000`
- the frame increment is then clamped to `0x10000`
- the increment is added into the gravity accumulator at `0x2c6d3`
- when that accumulator reaches the descent threshold, the flat-binary path at `0x00000d03` stores `0` back into `0x2c6d3` before collision resolution continues

Nominal reading:

- natural gravity reaches the `0x10000` descent threshold based on level
- held Down forces a half-threshold increment per gameplay frame
- in practice, that gives a two-frame row descent cadence when starting from an empty accumulator
- successful single-row descent consumes the accumulator completely instead of carrying remainder state

## `0x2c783` Is Not Generic DAS State

The field at `0x2c783` is easy to misread as another repeat timer, but its behavior is different.

Observed behavior:

- `0x09c8(EAX = 1)` clears it during gameplay initialization
- each normal gameplay frame decrements it by `1` when it is positive
- if Down is not held, it is reset to `0` immediately
- after a line-clear scoring event, the game loads it with `0x348`
- no non-line-clear spawn path currently shows an equivalent `0x348` seed

Best current interpretation:

- `0x2c783` is a held-Down lockout that prevents a carried Down press from automatically soft-dropping through line-clear and spawn transitions
- releasing Down clears the lockout state, so a fresh press can take effect immediately afterward
- ordinary non-line-clear piece entry appears to rely on the normal spawn reset path instead of a dedicated lockout seed

That makes this field preservation-important even though it is not part of the visible HUD.

## `0x1348` Fully Resets The Spawned Piece State

The spawn helper at `0x1348` is now precise enough to describe field-by-field.

After promoting the previous next-piece ID into the current-piece slot and choosing a new next-piece ID, it resets:

- `0x2c737` -> piece X = `4`
- `0x2c71b` -> piece Y = `0`
- `0x2c733` -> rotation index = `0`
- `0x2c6d3` -> gravity accumulator = `0`

This confirms that the earlier vague "fall-state" reading can now be split into:

- `0x2c733` -> rotation index
- `0x2c6d3` -> gravity accumulator

## Porting Impact

This pass gives the future Windows source port a much more faithful gameplay-input target:

- action repeat should be modeled per action, not with one shared timer
- held repeat cadence should be `8` gameplay frames
- soft drop should use a separate override path, not the left/right/rotate repeat logic
- the Down lockout around line clears and new piece entry should be preserved before any modernization is attempted
- gravity should be implemented as the executable’s level-scaled fixed-point model, not a guessed table
- successful row descent should zero the gravity accumulator rather than preserving fractional remainder

## Recommended Next Move

The best next target is now one level outside the inner movement logic:

- find the outer gameplay frame cadence and tick driver that call into `0x09c8`
- map the piece-lock and warning-effect timing edges against runtime capture
- use that timing model as the future fixed-step reference for the Windows port
