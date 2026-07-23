# ACiD Tetris Subsystem Port Plan

Date: 2026-04-21

## Purpose

This document turns the current reverse-engineering closure state into an implementation order for the native port.

Assumption:

- subsystem porting is now primarily blocked by implementation effort, not by missing executable-side understanding

Status snapshot:

- Milestone A is complete
- the recommended next implementation target is Milestone B

## Authoritative Inputs

Start each port pass from:

1. [behavior-spec.md](../specs/behavior-spec.md)
2. [transition-preservation-spec.md](../specs/transition-preservation-spec.md)
3. [checklist.md](checklist.md)
4. [restart-audit.md](../restart-audit.md)

Machine-readable inputs with the highest port value:

- `research/ghidra/gameplay-screen-asset-composition.json`
- `research/ghidra/frontend-state-transitions.json`
- `research/ghidra/sound-event-map.json`
- `research/ghidra/gameplay-present-order.json`
- `research/ghidra/gameplay-edge-paths.json`
- `extracted/converted/`

## Implementation Order

### 1. Asset-Backed Presentation Core

Goal:

- show the authored title and gameplay base screens correctly in the SDL port
- load chunk-3 digits and tetromino tiles
- establish a real 320x240 presentation surface instead of a flat clear color

Acceptance target:

- the port can present static title and gameplay scenes with authored assets, without gameplay logic yet

### 2. Frontend Shell

Goal:

- implement the frontend state graph and text rows
- preserve menu row ordering, selected-row pulse, and the chunk-7 object layering contract
- get `New Game`, `Options`, `Keyboard Setup`, `High Scores`, and `Credits` navigable

Acceptance target:

- the user can move through the major frontend states and see the right text, row order, and transition family

### 3. Gameplay Core

Goal:

- implement the 10x20 board, current and next piece, spawn rules, collision, gravity, and repeat timers
- render the live piece, preview, HUD digits, and piece-stat counts over the authored gameplay base

Acceptance target:

- a fresh run behaves like the original for ordinary in-progress play before line clears and top-out edge cases

### 4. Gameplay Edge Behavior

Goal:

- implement line clears, helper-driven debris, alert behavior, resume from frontend, and top-out staging
- preserve the `New Game` and `Return to Game` transition contracts already closed in the specs

Acceptance target:

- gameplay no longer diverges on the important edge paths

### 5. Persistence And End-State Flow

Goal:

- implement config persistence, keyboard rebinding, startup setup behavior, high-score qualification, and name entry
- preserve music continuity and SFX slot mapping

Acceptance target:

- the port covers the full shipped loop from startup through high-score return to main menu

## Current Code Cluster And Next Expansion

The current port surface now includes:

- `port/src/main.cpp`
- `port/src/audio_probe.cpp`
- `port/include/audio_probe.h`
- `port/src/indexed_asset_loader.cpp`
- `port/include/indexed_asset_loader.h`
- `port/src/milestone_a_demo.cpp`
- `port/include/milestone_a_demo.h`

The safest next subsystem expansion is to add one bounded frontend-shell cluster before gameplay rules:

- frontend text-row rendering from the existing menu/font assets
- selected-row pulse and mutable-row redraw
- chunk-`7` decorative-object cycling behind the text
- title/options/high-scores/credits state wiring over the authored title base

That keeps the next implementation step visual and behaviorally bounded while reusing the already-verified Milestone A presentation surfaces.

## Suggested Milestone Boundaries

### Milestone A

Status:

- complete on `2026-04-21`

- title base screen loads
- gameplay base screen loads
- chunk-3 digits and piece tiles draw over the gameplay base

### Milestone B

Status:

- next recommended milestone

- frontend rows and row selection work
- options values redraw correctly
- chunk-7 object cycle is visible behind text

### Milestone C

- ordinary gameplay loop works
- preview and HUD digits update
- new piece spawn and gravity match the executable model

### Milestone D

- line-clear collapse and helper effects work
- alert bands and top-out staging work
- return-to-game semantics hold

### Milestone E

- setup/config persistence works
- high-score qualification and name entry work
- audio slot mapping and continuity are preserved end to end

## Files To Update On Every Port Pass

Minimum project surfaces:

- `session-log-YYYY-MM-DD.md`
- [checklist.md](checklist.md)
- `port/README.md` when build, run, or milestone status changes

Update only when understanding changes:

- [behavior-spec.md](../specs/behavior-spec.md)
- [transition-preservation-spec.md](../specs/transition-preservation-spec.md)
- findings docs under `docs/findings/`

## What Is Still Optional RE, Not A Port Blocker

- cleaner main-menu certification capture
- cleaner restart-scene or line-clear capture sequences
- further whole-program map-density work

Those can still improve confidence and polish.
They should not stop subsystem-by-subsystem implementation.
