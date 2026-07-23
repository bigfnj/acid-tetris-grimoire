# Runtime Address Probe Pass

Date: 2026-04-15

## Summary

This pass hardens and executes a reproducible DOSBox-X runtime probe for the target code offsets:

- `0x0001765a`
- `0x00017660`
- `0x000175c5`
- `0x00017613`
- `0x00017821`

The harness now distinguishes true instruction-trace signals from generic emulator startup lines.

Result: this run confirms executable activity (`ATET.EXE` and `ATET.DAT`) but does **not** expose an instruction-level address-trace channel in the current DOSBox-X run mode.

## New Owned Artifacts

- [run_dosbox_address_probe.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_dosbox_address_probe.py)
- [summary.json (run 20260415T211722Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/address-probe/20260415T211722Z/summary.json)
- [dosbox.log (run 20260415T211722Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/address-probe/20260415T211722Z/dosbox.log)
- [dosbox.conf (run 20260415T211722Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/address-probe/20260415T211722Z/dosbox.conf)
- [stdout.log (run 20260415T211722Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/address-probe/20260415T211722Z/stdout.log)
- [stderr.log (run 20260415T211722Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/address-probe/20260415T211722Z/stderr.log)

## Method

The script:

1. creates a timestamped run directory under `Decompilation.Effort/research/runtime-trace/address-probe/`
2. writes a run-scoped DOSBox-X config with debug logging enabled
3. runs `.tools/bin/dosbox-x` headless (`SDL_VIDEODRIVER=dummy`, `-nogui`, `-noconsole`, `-time-limit 25`)
4. records `stdout`, `stderr`, and `dosbox.log`
5. evaluates whether an instruction-trace channel exists using strict patterns only:
   - segmented disassembly-style byte lines
   - `CS:IP` phrase
   - explicit `EIP=` / `IP=` assignments
   - general register dump assignments (`AX=...`, `BX=...`, etc.)
6. emits explicit hit/no-hit rows for each target offset

## Key Results

From [summary.json (run 20260415T211722Z)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/address-probe/20260415T211722Z/summary.json):

- `status`: `address_trace_not_detected_in_this_build_or_run_mode`
- `return_code`: `0`
- `atet_exe_execute_line_seen`: `true`
- `atet_dat_activity_seen`: `true`
- `address_trace_channel_detected`: `false`
- `address_trace_evidence`: empty
- target hits:
  - `0x0001765a`: `hit=false`
  - `0x00017660`: `hit=false`
  - `0x000175c5`: `hit=false`
  - `0x00017613`: `hit=false`
  - `0x00017821`: `hit=false`

## Interpretation

The harness itself is validated and useful for repeatable runtime evidence capture.

However, in this mode the DOSBox-X log stream does not include instruction-level trace lines for executable addresses, so target no-hit results are non-confirmatory for reachability.

## Fidelity/Port Impact

No gameplay behavior assumptions changed in this pass. This is purely a runtime instrumentation capability checkpoint and evidence baseline for subsequent higher-fidelity tracing steps.

## Next Step Gate

Before claiming runtime reachability of these offsets, we need a trace source that emits instruction-level guest addresses in a machine-readable stream (or equivalent debugger automation with deterministic capture).
