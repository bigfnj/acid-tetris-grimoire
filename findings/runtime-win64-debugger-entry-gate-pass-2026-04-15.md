# Runtime Win64 Debugger Entry Gate Pass

Date: 2026-04-15

## Summary

This pass checks two debugger-entry gates on the Win64 DOSBox-X runtime now installed under `.tools`:

1. whether `-break-start` actually halts before autoexec/game launch
2. whether `debuggerrun=watch` alone emits instruction-level trace artifacts
3. whether `DEBUGBOX` accepts a simple non-interactive `/C` command path for debugger commands

Result:

- `-break-start` **does** gate execution before autoexec in this runtime path
- `debuggerrun=watch` still does **not** emit instruction-level trace output by itself
- `DEBUGBOX /C ...` is not accepted (returns bad command for `/C ...`)

## New Owned Artifacts

- [break-start run folder](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-breakstart-atet-20260415T150709Z)
- [break-start dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-breakstart-atet-20260415T150709Z/dosbox.log)
- [watch-mode run folder](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-watch-atet-20260415T150749Z)
- [watch-mode dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-watch-atet-20260415T150749Z/dosbox.log)
- [DEBUGBOX option-test run folder](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-optiontest-20260415T151037Z)
- [DEBUGBOX option-test dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-optiontest-20260415T151037Z/dosbox.log)
- [DEBUGBOX ATET run folder](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-atet-20260415T150622Z)
- [DEBUGBOX ATET dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-atet-20260415T150622Z/dosbox.log)

## Method

### A) Break-start gate check

- Win64 DOSBox-X
- `-break-start`
- autoexec includes mount + `ATET.EXE`
- time-limited headless run

### B) Watch-mode trace check

- Win64 DOSBox-X
- `debuggerrun = watch`
- autoexec includes mount + `ATET.EXE`
- time-limited headless run

## Key Results

### A) Break-start gate check

From [break-start dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-breakstart-atet-20260415T150709Z/dosbox.log):

- startup and BIOS/init lines are present
- no `Parsing command line: ATET.EXE`
- no `Execute ATET.EXE`

Interpretation: `-break-start` is effective in this Win64 runtime path for our automation context.

### B) Watch-mode trace check

From [watch-mode dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-watch-atet-20260415T150749Z/dosbox.log):

- `Parsing command line: ATET.EXE` is present
- `Execute ATET.EXE` is present
- PMODE/W banner lines are present
- no `LOGCPU*.TXT` artifacts in run folder
- no instruction-level register/disassembly trace lines in logfile

Interpretation: watch mode alone is insufficient for address-level runtime trace closure.

### C) DEBUGBOX `/C` option test

From [DEBUGBOX option-test dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-optiontest-20260415T151037Z/dosbox.log):

- `Parsing command line: DEBUGBOX /C LOGS 50` is present
- DOS CON reports `Bad command or filename - "/C LOGS 50"`

Interpretation: no immediate non-interactive `/C` path was observed for issuing debugger commands.

### D) DEBUGBOX program handoff test (`DEBUGBOX ATET.EXE`)

From [DEBUGBOX ATET dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-atet-20260415T150622Z/dosbox.log):

- `Parsing command line: DEBUGBOX ATET.EXE` is present
- `Execute ATET.EXE` is present

Interpretation: `DEBUGBOX` successfully hands off to target executable launch path, but this alone does not produce instruction-level trace artifacts in current non-interactive automation mode.

## Fidelity/Port Impact

No behavior model changed. This is a debugger tooling gate pass only.

## Next Step Gate

Use Win64 runtime with `-break-start`, then drive actual debugger commands (`LOG`/`LOGS` or equivalent) through a debugger-input automation path to generate instruction-level trace artifacts for target-offset reachability checks.
