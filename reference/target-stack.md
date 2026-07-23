# Target Stack

This document defines the intended technology target for the ACiD Tetris restoration and source-port effort.

## Primary Goal

Rebuild the original DOS game as a faithful native Windows 11 application first, then consider optional modernization after behavioral parity is achieved.

## Approved Target Stack

- Language: `C++23`
- Primary IDE: `Visual Studio 2026`
- Primary compiler: `MSVC`
- Build system: `CMake`
- Platform layer: `SDL3`
- Target operating system: `Windows 11`

## Why This Stack

- The original game is a native DOS executable built with a Watcom-era C/C++ toolchain, so a modern native C++ rebuild is the most direct preservation path.
- `C++23` gives us strong language features, good structure, and safer implementation options without forcing a game engine.
- `Visual Studio 2026` and `MSVC` provide a modern, stable Windows-native development environment.
- `CMake` keeps the project build portable and source-controlled independently of the IDE.
- `SDL3` gives us practical control over windowing, input, audio, rendering, and timing while staying lightweight.

## Engineering Guidelines

- Keep core gameplay logic independent from SDL where practical.
- Use a fixed timestep for gameplay and timing-sensitive systems.
- Preserve the original low-resolution presentation model internally, then scale for modern displays.
- Prefer simple native systems over heavy engine dependencies during the faithful-port phase.

## Asset Targets

- Graphics: `PNG` for converted working assets
- Sound effects: `WAV`
- Music: preserve recovered source-like forms where possible, with converted playback assets as needed
- Configuration and persistence: simple modern text or structured data formats after original behavior is understood

## Preservation Rules

- Keep `Original.Game/` untouched.
- Preserve raw extracted data separately from converted assets.
- Treat the original DOS executable as the behavioral source of truth.
- Consider differences from the DOS original to be regressions unless they are intentional post-parity modernization changes.

## Secondary Note

The project should be `CMake`-first rather than IDE-first. `Visual Studio 2026` is the primary day-to-day target, but the build should remain organized so it can be generated and compiled outside the IDE if needed.
