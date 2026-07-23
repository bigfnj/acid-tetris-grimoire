# Runtime WSL Native Debugger Surface Pass

Date: 2026-04-16

## Summary

This pass continued the WSL-native DOSBox-X debugger track from the earlier feasibility note and answered the next concrete question:

- can a `.tools`-staged Linux debug build of DOSBox-X expose a real debugger surface under WSL?

Result:

- yes, a source-built native Linux debug binary under `.tools` is viable
- `-break-start` on that build produced debugger help output in a PTY-attached run
- a `LOGCPU.TXT` artifact was captured under the run directory

## Build Outcome

Native Linux debug build artifacts now exist under:

- launcher:
  - [.tools/bin/dosbox-x-linux-debug](/home/bigfnj/projects/@Project-Tetris/.tools/bin/dosbox-x-linux-debug)
- build driver:
  - [.tools/bin/dosbox-x-linux-debug-build](/home/bigfnj/projects/@Project-Tetris/.tools/bin/dosbox-x-linux-debug-build)
- installed binary:
  - [.tools/opt/dosbox-x-linux-debug/install/bin/dosbox-x](/home/bigfnj/projects/@Project-Tetris/.tools/opt/dosbox-x-linux-debug/install/bin/dosbox-x)

The `.tools` build root was extended far enough to compile and link the native debug binary. The final install only reported a non-fatal `setcap` permission issue, which is expected under unprivileged WSL and does not block debugger use.

## Probe Result

The existing debugger capability probe was re-used with an explicit binary override:

- `DOSBOX_X_BIN=.tools/bin/dosbox-x-linux-debug python3 Decompilation.Effort/scripts/run_dosbox_debugger_capability_probe.py`

Run artifact:

- [summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/20260416T162549Z/summary.json)

Key machine-readable result:

- `status = "debugger_surface_detected"`

Important signals from that run:

- debugger help text appeared in PTY output
- `LOGCPU.TXT` was harvested into the run directory
- `ATET.EXE` still executed during the run, so `-break-start` is not yet behaving like an early hard stop in the way we originally hoped

Trace artifact:

- [LOGCPU.TXT](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/20260416T162549Z/LOGCPU.TXT)

## Interpretation

This closes the main feasibility question for the WSL-native branch:

- the old packaged Linux runtime was not enough
- the new source-built Linux debug runtime is enough to expose a debugger command surface and produce CPU trace output

That means the project no longer has to treat Win64 DOSBox-X as the only plausible path for debugger-assisted runtime closure.

## Remaining Caveats

- the shell-level `DEBUGBOX /?` probe is still inconclusive in headless dummy-SDL mode:
  - [summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/debugbox-command/20260416T162533Z/summary.json)
- the successful native debugger pass relied on `-break-start` plus PTY injection, not on confirmed DOS-shell `DEBUGBOX` helper execution
- `break_start_effective` remained `false` in the probe summary because execution still advanced into `ATET.EXE`

## Practical Next Step

Use the native Linux debug build for the next closure pass around `0x1765a` and related reachability targets:

1. point the address/debugger probes at `.tools/bin/dosbox-x-linux-debug`
2. increase logging depth and capture length beyond the current capability pass
3. drive the run specifically toward the unresolved address set and inspect the harvested `LOGCPU*.TXT` evidence

## Bottom Line

WSL-native debugger work is now proven possible with the `.tools` source-built Linux debug binary.

The branch has moved from feasibility work into actual runtime-trace closure work.
