# Frontend Fade Transitions Pass

Date: 2026-07-21

## Summary

Implements the animated screen fades the frontend dispatcher (`0x3830`) runs at
the gameplay<->frontend boundary and at boot, and confirms (from the dispatcher
structure) that in-frontend navigation does NOT fade. Closes the last
frontend-parity item.

## Where fades occur (RE)

The dispatcher `0x3830` fades only at session boundaries, driven by the palette
fade helpers `0x64f0` (full->black) and `0x6498` (black->full), both `0x40` = 64
ticks (each waits on the `[0x2d2a3]` timer tick between palette steps):

- **Enter frontend** (Esc in play, or boot): palette fade-out the current screen
  (`0x64f0`) -> copy in the title base -> chunk-7 objects fade in (`0x63b8`).
- **Exit to gameplay** (New Game state `4` / Return state `2`): chunk-7 fade-out
  (`0x62e0`) -> restore the gameplay snapshot -> gameplay palette fade-in
  (`0x6498`).
- **Exit to DOS** (Exit Game state `3`): chunk-7 fade-out (`0x62e0`) -> quit.
- **Boot splashes** (`0x2998`): each page fades in, holds, fades out.
- **In-frontend navigation** (menu<->options<->credits<->high-scores<->keyboard):
  NO fade. Handlers return the next state and the dispatcher loops without
  re-running the entry bootstrap; the chunk-7 objects animate continuously and
  each screen does its own row reveal. The port already matched this.

## Port implementation

A screen fade-through-black controller in `MilestoneADemo` (approximating the
separate chunk-7-object / palette fades as one screen fade, which is the visible
effect):

- `BeginSceneFade(target)` starts a 64-tick fade-out (or switches instantly when
  fades are disabled). `AdvanceScreenFade()` steps it each tick; at black it runs
  `CompleteSceneSwitch()` (the deferred scene change) and starts the 64-tick
  fade-in. `Tick()` freezes game logic while a fade runs (the original busy-waits
  on the timer), and `HandleKeyDown` ignores input during it.
- Staged transitions: Esc (gameplay -> main menu), New Game, and Return to Game.
  Exit Game already fades (music-synced, `frame-cadence` / earlier pass).
- Boot splash pages fade in/out within the existing 120-frame page (modeled
  `kSplashFadeTicks = 24`, since it must fit the hold; the exact `0x2998` fade is
  longer than the port's page).
- `main.cpp` composites a single black overlay from the max of the demo's
  `screen_fade_alpha()` and the music exit-fade progress.
- Observable via a `fade=<none|out|in>:<frame>` field in `--debug-state`.

## Guardrails (determinism)

- `--no-fade` disables all fades.
- The `--live-demo` / `--topout-demo` shortcuts imply fades-off: they jump
  straight into gameplay (bypassing the real menu path) and must stay
  frame-deterministic for smoke runs. So every existing smoke scenario is
  unaffected; `pause-resume` (which crosses Esc/Return) stays instant.

## Verified

- All smoke scenarios `EXIT=0`; fingerprints unchanged.
- Real New Game (no shortcut): the menu holds ~64 ticks (fade-out), switches at
  black, then gameplay fades in ~64 ticks (`fade=out` x63, `fade=in` x64 in
  `--debug-state`). Shortcuts report `fade=none`.
- The smoke harness (`port/run-smokes.sh`) now asserts the fade timing and that
  shortcuts stay fade-free.

## Correction (2026-07-21): objects DO animate during the fade

Decoding `0x62e0`/`0x63b8` showed they are not separate "object-alpha" fades:
each is a full palette fade (via `0x284c`/`0x2574`, like `0x64f0`/`0x6498`) that
*also* runs the chunk-7 object update (`0x17613`/`0x175c5`) every tick, so the
floating objects keep drifting while the screen fades. The port already matches
this: its object animation lives in `RenderFrontendPointfield`
(`AdvanceFrontendPointfield`), which runs from `Render` every frame — and `Render`
keeps running during a fade (only `Tick` freezes). So the objects animate
through the fade with no extra work.

The one ordering refinement made: on the entry fade-in the original runs the
object fade first and only reveals the menu rows afterward. `RenderFrontend` now
freezes `AdvanceFrontendTimers` while a fade is active, so the row reveal is held
until the fade completes (verified: `rows=0` during a menu fade-in). The exact
`0x62e0`/`0x63b8` per-object-shading (vs the whole-screen palette scale) is the
only remaining nuance and is cosmetically indistinguishable.

## Splash fade

The port's splash fade (`kSplashFadeTicks = 24` in/out within the 120-frame page)
is modeled: the original `0x2998` fade is a ~0x80-tick palette ramp with a
key-break inside a different page structure, so exact replication would require
reworking the port's simplified 2-page splash. This is boot-only cosmetic and
left as the modeled approximation.

## Exact transition cadences

The remaining "exact cadence not certified" rows (menu reveal cadence, conceal
timing, splash hold) are not statically decidable from the code alone; they need
a side-by-side capture against the running original (the capture-certification
pass).
