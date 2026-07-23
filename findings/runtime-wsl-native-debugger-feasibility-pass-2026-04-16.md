# Runtime WSL Native Debugger Feasibility Pass

Date: 2026-04-16

## Summary

This pass tested whether the existing WSL-native DOSBox-X path can replace the Win64 debugger path for final runtime trace closure.

Result:

- the currently installed WSL-native package under `.tools` is **not** sufficient for debugger closure
- a WSL-native path still appears **possible**, but only through a new source-built Linux debug build staged under `.tools`

## What Was Tested

Current WSL-native runtime:

- `.tools/bin/dosbox-x`
- backing binary:
  - `.tools/opt/ubuntu-root/usr/bin/dosbox-x`
- packaged version:
  - `2024.03.01`

Source/build material now staged locally:

- cloned upstream source:
  - `.tools/src/dosbox-x`

Important upstream-local references inside that source tree:

- debugger behavior:
  - [.tools/src/dosbox-x/README.debugger](/home/bigfnj/projects/@Project-Tetris/.tools/src/dosbox-x/README.debugger)
- Linux SDL2 debug build helper:
  - [.tools/src/dosbox-x/build-debug-sdl2](/home/bigfnj/projects/@Project-Tetris/.tools/src/dosbox-x/build-debug-sdl2)
- general build instructions:
  - [.tools/src/dosbox-x/BUILD.md](/home/bigfnj/projects/@Project-Tetris/.tools/src/dosbox-x/BUILD.md)

## Key Results

### 1. The packaged Linux binary still does not expose `DEBUGBOX`

Even when launched from a real PTY with dummy SDL/audio settings and explicit DOS-shell command injection:

- `DEBUGBOX /?` was parsed by the DOS shell
- DOS CON still reported:
  - `Bad command or filename - "DEBUGBOX"`

This reproduces the earlier Linux-package limitation under better launch conditions.

### 2. `-break-start` on the packaged Linux binary still does not give a usable debugger stop

With a real PTY attached and the game mounted through autoexec:

- `ATET.EXE` still executed
- PMODE/W banner output appeared
- no debugger help banner appeared
- no evidence of the `TYPE HELP` debugger surface was observed

So the current Ubuntu-packaged Linux build under `.tools` is not a viable replacement for the Win64 debugger path.

### 3. The upstream source strongly suggests Linux debugger support is real in the right build

Local upstream documentation now present in `.tools/src/dosbox-x` states:

- on Linux, DOSBox-X must be started from a terminal in order to enable the debugger
- the debugger interface is console-based and built around `ncurses`
- `DEBUGBOX` and `LOG` / `LOGS` / `RUNWATCH` are documented debugger commands

The local upstream `build-debug-sdl2` helper also explicitly builds with:

- `--enable-debug=heavy`
- `--enable-sdl2`

That makes a source-built Linux debugger path technically credible even though the packaged Ubuntu binary is insufficient.

### 4. Current WSL build prerequisites are incomplete

Before this pass:

- no local `pkg-config` was available in PATH
- no build-oriented `.tools` dev package root existed yet

Network-approved dependency download into `.tools/downloads` has now started, which means the native-build path is no longer blocked on repository policy, only on finishing staged build prerequisites under `.tools`.

## Interpretation

The project should treat the WSL-native path as:

- **not currently usable** with the existing packaged `.tools/bin/dosbox-x`
- **still feasible** through a fresh Linux debug build staged under:
  - source in `.tools/src/`
  - build/install output in `.tools/opt/`
  - launcher wrapper in `.tools/bin/`

## Practical Recommendation

Best next move if we continue pursuing WSL-native:

1. finish staging Linux build dependencies under `.tools`
2. build a dedicated Linux debug variant from `.tools/src/dosbox-x`
3. install it into a separate prefix under `.tools/opt/`
4. expose it through a new wrapper instead of replacing the current packaged runtime immediately
5. repeat the terminal-attached `DEBUGBOX /?` and `-break-start` probes against that new build

## Bottom Line

The WSL-native route is possible enough to keep exploring, but not with the current packaged runtime.

If we continue down this branch, we should now switch from “probe the Ubuntu package” to “build a dedicated Linux debug binary under `.tools` and test that instead.”
