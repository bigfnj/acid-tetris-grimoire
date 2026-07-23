# Runtime DEBUGBOX Command Availability Pass

Date: 2026-04-15

## Summary

This pass resolves whether the DOSBox-X runtime currently installed in `.tools` exposes the DOS-shell helper command `DEBUGBOX`, which is a potential non-interactive route into debugger tracing workflows.

Result: `DEBUGBOX` is not available in this build/runtime path.

## New Owned Artifacts

- [run_dosbox_debugbox_command_probe.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_dosbox_debugbox_command_probe.py)
- [summary.json (run 20260415T213850Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/debugbox-command/20260415T213850Z/summary.json)
- [dosbox.log (run 20260415T213850Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/debugbox-command/20260415T213850Z/dosbox.log)
- [dosbox.conf (run 20260415T213850Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/debugbox-command/20260415T213850Z/dosbox.conf)
- [stdout.log (run 20260415T213850Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/debugbox-command/20260415T213850Z/stdout.log)
- [stderr.log (run 20260415T213850Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/debugbox-command/20260415T213850Z/stderr.log)

## Method

The probe script:

1. starts DOSBox-X with a minimal config (`-nogui -noconsole -time-limit 10 -log-con`)
2. runs DOS-shell command probe `DEBUGBOX /?`
3. exits via `EXIT`
4. parses log evidence for:
   - command parse line
   - command execution line
   - DOS-shell bad-command response

## Key Results

From [summary.json (run 20260415T213850Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/debugbox-command/20260415T213850Z/summary.json):

- `status`: `debugbox_command_missing_in_current_build`
- `parse_debugbox_hits`: present (`Parsing command line: DEBUGBOX /?`)
- `bad_command_hits`: present (`Bad command or filename - "DEBUGBOX"`)
- `execute_debugbox_hits`: none

## Interpretation

The current DOSBox-X package accepts the command line text but does not provide the `DEBUGBOX` helper program in the DOS environment.

This confirms that one major scripted debugger path is unavailable in the current toolchain configuration.

## Fidelity/Port Impact

No game-behavior conclusions changed. This is a runtime-tooling capability closure step.

## Next Step Gate

For deterministic runtime instruction tracing, we should now treat debugger surface as an external toolchain requirement (for example, a debugger-capable DOSBox-X workflow that actually exposes debugger commands and trace output artifacts).
