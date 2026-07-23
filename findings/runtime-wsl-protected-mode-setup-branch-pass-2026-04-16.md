# Runtime WSL Protected-Mode Setup-Branch Pass

Date: 2026-04-16

## Summary

This pass added explicit `ATET.EXE` startup-argument support to the native WSL protected-mode probe and used it to force the `setup` startup branch.

Main result:

- explicit `setup` mode is now an owned probe path
- setup-mode plus selected sound-setup mutations can still reach PMW1 object `2`
- but none of the tested setup-path variants beat the current best earliest-object2 floor of `0868:000178B1`

The closest setup-path results were:

- `0868:000178B4`
- `0868:0001791F`

Both remain later than the best plain-launch control.

## Tooling Change

Updated:

- [run_dosbox_protected_mode_breakpoint_probe.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_dosbox_protected_mode_breakpoint_probe.py)
- [README.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/README.md)

New capability:

- repeated `--atet-arg` values are appended to the `ATET.EXE` command line in `[autoexec]`

This allows startup-path probes such as:

- `ATET.EXE setup`

Verification:

- `python3 -m py_compile Decompilation.Effort/scripts/run_dosbox_protected_mode_breakpoint_probe.py`

## Key Artifacts

Initial setup-mode comparison:

- [20260416T231649Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231649Z/summary.json)
- [20260416T231649Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231649Z-01/summary.json)
- [20260416T231649Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231649Z-02/summary.json)

Sound-setup row-family sweep:

- [20260416T231807Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231807Z/summary.json)
- [20260416T231807Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231807Z-01/summary.json)
- [20260416T231807Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231807Z-02/summary.json)
- [20260416T231807Z-03/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231807Z-03/summary.json)

Follow-up sound-setup variations:

- [20260416T231917Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231917Z/summary.json)
- [20260416T231917Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231917Z-01/summary.json)
- [20260416T231917Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231917Z-02/summary.json)

## Findings

### 1. Setup-mode alone did not help

At `LOGC 0x3C0800`:

- plain-launch control in this batch landed in object `3` at `0868:000247D0`
- `ATET.EXE setup` plus a simple `AUTOTYPE -w 6 enter` landed at the same `0868:000247D0`

So forcing the setup branch by itself did not move the runtime into an earlier object-2 landing.

### 2. Setup-mode plus sound-setup mutations can reach object `2`

The first promising setup-path mutation was:

- `AUTOTYPE -w 6 -p 0.3 right down down down down enter`
  actually tested in the first batch as:
- `AUTOTYPE -w 6 -p 0.3 right down down down down enter`

The strongest early setup-path result from the first batch was:

- [20260416T231649Z-02/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231649Z-02/summary.json)
  -> object `2` at `0868:0001791F`

The stronger sound-setup row-family sweep improved that to:

- [20260416T231807Z/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231807Z/summary.json)
  `AUTOTYPE -w 6 -p 0.3 down right down down down enter`
  -> object `2` at `0868:000178B4`
- [20260416T231807Z-01/summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/20260416T231807Z-01/summary.json)
  `AUTOTYPE -w 6 -p 0.3 right right down down down down enter`
  -> object `2` at `0868:000178B4`

That is only 3 bytes later than the current best overall floor.

### 3. The setup-path improvements were not stable enough to beat the control

Nearby sound-setup variations fell back into object `3`:

- `down down right down down enter` -> `0868:000247CC`
- `down down down right down enter` -> `0868:000247D3`
- `right down down down down enter` -> `0868:000236BC`
- `down right right down down down enter` -> `0868:000247C6`
- `right down right down down down enter` -> `0868:000247D7`

So the setup-mode branch is real and branch-sensitive, but it did not produce a new floor below `0x178B1`.

### 4. The best control remains unchanged

After this pass, the best-known earliest object-2 control is still:

- `LOGC 0x3C0800`
- `AUTOTYPE -w 6 enter enter`
- selected live stop `0868:000178B1`

The best explicit setup-path result remains slightly later:

- `0868:000178B4`

## Practical Next Step

This pass keeps the startup-branch question from staying fuzzy:

1. explicit `setup`-mode probes are now available and owned
2. the setup/sound-setup branch can approach the floor closely, but did not beat it
3. future setup-path work should be treated as a real branch family, but not as the new default control
4. the current baseline earliest-object2 control remains the plain-launch `LOGC 0x3C0800` plus `AUTOTYPE -w 6 enter enter`
