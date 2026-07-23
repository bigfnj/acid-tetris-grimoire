# Runtime WSL Protected-Mode Breakpoint Pass

Date: 2026-04-16

## Summary

This pass moved the native WSL debugger work from generic surface detection into protected-mode breakpoint viability.

Main result:

- native WSL DOSBox-X can re-enter live `ATET.EXE` code with `VRT`
- that re-entry exposes a stable protected-mode code selector in the debugger UI
- protected-mode code breakpoints can be armed against that live selector

What is still open:

- a no-hit on `0x1765a` is still not closure by itself unless we first drive the game into a state where the neighboring helper family is expected to execute

## New Owned Tooling

- [run_dosbox_protected_mode_breakpoint_probe.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_dosbox_protected_mode_breakpoint_probe.py)

This script stages a native WSL debugger run around:

1. `BPINT 21 4B`
2. `RUN`
3. `VRT`
4. protected-mode code selector discovery
5. target code-breakpoint arming

The first machine-readable summaries from that script are useful as scaffolding, but the selector parser still needs another hardening pass before those automated summaries should be treated as authoritative for final reachability claims.

## Strong Manual Calibration Evidence

The strongest evidence in this pass is preserved under:

- [manual-calibration](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/manual-calibration)

### 1. `VRT` Re-Enters Live `ATET.EXE`

Artifact:

- [vrt-selector-exp9/pty.txt](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/manual-calibration/vrt-selector-exp9/pty.txt)

Key signal:

- after `Execute ATET.EXE`, the debugger re-entered at `0824:0000006A`

That is the first owned WSL-native proof that `VRT` can break back into live protected-mode game code rather than only BIOS / DOS loader space.

### 2. Post-`VRT` CPU Log Stays In Selector `0824`

Artifact:

- [vrt-logs-exp12/LOGCPU.TXT](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/manual-calibration/vrt-logs-exp12/LOGCPU.TXT)

Key signal:

- the short CPU log begins at `0824:006A`
- the captured `512` lines all stay in selector `0824`
- the logged instruction stream is repeated `repe movsw` activity from the live game-side copy path

This confirms that the post-`VRT` selector is not a one-line display artifact; it is the active code selector for the current execution window.

### 3. Protected-Mode Code Breakpoint Control Works

Artifact:

- [current-breakpoint-control-exp13/pty.txt](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/manual-calibration/current-breakpoint-control-exp13/pty.txt)

Key signal:

- the debugger accepted `BP 0824:006A`
- the prompt returned immediately after `RUN`

That gives us a practical positive control: code breakpoints on the live protected-mode selector are not hypothetical in native WSL; they do fire.

## Path-Specific No-Hit Evidence

Artifact:

- [known-helper-nohit-exp10/pty.txt](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/manual-calibration/known-helper-nohit-exp10/pty.txt)
- [known-helper-nohit-exp10/dosbox.log](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/protected-mode-breakpoint-probe/manual-calibration/known-helper-nohit-exp10/dosbox.log)

Key signal:

- the debugger accepted `BP 0824:175C5`
- the run continued deep into `ATET.DAT` activity without a quick prompt return

Interpretation:

- this is a real no-hit for that specific early boot plus `VRT` path
- it does **not** mean `0x175c5` is globally unreachable
- it means our current post-`VRT` re-entry point is still too early or too narrow to use a no-hit as final closure for helper-family reachability

## Impact On `0x1765a`

This pass changes the `0x1765a` question in an important way:

- before this, WSL-native could not yet prove that protected-mode code breakpoints were even viable
- now they are viable
- so a future no-hit on `0x1765a` can become meaningful once we first drive runtime into a validated frontend or gameplay state where helper execution is expected

That means the remaining gap is no longer debugger feasibility.

It is now state steering.

## Practical Next Step

Use the new native WSL probe path, but add one more layer before arming `0x1765a`:

1. reach a known frontend or gameplay steady state after `VRT`
2. validate that a neighboring helper expected in that state can trip
3. only then interpret a `0x1765a` no-hit as real closure evidence

## Bottom Line

Native WSL debugger work is now strong enough to support protected-mode code-breakpoint experiments.

The blocker has shifted from "can WSL do this?" to "how do we drive the program into the right state before judging `0x1765a`?"
