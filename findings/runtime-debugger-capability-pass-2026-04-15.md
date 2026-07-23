# Runtime Debugger Capability Pass

Date: 2026-04-15

## Summary

This pass checks whether our current project-local DOSBox-X runtime can be driven into an instruction-level debugger surface suitable for scripted address tracing.

Result: in the tested mode, debugger control did not engage. `-break-start` did not pause autoexec execution, and no debugger output surface (or `LOGCPU*.TXT` trace files) was produced.

## New Owned Artifacts

- [run_dosbox_debugger_capability_probe.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_dosbox_debugger_capability_probe.py)
- [summary.json (run 20260415T213150Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/20260415T213150Z/summary.json)
- [dosbox.log (run 20260415T213150Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/20260415T213150Z/dosbox.log)
- [pty-output.txt (run 20260415T213150Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/20260415T213150Z/pty-output.txt)
- [pty-output.bin (run 20260415T213150Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/20260415T213150Z/pty-output.bin)
- [dosbox.conf (run 20260415T213150Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/20260415T213150Z/dosbox.conf)

## Method

The probe script:

1. writes a dedicated DOSBox config with:
   - `debuggerrun = debugger`
   - autoexec mount to `Original.Game`
   - `ATET.EXE` launch
2. starts DOSBox-X using:
   - `-break-start`
   - `-nogui`
   - fixed time-limit
3. runs DOSBox-X through a PTY and injects:
   - `help`
   - `logs 200`
   - `run`
4. captures:
   - DOSBox logfile
   - PTY output transcript
5. checks for:
   - whether autoexec still executed `ATET.EXE`
   - debugger-specific hint output
   - `LOGCPU*.TXT` generation

## Key Results

From [summary.json (run 20260415T213150Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/20260415T213150Z/summary.json):

- `status`: `debugger_surface_not_detected_in_current_build_or_mode`
- `break_start_effective`: `false`
- `execute_atet_hits`: present
- injected command echoes were seen in PTY output (`help`, `logs 200`, `run`)
- `debugger_hint_hits`: empty
- `logcpu_files`: empty

Interpretation of this run:

- command injection path itself works (echoed inputs are captured)
- in this mode, those inputs are not entering an active debugger command surface
- this runtime cannot currently provide deterministic instruction-level trace output for target-address reachability claims

## Fidelity/Port Impact

No gameplay/asset behavior model changed. This pass is strictly a tooling-capability gate for runtime trace confidence.

## Next Step Gate

To continue ordered runtime reachability closure, we need a trace-capable runtime path that emits guest instruction addresses (for example, a debugger-enabled DOSBox-X workflow that actually halts and accepts debugger commands in automation).
