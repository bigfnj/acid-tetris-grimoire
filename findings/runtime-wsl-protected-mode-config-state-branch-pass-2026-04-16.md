# Runtime WSL Protected-Mode Config-State Branch Pass

Date: 2026-04-16

## Summary

This pass tested whether persisted startup state inside `SETUP.DAT` can move the protected-mode late-state scout closer to the object-2 entry floor.

The main result is:

- setup-state validity is a real startup branch, but in this lane it made the scout worse, not better
- persisted audio fields are also a real steering lever
- the strongest audio-state variant reached object `2`, but only at `0x178E9`
- the best overall earliest-object2 control therefore remains unchanged:
  - `LOGC 0x3C0800`
  - `AUTOTYPE -w 6 enter enter`
  - selected live stop `0868:000178B1`

## Tooling Change

The protected-mode probe now accepts:

- `--game-dir`

That path becomes DOS `C:` inside the probe's generated `dosbox.conf`.

This makes it possible to run setup-state experiments against disposable cloned game folders instead of mutating the canonical:

- `Original.Game/`

The script and script index were updated accordingly:

- [run_dosbox_protected_mode_breakpoint_probe.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_dosbox_protected_mode_breakpoint_probe.py)
- [README.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/README.md)

## Fixture Setup

Disposable cloned game dirs were created under:

- [runtime-fixtures](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures)

The comparison families were:

- untouched control copy
- missing `SETUP.DAT`
- invalid-signature `SETUP.DAT`
- `device=None`
- minimal-audio config:
  - `device=None`
  - `11025 Hz`
  - mono
  - `8-bit`
  - music `0`
  - SFX `0`
  - track index `5`

All runs used the current best scout lane:

- `--post-vrt-log-mode logc`
- `--post-vrt-log-steps 0x3C0800`
- `--autoexec-line 'AUTOTYPE -w 6 enter enter'`
- `--target-offset 0x17719`

## Config Validity Comparison

Runs:

- [20260416T235605Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T235605Z/summary.json)
  - control copy
  - object `3` at `0868:00023B4B`
- [20260416T235605Z-01](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T235605Z-01/summary.json)
  - missing `SETUP.DAT`
  - object `3` at `0868:000247C8`
- [20260416T235605Z-02](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T235605Z-02/summary.json)
  - invalid `SETUP.DAT` signature
  - object `3` at `0868:000247C8`

What this means:

- missing or invalid setup state does force the known early startup setup branch
- but along this scout lane it does not help us escape into an earlier object-2 stop
- both broken-setup variants were worse than the control repeat

So config validity should not be treated as a promising floor-improver for this search.

## Persisted Audio-State Comparison

Runs:

- [20260416T235743Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T235743Z/summary.json)
  - `device=None`
  - object `3` at `0868:00023B17`
- [20260416T235743Z-01](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T235743Z-01/summary.json)
  - minimal-audio config
  - object `2` at `0868:000178E9`

Interpretation:

- persisted audio state is a real steering lever
- simply forcing `device=None` was not enough
- the stronger minimal-audio configuration did reach object `2`
- but `0x178E9` is still later than the current best floor at `0x178B1`

So this family is more interesting than missing/invalid setup, but it still did not replace the current control.

## Esc Follow-Up Classification

The earlier setup-mode `Esc` follow-up also now has a clean interpretation:

- [20260416T234704Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T234704Z/summary.json)
  - `AUTOTYPE -w 6 esc`
  - object `3` at `0868:00023B56`
- [20260416T234704Z-01](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T234704Z-01/summary.json)
  - `AUTOTYPE -w 6 -p 0.3 down esc`
  - object `3` at `0868:000247E2`
- [20260416T234704Z-02](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T234704Z-02/summary.json)
  - `AUTOTYPE -w 6 -p 0.3 enter esc`
  - object `2` at `0868:000178C2`

That keeps the setup `Esc` branch in the same general story:

- it can reach object `2`
- but only later than the current floor

## Bottom Line

This pass closes two adjacent branch questions:

1. broken or missing `SETUP.DAT` is not a useful floor-improver here
2. persisted audio state can bias the lane, but the tested audio variants still do not beat `0x178B1`

The next promising branch should therefore be more structural than:

- config validity flips
- simple setup-mode `Esc` exits
- or the first batch of persisted audio-field mutations
