# Transition Preservation Spec

Date: 2026-04-14

## Purpose

This document captures the current best executable-side preservation rules for scene and state transitions in ACiD Tetris.
It is meant to guide the future `C++23 + SDL3` source port toward faithful behavior before modernization changes are considered.

## Scope

This spec currently covers:

- cold boot into the frontend
- the early sound-setup detour
- `New Game`
- `Return to Game`
- finished game-over into high-score qualification / display
- transition-time input, snapshot, palette, and music behavior

## Shared Transition Rules

### 1. Snapshot Rule

Gameplay-side frontend entry snapshots the working gameplay screen only after:

- tracked transient pixels have been restored through `0x17875`

and before:

- a new `0x2f24` transient draw occurs

So saved gameplay snapshots should exclude frame-local tracked particle overlays.

### 2. Timing Rule

After the frontend returns to gameplay, the outer loop resets its timing baseline through `0x24d0(AL = 1)`.
Menu dwell time should **not** turn into a catch-up burst.

The first post-frontend gameplay render should receive:

- one fresh gameplay-step budget

not:

- a menu-time accumulation burst

### 3. Music Rule

Music continuity is scene-stable by default.

Preserve the current track across:

- `New Game`
- `Return to Game`
- post-game-over high-score flow

Track changes should happen only through:

- cold startup track selection
- the main-menu `Music` row
- explicit exit fade-out behavior

### 4. Render Boundary Rule

Do not collapse all presentation helpers into one generic "present frame" abstraction.

Current best model:

- `0x2938`
  immediate full-page seed/upload into a caller-selected VGA page
- `0x17719`
  incremental dirty-cell flush into the current back page
- `0x24d0`
  page-ring rotate, display-page swap, pacing, and next-step-count return

At major scene boundaries, the original executable seeds all three VGA pages with the same base image before returning to incremental flush/present behavior.

That scene-seeding rule is important to preserve conceptually even if the port does not emulate planar VGA.

Current direct-path survey also supports one useful negative rule:

- the only currently confirmed single-page direct-to-display upload path is the fullscreen splash presenter `0x2998`
- normal gameplay/frontend scene changes reseed all three page roots instead of presenting only one page

That statement is now supported by a direct whole-flat-binary `0x2c6bf` reference sweep, not only by the exported helper subset.

### 5. Dirty Propagation Rule

If the port keeps any dirty-region or dirty-cell presentation model, it should preserve the original propagation intent across every presentable surface, not just one back buffer.

Current best reading:

- localized changes are commonly marked with dirty value `3`
- `0x17719` decrements that value each flush
- the three-page VGA ring means a value of `3` naturally propagates one changed cell across all three pages over three presented frames

Current direct writer survey in owned exports confirms that:

- `0x2d30` writes `0x03` into the coarse dirty map
- `0x175c5` writes `0x03` for chunk-7/object plot cells
- `0x17613` writes `0x03` for chunk-7/object clear cells
- `0x17821` writes `0x03` for tracked-particle plot cells
- `0x17875` writes `0x03` for tracked-particle restore cells

The shared flush backend `0x17719` is now also executable-tight enough to preserve more directly:

- it builds an explicit copy queue at `0x1b6f7` with count at `0x1ad8f`
- queue entries are `8` bytes and the queue region up to `0x201f7` is exactly `2400` entries, matching one full `40x60` dirty sweep
- it decrements dirty bytes as cells are queued, then flushes queued cells by VGA plane
- all currently known direct `0x17719` callsites pair it immediately with `0x24d0`, so the flush remains the last changed-cell step before present pacing

We do **not** yet claim that every writer in the whole executable uses only `3`.
But the currently confirmed direct writers all support that propagation model cleanly.

If the port does **not** keep a dirty-cell model, it must still preserve the same higher-level effect:

- no stale page/surface contents should become visible on later flips after a localized update

### 6. Input Rule

Frontend exit clears only the release latch, not the live pressed-state table.

So:

- discrete frontend release events should not leak into gameplay
- physically held gameplay keys can carry across a frontend exit

That carry should interact with preserved or reset gameplay state depending on the transition path.

### 7. Secondary Frontend Menu Rule

Do not treat options, keyboard setup, and first-run sound setup as ad hoc dialogs.

Current best model:

- they share the same eight-row reveal scaffold as the main menu
- options uses four active rows plus four blank padding rows
- keyboard setup uses six active rows plus two blank padding rows
- sound setup uses six active rows plus two blank padding rows
- row-local text/value rewrites happen inside the steady loop before the later chunk-7 object redraw

Preserve the input asymmetry inside that shared scaffold:

- Up/Down row changes are release-gated one-shot actions through `0x964` and `0x3fd8`
- options left/right value edits use live held-state checks from the key table
- options music-volume edits clamp to `0..100` and apply immediately through `0x68bb`
- options SFX-volume edits clamp to `0..100` but only affect later sound playback such as navigation pulses
- sound-setup row edits mutate config in memory but do not live-reinitialize the audio backend

### 8. High-Score Interaction Rule

Do not collapse high-score qualification and name entry into a generic text-entry widget.

Current best model:

- reveal the table first
- run live typing as a row-local rewrite loop
- build the prompt with `"%s_"`, so the visible cursor is the shared underscore-glyph blink handled by `0x60cc`
- stage commit on released `Enter`
- copy the temporary name buffer into both the persistent record and the visible row before the one-shot row gate finishes
- only after that move into the footer-exit gate
- only after footer exit begin the shared conceal

### 9. Renderer Primitive Separation Rule

Do not collapse all localized screen writers into one generic "sprite" or "particle" layer.

Current direct-call evidence supports at least four distinct primitive families:

- frontend object pixels
  - `0x175c5` plot-if-empty
  - `0x17613` clear
- gameplay tracked transient pixels
  - `0x17821` plot and queue restore
  - `0x17875` restore queued pixels
- gameplay block blits
  - `0x17688` for `5x5` digits
  - `0x176df` for `8x8` tile rows
  - `0x17700` for matching `8x8` zero erases
- one documented non-required untracked pixel swap primitive
  - `0x1765a`
  - keep as a provisional preserved primitive only until a real caller or runtime lane appears

These families all feed the same dirty-cell flush backend through `0x17719`, but they are not interchangeable.

Preserve this concept in the port:

- frontend floating-object pixels should stay logically separate from gameplay transient particles
- tracked gameplay pixels should still restore on the next gameplay frame rather than behaving like persistent object pixels
- HUD digits and gameplay tile blits should remain direct block writes, not forced through the same transient-pixel path

### 10. Gameplay Present Ordering Rule

When gameplay owns the frame, the preserved present boundary is:

1. tracked transient cleanup through `0x17875`
2. per-step particle/object update through `0x2e18`
3. gameplay rules/frame step through `0x09c8`
4. alert-tile update through `0x206c`
5. fresh transient draw through `0x2f24`
6. dirty-cell flush through `0x17719`
7. page-ring flip/pacing through `0x24d0`

If multiple fixed steps are owed, items `2..4` repeat inside the catch-up loop before items `5..7` run once for the presented frame.

Inside that ordering, preserve one more layering rule:

- direct gameplay tile/HUD block blits that occur inside `0x09c8`
- and alert-tile restore/reveal work that occurs inside `0x206c`

can already alter the working screen before `0x2f24` draws tracked transient particle pixels for the presented frame.

Tracked transient pixels should also preserve their queue-backed overwrite model:

- `0x17821` overwrites the destination pixel and queues the previous byte plus restore pointers
- `0x17875` restores those queued writes in reverse order on the next outer gameplay frame
- if `0x2e18` recycles every active object before `0x2f24` runs, the next gameplay-owned frame can legitimately be a cleanup-only transient frame with no replacement transient draws

On spawn or next-piece transition frames, preserve one more edge rule:

- `0x09c8` can already redraw gameplay HUD counters through `0x2c58`
- call `0x1348` to erase the old preview piece and draw the new preview piece
- update the per-piece statistics counter
- immediately check spawn collision through `0x12ac`
- on failed spawn, stage alert `6`, seed top-out state through `0x1f8c(EAX = 2)`, and fire slot `5`
- still draw the colliding newly spawned live piece through `0x10ec` on that same failed-spawn frame when no line-clear short-circuit is active
- then let the outer `0x206c` step provide the first visible alert reveal before present

all before `0x2f24` draws tracked transient pixels for that presented frame.

That ordering matters for fidelity, especially on:

- first resumed gameplay frames
- first new-game gameplay frames
- frames with active transient particles or alerts

One more coexistence rule is now strong enough to preserve directly:

- the known alert block occupies its own fixed dirty-cell region at x cells `2..8`, y cells `35..47`
- the currently known tracked-particle emitters originate from the gameplay board area rather than that alert block
- but if later transient motion crosses into the alert block, do not special-case it away

Preserve the original ordering instead:

- tracked cleanup first
- alert refresh or restore second
- fresh tracked overlay last

That allows transient pixels to temporarily overlay the alert region for a presented frame without causing persistent corruption on later frames.

One more line-clear rule is now strong enough to preserve directly:

- each cleared board row can attempt a high-density debris queue (`8` scanlines x `80` pixels = `640` attempts)
- a four-line clear can therefore attempt up to `2560` queue inserts in one gameplay step
- both queue helpers (`0x2f78` and `0x3034`) are capped by the shared active-count limit at `0x1000` objects
- overflow attempts are dropped rather than deferred

So under heavy existing object load, line-clear debris can legitimately thin.
The port should preserve this cap-and-drop behavior rather than forcing all debris requests to appear.

Line-clear collapse interaction should also stay ordered:

- line-clear helpers queue debris from the pre-clear row image
- row bands are cleared immediately
- while clears remain pending, `0x09c8` short-circuits into `0x1d04` collapse steps
- outer-frame transient update and redraw (`0x2e18` then `0x2f24`) still run around that collapse work

That means collapse frames can legitimately include both moving board-collapse imagery and transient debris overlays.

Alert timing under line-clear load should also stay ordered:

- line-clear alerts are triggered from the clear-detect step in `0x09c8` with finite lifetime (typically `0xa0`)
- while pending clears remain (`0x184df > 0`), `0x09c8` short-circuits into `0x1d04`
- that short-circuit bypasses normal stack-warning refresh (`0x21c4`) and the later in-body line-clear alert dispatch block
- the stack-warning cooldown counters (`0x2c753`, `0x2c757`, `0x2c76b`) are decremented inside `0x21c4`, so that bypass also pauses warning-sound cadence during collapse
- but `0x206c` still runs after each `0x09c8` step in the outer gameplay loop

So alert lifetime and reveal can continue to age during collapse frames, and a line-clear-driven alert can expire mid-collapse before normal warning refresh resumes.
The port should preserve that behavior rather than freezing alert timing during line-clear animation.

## Cold Boot

### Normal Cold Boot

Current best model:

1. runtime startup allocates buffers and installs low-level services
2. frontend resources and title base are preloaded
3. optional early sound-setup detour may occur
4. concrete audio init and SFX loading occur
5. splash `0` (`DDD`) runs through `0x2998`
6. splash `2` (warning screen) runs through `0x2998`
7. gameplay-side chunks are loaded
8. selected music track starts through `0x6544`
9. `0x3830(1)` enters the normal frontend bootstrap
10. title/logo base plus chunk-7 floating objects become visible
11. eight main-menu rows reveal afterward

### Early Sound-Setup Detour

When explicit `setup` mode is requested or `SETUP.DAT` had to be seeded:

- startup enters `0x3830(5)`
- sound setup behaves as a normal frontend menu-family screen
- state `2` return restores the still-zero startup snapshot
- startup then continues into the later normal cold-boot path

This return should be modeled as effectively black or near-black, not as a return to the title backdrop.

## New Game

### Reset Rules

`New Game` should reset:

- logical board
- score
- live level from selected start level
- current and next piece bootstrap state
- gravity accumulator
- repeat timers
- Down lockout
- gameplay dirty map

### Visible Transition Rules

The original does not switch instantly from frontend to a fully settled new gameplay scene.

The preserved model is:

1. dispatcher restores the saved gameplay snapshot
2. dispatcher tail for state `4` clears the dirty-region map
3. `0x05e0` starts a fresh run
4. `0x05e0` performs an early page upload of a partially rebuilt gameplay scene
5. later HUD, preview, piece-stat, board, and alert-reset work are staged only into the working screen and dirty map
6. the shared state-`4` tail performs only a palette-side reveal, not a new screen-content present
7. outer gameplay loop resets timing baseline
8. one fresh gameplay step runs
9. first gameplay-side flush/present reveals the staged new-run state

So the first visible gameplay image after `New Game` may still reflect staged rebuilding rather than a perfectly atomic scene replacement.

There is no separately presented visible frame where the completed bootstrap layout is shown without the live piece.
That completed no-piece image exists only as an offscreen staging interval before the first gameplay-owned flush.

When that first live gameplay-owned step happens to take the spawn path, the port should also preserve that preview erase or redraw, per-piece counter update, and live-piece draw can all land before the transient overlay pass.

Important render-side detail:

- the restored gameplay snapshot and the early cleared-board image are seeded into all three pages first
- later staged HUD/piece/alert work then lands through the incremental dirty-flush/present boundary

## Return To Game

### Preserve Rules

`Return to Game` should preserve:

- logical board
- current piece and next piece
- gravity accumulator and rate
- repeat timers
- Down lockout
- score / lines / level
- live-game flag
- current music track
- persistent particle/object pool state
- alert-tile subsystem state

### Visible Transition Rules

The preserved model is:

1. dispatcher restores the saved gameplay snapshot
2. no `0x05e0` bootstrap occurs
3. outer loop resets timing baseline
4. first resumed frame begins with tracked transient cleanup if queued
5. one fresh gameplay step runs
6. particle/object draw, dirty flush, and present follow

The first resumed gameplay frame can therefore differ from the saved snapshot through:

- tracked-pixel cleanup
- one particle/object simulation update
- one gameplay step that may already perform direct block blits, or may instead short-circuit into collapse-only `0x1d04` or dissolve-only `0x1f8c` work
- one alert-tile update
- one new transient draw pass

So the first resumed frame is not just "old snapshot plus particles."

Resume-specific exception rules:

- if pending clears remain (`0x184df > 0`), the first resumed `0x09c8` step can short-circuit into `0x1d04` and return before the ordinary erase / move / spawn band
- if active top-out dissolve is still in progress (`0x184db >= 0`), the first resumed `0x09c8` step can short-circuit into `0x1f8c` and return before that ordinary band
- finished game-over sentinel (`0x184db == -2`) is **not** a real `Return to Game` case, because the dissolve completion path has already cleared live-game flag `0x2c72b`

If that first resumed gameplay-owned step enters the spawn path, it can also already redraw the preview box, update the per-piece counter, perform the immediate spawn-collision test, and draw the newly spawned live piece before transient pixels are replotted.

For the alert-tile subsystem, preserve the stronger lifetime rules too:

- fresh finite alerts can begin as six-step staged reveals
- fresh idle-start trigger paths in `0x2008` only seed effect, lifetime, and reveal state; they do not full-draw immediately
- active same-effect refresh or active different-effect replacement can store the requested effect first and full-draw that requested tile immediately without reseeding the reveal counter
- lifetime `-1` alerts can persist after reveal completes
- finite alerts that reach lifetime `0` should restore the saved alert region in that same gameplay step, so the expiry frame ends clean

For top-out and finished game-over presentation, preserve the staged visual chain too:

- failed spawn can immediately trigger alert `6`, slot `5`, and a still-presented colliding spawned piece
- the failed-spawn trigger frame is not yet the first dissolve step because `0x1f8c(EAX = 2)` only seeds `0x184db = 0` and returns
- the first visible alert reveal for that top-out path arrives through the outer `0x206c` step in the same presented frame
- the gameplay image then begins dissolving row by row on the following gameplay step through `0x1edc` under `0x1f8c`
- only after that dissolve completes does the saved-under `GAME OVER` overlay appear
- only after the finished-game-over handoff does `0x23ac` restore that underlay and let frontend state `9` begin
It can already contain freshly redrawn gameplay tiles, HUD counters, and alert-region changes before transient overlays are plotted.

Render-side consequence:

- the restored gameplay snapshot is first reseeded into all three pages
- later resumed visible differences are then carried by the incremental `0x17719 -> 0x24d0` boundary

## Finished Game-Over To High Scores

### Overlay Rule

The `GAME OVER` text is a temporary overlay, not part of the final high-score snapshot.

Preserve this sequence:

1. `0x2300` saves the covered gameplay region into `0x2c603`
2. `0x2300` draws the `GAME OVER` overlay from chunk `8`
3. dissolve progression eventually reaches finished state
4. `0x23ac` restores the saved background
5. only then does frontend state `9` begin

So state `9` should inherit the cleaned gameplay underlay, not a composite with `GAME OVER` text still visible.

### Trigger Rule

The finished game-over handoff is now directly confirmed from the flat-relocated binary.

Preserve this exact control-flow rule:

- the session loop compares the release-latch tail byte at `[0x2c227] + 0x1f` against literal byte `0x01`
- on equality it clears the latch through `0x094c`
- if `0x184db == -2`, it calls `0x23ac` and enters frontend state `9`
- otherwise the same seam enters frontend state `1`

So the accurate preservation wording is now:

- after finished game-over cleanup, the released-`Esc` gameplay/frontend handoff seam routes to frontend state `9` instead of frontend state `1`

Direct transition capture is still useful for presentation timing, but the control-flow seam itself is now directly executable-proven.

### Frontend Rule

State `9` should be treated as a normal frontend entry followed by high-score-specific presentation:

1. frontend snapshot save
2. gameplay-palette fade-out
3. title/logo base copy
4. seed all three pages from that title/logo base
5. chunk-7 object fade-in
6. high-score qualification / reveal through `0x50b0`

Do not model this path as a direct jump from gameplay overlay to score table.

Practical first-visible-frame rule:

- the first visible state-`9` frontend stage belongs to the shared title/object bootstrap
- the high-score reveal dominates only after that bootstrap has already begun

Capture-side support is now a little stronger too:

- the owned high-score screenshot confirms that the high-score screen family preserves the shared title/logo base and chunk-7 floating-object layer
- the owned runtime clip `topout-exit-hiscore-menu.mkv` now directly shows a title/logo-dominant frontend stage before the high-score table fully settles
- sub-second similarity analysis now bounds the state-`9` bootstrap crossover onset to about `21.08s`, with stronger dominance around `21.33s`, before the high-score-family crossover around `23.17s`
- so the perceptibility of the shared state-`9` bootstrap is now capture-supported as well as executable-supported

### Audio Rule

Preserve current music across the handoff into state `9`.

Do not add an explicit SFX cutoff at the handoff unless later runtime tracing proves one.
Current best reading is:

- music continues
- one-shot SFX are not explicitly stopped here
- the top-out/game-over SFX is fired once at immediate spawn failure, not during the later overlay or high-score bootstrap
- the later dissolve, overlay, and state-`9` path are audio-passive in the current owned executable evidence
- capture-fit correlation now supports a large timing gap: slot-`5` best fit in the owned top-out clip is `8.439456s`, fitted end is `9.309456s`, and first state-`9` bootstrap support frame is around `22.0s` (`~12.690544s` later)

## Menu And Frontend Presentation Rules

### Shared Frontend Bootstrap

Before any frontend handler begins its local logic, the dispatcher should preserve:

- gameplay snapshot save
- gameplay-palette fade-out
- title/logo base copy
- chunk-7 object fade-in

### Shared Frontend Exit Tail

When leaving the frontend for gameplay:

- clear only the release latch
- run the frontend fade-out
- save setup state
- restore the saved gameplay snapshot
- if state `4`, run new-game bootstrap
- reveal gameplay palette

### Persistent Frontend Object Cycle

The chunk-7 floating smiley/tetrimino system is global persistent frontend state.
Do not reseed it per menu screen.

### Frontend Menu Steady-State Layering Rule

In steady menu presentation, preserve this ordering:

1. clear previously drawn floating object pixels
2. run one or more catch-up logic steps
3. redraw text/value/highlight state during those logic steps
4. redraw the current floating object pixels for the presented frame
5. flush and present

The critical visual rule is:

- frontend text and highlight redraw should have priority over floating object pixels

Current best executable-side reason:

- menu text and pulse redraw happen before the later chunk-7 point redraw
- the later object redraw uses `0x175c5`
- `0x175c5` only plots into zero pixels

So floating object pixels should not overwrite already drawn menu text or highlight pixels.

Mutable menu rows should also preserve their localized rewrite behavior:

- clear only the affected row band
- rewrite the row text or prompt immediately inside the steady loop
- let the later floating-object redraw remain non-destructive with respect to that rewritten text

### High-Score Name Entry Rule

State `9` high-score entry should preserve its phased interaction rather than behaving like a generic modal text field.

The current best-preserved model is:

1. shared frontend bootstrap
2. shared high-score reveal
3. live name-entry loop with row-local band clear and immediate string rebuild
4. continuous active-row pulse through `0x5868(row, 0)` while typing
5. one-shot row-commit gate through `0x5868(row, 1)` after commit is staged
6. separate footer-exit gate on row `6` through `0x3fd8`
7. shared high-score conceal

Inside the live name-entry loop, preserve these interaction rules:

- released `Enter` staging is distinct from ordinary character insertion
- the active row band is cleared locally through `0x61c8` before the rebuilt name string is redrawn
- `Backspace`, local case-toggle, and translated character insertion happen inside that steady loop
- later chunk-7 object redraw remains non-destructive with respect to the rebuilt row text

## Implementation Guidance For The Port

The future source port should expose transitions as explicit state-machine edges with preserved side effects instead of treating them as generic screen swaps.

At minimum, implement:

- a gameplay snapshot buffer distinct from live working surfaces
- fixed-step resume logic with explicit baseline reset
- separate tracked-transient restoration and fresh transient draw stages
- a clear distinction between scene-seeding uploads and incremental presentation flushes
- separate frontend-object, gameplay-transient, and direct block-blit primitive families even if they ultimately target the same modern backbuffer
- preserve frontend text/highlight priority over floating menu objects rather than letting the object layer overdraw text
- preserve localized menu row rewrites and key-capture prompt/update sequencing instead of replacing them with broad full-menu redraws
- if using buffered presentation, enough surface synchronization to avoid stale page exposure after localized updates
- persistent music continuity across gameplay/frontend boundaries
- a separate temporary `GAME OVER` overlay layer with saved-under restore
- a frontend bootstrap shared by main menu, options, setup, credits, and high scores

## Confidence Notes

High confidence:

- snapshot timing relative to tracked transient restore
- baseline reset on frontend exit
- music continuity across `New Game`, `Return to Game`, and state `9`
- `GAME OVER` overlay save/restore behavior
- shared frontend bootstrap and exit-tail structure
- renderer primitive separation between frontend object pixels, tracked gameplay transients, and direct gameplay block blits
- gameplay present ordering around `0x17875 -> 0x2e18 -> 0x09c8 -> 0x206c -> 0x2f24 -> 0x17719 -> 0x24d0`
- frontend steady-state menu layering where text/highlight redraw wins over later floating object pixels

Still worth tightening later:

- runtime reachability proof (or disproof) for `0x1765a`; direct and extended static scans plus known-function callgraph reachability all currently keep it unreachable, but a live trace would fully close it

Capture-supported but still worth keeping scoped correctly:

- steady high-score-family screens clearly retain the shared title/logo base and chunk-7 object layer
- state-`9` bootstrap onset is now sub-second bounded from capture similarity (`~21.08s` crossover, `~21.33s` stronger dominance), but remains approximate rather than semantic per-frame scene labeling
- top-out slot-`5` versus state-`9` timing is now strongly bounded by capture-fit ranking, but still uses approximate frame-index seconds and mixed-bed audio matching rather than isolated stems
