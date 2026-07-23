# Runtime Win64 Debugger Command Injection Pass

Date: 2026-04-15

## Summary

This pass re-ran debugger command automation after user permission timing was corrected.

Outcome:

- we can reliably land in debugger-start state on Win64 runtime
- we can focus the DOSBox window by process ID
- command injection is still not executing debugger commands yet in this non-interactive shell context

## New Owned Artifacts

- [window-title probe run](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-windowtitle-probe-20260415T152909Z)
- [window-title process metadata](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-windowtitle-probe-20260415T152909Z/process.txt)
- [sendkeys injection run](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-sendkeys-20260415T152951Z)
- [sendkeys run log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-sendkeys-20260415T152951Z/dosbox.log)
- [stdin injection run](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugger-inject-20260415T152737Z)
- [interactive injection run](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-interactive-inject-20260415T153920Z)
- [manual launcher script](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_win64_debugger_manual.ps1)

## Method

1. `stdin` injection attempt:
   - launched Win64 DOSBox-X with `-console -break-start`
   - piped `LOGS 200`, `RUN` over stdin
2. window-focus probe:
   - launched with `-console -break-start`
   - captured process window metadata
3. SendKeys attempt:
   - activated process window via `WScript.Shell.AppActivate(processId)`
   - sent `LOGS 200{ENTER}` then `RUN{ENTER}`

## Key Results

### Window-focus probe

From [process.txt](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-windowtitle-probe-20260415T152909Z/process.txt):

- process is responsive
- main window title observed:
  - `DOSBox-X 2026.03.29: DOSBOX-X - 3000 cycles/ms`

### SendKeys run

From [sendkeys run log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-sendkeys-20260415T152951Z/dosbox.log):

- debugger entry banner appears:
  - `***| TYPE HELP (+ENTER) TO GET AN OVERVIEW OF ALL COMMANDS |***`
- no evidence of executed `LOGS`/`RUN` commands
- no `LOGCPU*.TXT` artifacts produced

### Stdin run

- run directory created and startup path confirmed
- no debugger-command execution evidence captured from piped stdin

### Interactive launcher run

From [interactive injection run](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-interactive-inject-20260415T153920Z):

- process/window activation by PID and title both reported true
- debugger entry banner still appears
- command execution evidence (`HELP`, `LOGS`, `RUN`) still absent in logfile

## Interpretation

Permission timing is no longer the blocker for launching/debugger entry.

The remaining blocker is command delivery into the active debugger input surface. Current shell-driven injection (stdin + simple SendKeys) is not yet reaching/committing debugger commands in a reproducible way.
In addition, this agent shell context enforces command-timeout boundaries on long-lived GUI subprocess runs, so final debugger-command entry should be performed from a user-owned interactive desktop terminal.

## Next Step Gate

Use the explicit user-driven launcher script to perform debugger entry commands (`HELP`, `LOGS 400`, `RUN`) on a desktop-attached session, then harvest `LOGCPU*.TXT` into `Decompilation.Effort/research/runtime-trace/` and resume target-offset closure.
