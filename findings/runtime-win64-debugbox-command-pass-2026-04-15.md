# Runtime Win64 DEBUGBOX Command Pass

Date: 2026-04-15

## Summary

This pass adds an upstream DOSBox-X Win64 portable runtime under `.tools` and verifies that its DOS shell exposes `DEBUGBOX` command help text.

Result: unlike the Linux package currently in `.tools/bin/dosbox-x`, the Win64 portable runtime reports `DEBUGBOX` command usage and debugger-entry purpose text.

## New Owned Artifacts

- [dosbox-x-vsbuild-win64-2026.03.29-portable.zip](/home/bigfnj/projects/@Project-Tetris/.tools/downloads/dosbox-x-vsbuild-win64-2026.03.29-portable.zip)
- [Win64 portable runtime folder](/home/bigfnj/projects/@Project-Tetris/.tools/opt/dosbox-x-vsbuild-win64-2026.03.29)
- [probe run folder](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-help-quiet-20260415T150341Z)
- [dosbox.log (probe run)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-help-quiet-20260415T150341Z/dosbox.log)

## Method

Using the Win64 DOSBox-X executable from the extracted portable package, we ran:

- `DEBUGBOX /?`
- followed by `EXIT`

with `-log-con` enabled so DOS-shell response lines were captured in `dosbox.log`.

## Key Results

From [dosbox.log (probe run)](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/debugger-capability/win64-debugbox-help-quiet-20260415T150341Z/dosbox.log):

- `Parsing command line: DEBUGBOX /?` is present
- DOS CON line with:
  - `Runs program and breaks into debugger at entry point.`
- DOS CON line with:
  - `DEBUGBOX [command] [options]`

This confirms `DEBUGBOX` command availability in this runtime path.

## Interpretation

We now have a project-local DOSBox-X runtime option with debugger helper command surface present.

This closes the previously observed limitation where Linux package probes returned:

- `Bad command or filename - "DEBUGBOX"`

## Next Step Gate

Use this Win64 runtime for the next ordered capability test:

- confirm whether we can generate deterministic instruction-level trace artifacts (`LOGCPU*.TXT` or equivalent) for target-address reachability closure.
