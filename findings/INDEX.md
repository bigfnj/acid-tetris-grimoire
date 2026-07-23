# Findings Index

One-line takeaway per findings pass, grouped by subsystem. Entries sorted chronologically within each group (oldest first; latest state is at the bottom of each section).

## Exe Structure & RE Foundation

- [exe-structure-pass-2026-04-13](exe-structure-pass-2026-04-13.md) — DOS MZ declares 0x2CB0 bytes; actual file larger; resolved graphics chunks match executable decode models.
- [exe-loader-function-map-2026-04-13](exe-loader-function-map-2026-04-13.md) — ATET.DAT chunk table loaded at runtime; `0x3370` is DAT stdio loader; palette/page-flip/video-mode helpers identified.
- [prior-attempt-review-2026-04-13](prior-attempt-review-2026-04-13.md) — Prior project hypotheses useful but not authoritative; 4-mode RLE verified; chunk-7 RLE confirmed.
- [capture-review-2026-04-13](capture-review-2026-04-13.md) — Captured media covers menus, keyboard setup, seeded high-score, gameplay HUD, and intro/music evidence.
- [palette-and-graphics-pass-2026-04-13](palette-and-graphics-pass-2026-04-13.md) — Palette export produces six candidates; chunk 7 visualization confirms not plain bitmap; RLE compression confirmed.
- [engine-runtime-structure-pass-2026-04-14](engine-runtime-structure-pass-2026-04-14.md) — Startup seeds buffers, object pool, random table, graphics/input handlers, then audio and resources.
- [whole-program-function-map-density-expansion-pass-2026-04-21](whole-program-function-map-density-expansion-pass-2026-04-21.md) — Added the helper/runtime cluster to the living function map; resolved edge count rose and unresolved call count fell materially.
- [runtime-vector-helper-density-pass-2026-05-15](runtime-vector-helper-density-pass-2026-05-15.md) — Named the DOS interrupt-vector, heap-free, and exit-callback helpers around `0xeb58`; unresolved rel32 calls fell from `1298` to `1251`.

## Startup / Boot Path

- [startup-init-pass-2026-04-13](startup-init-pass-2026-04-13.md) — Chunk 2 is full-screen bundle like chunk 0; SFX slots loaded first; startup path now much clearer.
- [startup-direct-gameplay-pass-2026-04-14](startup-direct-gameplay-pass-2026-04-14.md) — Early conditional state-5 entry real but not only startup frontend; later unconditional state-1 main-menu always runs.
- [startup-sequence-correction-pass-2026-04-14](startup-sequence-correction-pass-2026-04-14.md) — Two-stage startup: optional early sound-setup, then always normal main-menu after splash/resource/music setup.
- [startup-to-first-gameplay-present-pass-2026-04-14](startup-to-first-gameplay-present-pass-2026-04-14.md) — Dispatcher tail restores snapshot, mirrors to all pages, then `0x05e0` seeds fresh run before first present.
- [frontend-startup-gate-pass-2026-04-14](frontend-startup-gate-pass-2026-04-14.md) — Early state-5 sound setup only on setup-mode or missing SETUP.DAT; normal startup still reaches menu.
- [resume-and-startup-fine-tuning-pass-2026-04-14](resume-and-startup-fine-tuning-pass-2026-04-14.md) — Resume cleanup runs first; splash/title handoff is non-crossfade; early sound-setup return stays black.
- [startup-path-predicate-pass-2026-04-17](startup-path-predicate-pass-2026-04-17.md) — Pre-main-menu startup path bounded to small concrete predicate family.

## Chunk 3 (Gameplay Resources)

- [startup-and-chunk3-resolution-pass-2026-04-14](startup-and-chunk3-resolution-pass-2026-04-14.md) — Chunk 3 is gameplay resource pack: 0x400 color-ramp lookup, seven 8x8 piece tiles, ten 5x5 decimal glyphs.
- [particle-ramp-semantics-pass-2026-04-14](particle-ramp-semantics-pass-2026-04-14.md) — Chunk-3 ramp table keyed by on-screen palette index, not event ID; particle system is fundamentally color-driven.

## Chunk 7 (Animation Banks)

- [chunk7-usage-pass-2026-04-13](chunk7-usage-pass-2026-04-13.md) — Chunk 7 is a frontend animation bank: 0x38000 expands into eight 0x7000 slices.
- [chunk7-bank-analysis-pass-2026-04-13](chunk7-bank-analysis-pass-2026-04-13.md) — Eight 0x7000 banks each hold 1024 0x1c-byte records with authored and zeroed runtime fields.
- [chunk7-record-map-pass-2026-04-13](chunk7-record-map-pass-2026-04-13.md) — Chunk 7 expands into eight banks with source coordinates plus runtime projection/shading fields.
- [chunk7-render-correlation-pass-2026-04-13](chunk7-render-correlation-pass-2026-04-13.md) — Bank 0 is the default first morph target; initial geometry splits into two authored point families.
- [frontend-object-bank-identification-pass-2026-04-14](frontend-object-bank-identification-pass-2026-04-14.md) — Chunk 7 floating objects are banked dotted silhouettes (smiley, tetromino), not abstract pointfield.
- [frontend-object-transform-pass-2026-04-14](frontend-object-transform-pass-2026-04-14.md) — Floating objects undergo live rotation, drift, reprojection, reshading every step via state at 0x2d22b..0x2d247.
- [frontend-object-cycle-continuity-pass-2026-04-14](frontend-object-cycle-continuity-pass-2026-04-14.md) — Chunk-7 object system starts at boot and runs continuously across frontend states, not re-seeded per screen.

## Frontend: State Map & Dispatcher

- [frontend-state-map-2026-04-13](frontend-state-map-2026-04-13.md) — Dispatcher maps 11 states: main menu, options, keyboard, high-score view/entry, credits, sound setup, exit.
- [frontend-menu-resolution-pass-2026-04-13](frontend-menu-resolution-pass-2026-04-13.md) — Main menu rows resolved; options rows identified; keyboard setup raw disassembly recovered.
- [frontend-transition-motion-pass-2026-04-13](frontend-transition-motion-pass-2026-04-13.md) — Menu transition helpers wrap chunk-7 frame pump; entry/exit both run 0x30 frames with text-reveal parameter.
- [frontend-capture-correlation-pass-2026-04-13](frontend-capture-correlation-pass-2026-04-13.md) — Captures prefer mid-morph title-overlay renders, especially bank 3 progress 64 and bank 2 progress 32.
- [title-options-dynamic-sequence-pass-2026-04-13](title-options-dynamic-sequence-pass-2026-04-13.md) — Dynamic sequence renders added; raw best matches "pure title screen"; visible-only threshold reveals actual pointfield.
- [title-options-frame-correlation-pass-2026-04-13](title-options-frame-correlation-pass-2026-04-13.md) — All six title/options frames prefer bank 3 progress 64; second place bank 2 progress 32 consistently.
- [font-and-menu-text-pass-2026-04-13](font-and-menu-text-pass-2026-04-13.md) — Chunk 5 is 66 glyphs × 16×12 pixels with separate width table; text renderer uses 192-byte stride.
- [frontend-dispatcher-transition-pass-2026-04-14](frontend-dispatcher-transition-pass-2026-04-14.md) — Dispatcher saves gameplay snapshot, fades out, restores title base, fades in, then dispatches handlers in loop.
- [frontend-fade-transitions-2026-07-21](frontend-fade-transitions-2026-07-21.md) — Implemented the 64-tick screen fades at the gameplay<->frontend boundary + boot splashes (`0x64f0`/`0x6498`/`0x2998`); in-frontend nav does NOT fade (dispatcher loops). Staged Esc/New Game/Return with a fade-through-black controller; `--no-fade` + demo shortcuts stay instant. Observable via `fade=` in `--debug-state`; harness-guarded.
- [frontend-title-base-loader-pass-2026-04-14](frontend-title-base-loader-pass-2026-04-14.md) — `0x5f78` preloads title/logo from chunk 4 into working buffer before frontend, does not present yet.
- [frontend-presentation-sequencing-pass-2026-04-14](frontend-presentation-sequencing-pass-2026-04-14.md) — Frontend menus use gameplay-like structure: clear transients, run catch-up steps, redraw, flush, present.
- [frontend-secondary-state-pass-2026-04-14](frontend-secondary-state-pass-2026-04-14.md) — Keyboard setup, sound setup, credits, high-score all follow menu-family phase structure with action-mode state.
- [frontend-menu-layering-pass-2026-04-14](frontend-menu-layering-pass-2026-04-14.md) — Menu text/highlight redraw has visual priority over floating objects; text bands overwrite, objects empty-pixel-only.
- [frontend-mutable-rows-and-key-capture-pass-2026-04-14](frontend-mutable-rows-and-key-capture-pass-2026-04-14.md) — Mutable menu rows clear locally, update value, rewrite row immediately during steady loop, not deferred.
- [frontend-text-reveal-and-cursor-pass-2026-04-14](frontend-text-reveal-and-cursor-pass-2026-04-14.md) — Text reveal uses vertical resample; underscore cursor is shared glyph blink, not separate system.
- [frontend-shared-present-boundary-pass-2026-04-14](frontend-shared-present-boundary-pass-2026-04-14.md) — Menu transitions share same dirty-flush present boundary (0x17719, 0x24d0) as gameplay.
- [frontend-exit-and-music-ramp-pass-2026-04-14](frontend-exit-and-music-ramp-pass-2026-04-14.md) — State 0 is no-op redispatch; state 3 uses timed music-fade controller before exit, not generic leave-menu branch.
- [title-options-correlation-closure-pass-2026-04-20](title-options-correlation-closure-pass-2026-04-20.md) — Older bank 3 progress 64 winner closes as heuristic only; title/options branch now paused model-limited.
- [title-options-composite-boundary-correction-pass-2026-04-21](title-options-composite-boundary-correction-pass-2026-04-21.md) — Corrected missing Music row in local menu reconstruction; title/options branch now implementation-safe with capture-certification limits only.
- [title-options-screenshot-certification-pass-2026-04-21](title-options-screenshot-certification-pass-2026-04-21.md) — Screenshot branch now certifies a corrected options state, but still does not honestly certify the main-menu screenshot.

## Frontend: Menus & Screens

- [main-menu-row-gating-pass-2026-04-14](main-menu-row-gating-pass-2026-04-14.md) — Main menu has eight visible rows including "Return to Game"; that row only actionable when live-game flag set.
- [menu-pulse-and-highscore-row-animation-pass-2026-04-14](menu-pulse-and-highscore-row-animation-pass-2026-04-14.md) — Menu highlight and high-score row animation both use sine-driven reveal through `0x60cc` text renderer.
- [secondary-frontend-scaffold-and-live-audio-pass-2026-04-14](secondary-frontend-scaffold-and-live-audio-pass-2026-04-14.md) — Secondary states share eight-row scaffolds; options input is release-gated for movement and held for edits.
- [cold-boot-menu-and-held-input-pass-2026-04-14](cold-boot-menu-and-held-input-pass-2026-04-14.md) — Cold boot shows title/logo, then objects, then rows; release latch clears but held-key state persists.
- [credits-raw-coverage-pass-2026-04-14](credits-raw-coverage-pass-2026-04-14.md) — Credits is six-page fixed-text pager, not streaming text; once drawn per page, only chunk-7 objects and timer advance.
- [highscore-nameentry-and-helper-boundary-pass-2026-04-14](highscore-nameentry-and-helper-boundary-pass-2026-04-14.md) — High-score name entry is two-input row-local redraw loop; `0x1765a` likely internal cut code, not public entry.
- [highscore-prompt-blink-and-commit-pass-2026-04-14](highscore-prompt-blink-and-commit-pass-2026-04-14.md) — Cursor is underscore glyph, blinks via shared text renderer; commit persists name before row gate finishes.
- [highscore-name-entry-ownership-closure-pass-2026-04-21](highscore-name-entry-ownership-closure-pass-2026-04-21.md) — `0x50b0` now owns a closed helper family; `0x1765a` is no longer a local boundary question.
- [sound-setup-row-action-pass-2026-04-14](sound-setup-row-action-pass-2026-04-14.md) — Sound setup mutates in-memory config values, rewrites rows, does not live-reinitialize audio backend.
- [sound-setup-return-and-resume-fidelity-pass-2026-04-14](sound-setup-return-and-resume-fidelity-pass-2026-04-14.md) — Early sound-setup returns to pre-resource path; Return-to-Game resumes saved state without reinit.
- [secondary-frontend-exit-contract-closure-pass-2026-04-20](secondary-frontend-exit-contract-closure-pass-2026-04-20.md) — Secondary frontend closes as one dispatcher graph: nested menus, sound-setup exits, terminal viewers.

## Rendering / Present Boundary / Pages

- [capture-certification-2026-07-22](capture-certification-2026-07-22.md) — First port-vs-running-original side-by-side (DOSBox-X + Xvfb capture on kryptos). FOUND+FIXED a global color bug: indexed art used `SDL_PIXELFORMAT_RGBA8888` on an R,G,B,A byte array (LE reads it A,B,G,R) → everything red-shifted; fix `RGBA32` (commit `819c28f`). Now menu/gameplay match the original (blue bg, purple menu, correct piece colors). Confirmed 320x200/70Hz at runtime. Open minor: menu shows Level 1 vs original Level 0.
- [dirty-flush-backend-pass-2026-04-14](dirty-flush-backend-pass-2026-04-14.md) — `0x17719` is shared dirty-cell flush backend for both gameplay and frontend using 40×60 grid of 8×4 cells.
- [page-ring-and-dirty-propagation-pass-2026-04-14](page-ring-and-dirty-propagation-pass-2026-04-14.md) — Three-page VGA ring seeded at startup; `0x24d0` rotates in stable order; dirty value 3 propagates across frames.
- [renderer-primitive-separation-pass-2026-04-14](renderer-primitive-separation-pass-2026-04-14.md) — Four distinct writer families, not one generic; frontend pixels, tracked transients, direct blits.
- [screen-buffer-role-correction-pass-2026-04-14](screen-buffer-role-correction-pass-2026-04-14.md) — `0x2c69f` is saved gameplay snapshot; `0x2c6af` is title/menu base; `0x2c727` is live working screen.
- [screen-save-buffer-resolution-pass-2026-04-14](screen-save-buffer-resolution-pass-2026-04-14.md) — `0x2c69f` saves current working screen on dispatcher entry; `0xeb0c` copy direction proves save direction.
- [session-loop-artifact-and-helper-family-pass-2026-04-14](session-loop-artifact-and-helper-family-pass-2026-04-14.md) — Session loop outer artifact created; renderer primitives split into four writer families with different plotting rules.
- [session-loop-handoff-confirmation-pass-2026-04-14](session-loop-handoff-confirmation-pass-2026-04-14.md) — State-9 trigger directly confirmed from flat binary; released-Esc branch specialized by game-over state.
- [state9-handoff-and-render-boundary-pass-2026-04-14](state9-handoff-and-render-boundary-pass-2026-04-14.md) — Render/present boundary cleaner: `0x2938` full-page seed, `0x17719` incremental flush, `0x24d0` page-ring rotate.
- [gameplay-present-order-and-resume-delta-pass-2026-04-14](gameplay-present-order-and-resume-delta-pass-2026-04-14.md) — First resumed frame layers: transient cleanup, object advance, gameplay block work, alert animation, particle redraw.
- [resumed-frame-and-post-gameover-transition-pass-2026-04-14](resumed-frame-and-post-gameover-transition-pass-2026-04-14.md) — First resumed frame bounded by tracked-pixel cleanup, one particle update, one gameplay step, no long catch-up burst.
- [transition-snapshot-and-music-continuity-pass-2026-04-14](transition-snapshot-and-music-continuity-pass-2026-04-14.md) — Menu snapshots clean gameplay without frame-local overlays; first post-frontend gets reset timing, not catch-up burst.
- [new-game-visible-bootstrap-pass-2026-04-14](new-game-visible-bootstrap-pass-2026-04-14.md) — New Game restores gameplay snapshot into VGA, shows incomplete scene, first outer flush completes it.
- [raw-evidence-boundary-pass-2026-04-14](raw-evidence-boundary-pass-2026-04-14.md) — State-9 trigger medium-confidence (exe-strong, capture-light); dirty-3 writers confirmed; full-page bypass confirmed.
- [dirty-flush-queue-capacity-pass-2026-04-15](dirty-flush-queue-capacity-pass-2026-04-15.md) — Flush queue holds 2400 entries (one full dirty sweep); deterministic planar copy pattern with consistent caller pairing.
- [return-to-game-first-frame-closure-pass-2026-04-20](return-to-game-first-frame-closure-pass-2026-04-20.md) — Resume restores clean snapshot, preserves live run, then presents exactly one fresh gameplay-owned frame.
- [restart-scene-capture-normalization-pass-2026-04-21](restart-scene-capture-normalization-pass-2026-04-21.md) — Gameplay-space normalization now preserves restart evidence durably, but gameplay-video stills remain too noisy for frame-tight reveal certification.

## Palette & Splash / Fade

- [simple-palette-fade-pass-2026-04-14](simple-palette-fade-pass-2026-04-14.md) — Simple palette fades wait on tick, scale palette through `0x284c`, upload through `0x2574` (no redraw).
- [simple-palette-direction-correction-pass-2026-04-14](simple-palette-direction-correction-pass-2026-04-14.md) — `0x6498` fades from black to full; `0x64f0` fades from full to black (not reversed).
- [fullscreen-splash-loader-pass-2026-04-14](fullscreen-splash-loader-pass-2026-04-14.md) — `0x2998` loads full-screen splash: black palette, load chunk+palette, decompress, upload, fade-in/hold/fade-out.

## Audio / SFX / Music

- [audio-backend-service-pass-2026-04-14](audio-backend-service-pass-2026-04-14.md) — Audio stack layers as music-fade controller, IRQ service, six-slot scheduler, with mode-specific service init.
- [audio-device-and-slot-pass-2026-04-14](audio-device-and-slot-pass-2026-04-14.md) — Startup tries configured audio backend, then falls back to device-4 None; SFX stays separate from music.
- [audio-playback-parameter-pass-2026-04-14](audio-playback-parameter-pass-2026-04-14.md) — `0x6817` scales SFX volume, selects slot/voice, sets pan, maps mixer-channel offset for playback-class routing.
- [audio-caller-completion-pass-2026-04-14](audio-caller-completion-pass-2026-04-14.md) — Thirteen direct `0x6817` callers identified; keyboard setup contributes two missed navigation sounds at slots 1/0x80.
- [sound-event-mapping-pass-2026-04-14](sound-event-mapping-pass-2026-04-14.md) — Startup loads twelve WAV chunks into slots 0..11; gameplay copies explicit line-clear alert/sound/score tables.
- [post-gameover-audio-tail-pass-2026-04-14](post-gameover-audio-tail-pass-2026-04-14.md) — Slot 5 top-out/game-over fires once at spawn-fail; later dissolve/overlay/state-9 are audio-passive with natural tail.
- [slot2-resolution-pass-2026-04-14](slot2-resolution-pass-2026-04-14.md) — Slot 2 loaded at startup from chunk 23; no executable caller plays it; likely unused cut UI/impact cue.
- [audio-slot-call-closure-pass-2026-04-15](audio-slot-call-closure-pass-2026-04-15.md) — Slot 2 startup-loaded but unplayed in shipped executable; all runtime SFX funnels through `0x6817`.
- [alert-sound-trigger-cadence-pass-2026-04-15](alert-sound-trigger-cadence-pass-2026-04-15.md) — Warning sounds gate through three cooldown timers by stack height band; cooldowns pause during line-clear collapse.
- [persisted-audio-matrix-pass-2026-04-17](persisted-audio-matrix-pass-2026-04-17.md) — Persisted audio real steering family; mono-only best single-field mover at 0868:000178C2.
- [persisted-audio-focus-pass-2026-04-17](persisted-audio-focus-pass-2026-04-17.md) — Focused combinations did not beat global floor; showed which audio levers cancel each other.
- [uni-music-conversion-closure-pass-2026-04-20](uni-music-conversion-closure-pass-2026-04-20.md) — All six `UNI` chunks play as-is in `mikmod`; `openmpt123` rejects them and no MOD/IT converter is present.
- [slot2-sound-map-closure-pass-2026-04-20](slot2-sound-map-closure-pass-2026-04-20.md) — Slot 2 leaves unresolved status: shipped-but-unused chunk 23 cue, preserved but not a required SoundEvent.
- [slot2-audio-supersede-wording-cleanup-pass-2026-04-21](slot2-audio-supersede-wording-cleanup-pass-2026-04-21.md) — Slot 2 closure artifact now matches the real pre-pass drift: findings unresolved, live sound map already closed.
- [gus-and-module-stream-density-pass-2026-05-15](gus-and-module-stream-density-pass-2026-05-15.md) — `0xb329` is GUS driver infrastructure and `0x84bf` is a bounded module-command stream reader; unresolved rel32 calls fell from `1251` to `1072`.
- [music-ramp-port-parity-pass-2026-07-21](music-ramp-port-parity-pass-2026-07-21.md) — Port now reproduces the RE-derived music ramp: track change fades out/in `0x20` ticks each (`0x6544`), `Exit Game` fades out `0x40` ticks then quits (state `3`); verified via new `music_ramp` `--debug-state` fields.
- [sfx-mixer-port-parity-pass-2026-07-21](sfx-mixer-port-parity-pass-2026-07-21.md) — Port now models `0x6817`: SFX volume scaled into engine `0..0x40`, per-play pan rendered as L/R via mono->stereo mixdown, coarse voice class carried; the missing slot-`0` piece-lock sound is wired with column-derived pan (center->118, right->181).

## Alerts & Particles

- [alert-id-correlation-pass-2026-04-13](alert-id-correlation-pass-2026-04-13.md) — Alert effect ID N maps directly to chunk-6 tile N, confirmed by executable indexing and capture evidence.
- [alert-id13-usage-closure-pass-2026-04-20](alert-id13-usage-closure-pass-2026-04-20.md) — Alert ID 13 is the only ID absent from owned shipped direct `0x2008` trigger paths.
- [alert-id1-shared-callsite-confirmation-pass-2026-04-20](alert-id1-shared-callsite-confirmation-pass-2026-04-20.md) — Shared `0x0f9f` proves shipped alert ID 1 beside ID 10 in the fast-follow-up branch.
- [alert-trigger-active-replacement-closure-pass-2026-04-21](alert-trigger-active-replacement-closure-pass-2026-04-21.md) — Active `0x2008` replacement stores and full-draws the requested new tile immediately; only fresh idle-start uses staged reveal.
- [board-alert-pass-2026-04-13](board-alert-pass-2026-04-13.md) — Alert tile is fixed 50×50 gameplay-side UI at screen position x=20 y=140 with dedicated save/restore backing buffer.
- [alert-and-tracked-particle-lifetime-pass-2026-04-14](alert-and-tracked-particle-lifetime-pass-2026-04-14.md) — Alerts restore background on expiry; tracked particles overwrite directly and restore in reverse order.
- [alert-particle-coexistence-pass-2026-04-14](alert-particle-coexistence-pass-2026-04-14.md) — Alert tile and tracked particles coexist cleanly; frame order preserves layering without special cases.
- [alert-lifetime-under-lineclear-load-pass-2026-04-15](alert-lifetime-under-lineclear-load-pass-2026-04-15.md) — Line-clear collapse ages alerts while refresh suppression lets stack-warning alerts expire during debris.
- [line-clear-particle-overlap-pass-2026-04-15](line-clear-particle-overlap-pass-2026-04-15.md) — One cleared row emits 640 debris; four-line clears try 2560 inserts, so pool overflow drops the rest.
- [line-clear-style-source-closure-pass-2026-04-20](line-clear-style-source-closure-pass-2026-04-20.md) — Line-clear style closes as motion-by-level plus row-pixel color through shared chunk-3 ramps.
- [line-clear-helper-decode-pass-2026-07-21](line-clear-helper-decode-pass-2026-07-21.md) — Decoded all 10 helpers (`0x14b4`..`0x1c40`) + spawners `0x2f78` (polar `speed*(cos,sin)`) / `0x3034` (target `(tgt-start)/life`); 16.16 fixed-point, `0x400/life` decay, 4096 cap; RNG-burst (#0/#1/#2, 2-4 draws) vs deterministic (#3-#9). Unblocks the paused branch; port impl now specified. Done via capstone on the preserved flat binary (no Ghidra needed).
- [line-clear-helper-capture-coverage-pass-2026-04-20](line-clear-helper-capture-coverage-pass-2026-04-20.md) — Owned captures prove debris exists, but not enough to map individual line-clear helper families.
- [line-clear-helper-capture-normalization-pass-2026-04-21](line-clear-helper-capture-normalization-pass-2026-04-21.md) — Normalized gameplay stills preserve the branch better, but still do not separate the ten helper families honestly.
- [gameplay-screen-asset-composition-pass-2026-04-20](gameplay-screen-asset-composition-pass-2026-04-20.md) — Gameplay screen closes as chunk1 base plus chunk3, chunk6, and chunk8 overlays.
- [gameplay-piece-stat-icon-ownership-closure-pass-2026-04-21](gameplay-piece-stat-icon-ownership-closure-pass-2026-04-21.md) — Right-panel tetromino silhouettes close as fixed chunk1 base art; runtime redraws there are chunk3 digits only.
- [gameplay-hud-base-art-ownership-closure-pass-2026-04-21](gameplay-hud-base-art-ownership-closure-pass-2026-04-21.md) — Left HUD labels, preview housing, and bottom LINES band close as fixed chunk1 base art; runtime redraws there are chunk3 digits and preview piece only.
- [gameplay-well-frame-ownership-closure-pass-2026-04-21](gameplay-well-frame-ownership-closure-pass-2026-04-21.md) — The central well frame closes as fixed chunk1 base art; runtime clears and chunk3 piece tiles own only the 80x160 interior.

## Gameplay Core

- [gameplay-entry-pass-2026-04-13](gameplay-entry-pass-2026-04-13.md) — `0x05e0` starts a new run: reset stats, redraw HUD, refill random table, spawn piece, set live-game flag.
- [gameplay-helper-pass-2026-04-13](gameplay-helper-pass-2026-04-13.md) — `0x09c8` is main gameplay loop; lines stat is total cleared lines; line-clear score table is [100,300,600,1200].
- [piece-and-board-pass-2026-04-13](piece-and-board-pass-2026-04-13.md) — `0x12ac` collision tests; `0x10ec` draws piece; board is 10×20; four orientations; negative Y allowed for spawn.
- [line-clear-theme-pass-2026-04-13](line-clear-theme-pass-2026-04-13.md) — Line-clear dispatch keys off level%10; themed helpers queue particles before board compaction.
- [gameplay-input-timing-pass-2026-04-14](gameplay-input-timing-pass-2026-04-14.md) — Four independent 8-frame repeat timers for left/right/rotate; gravity is (level<<9)+0x200 fixed-point.
- [gameplay-frame-driver-pass-2026-04-14](gameplay-frame-driver-pass-2026-04-14.md) — `0x23b` is game session loop with fixed-step catch-up and direct `0x09c8` call; outer loop paces via `0x24d0`.
- [gameplay-escape-and-transient-render-pass-2026-04-14](gameplay-escape-and-transient-render-pass-2026-04-14.md) — Esc during play returns to state-1; Esc after game-over restores overlay underlay and enters state-9.
- [gameplay-edge-paths-pass-2026-04-14](gameplay-edge-paths-pass-2026-04-14.md) — First gameplay frame can clean transients, advance objects, redraw blocks, animate alert, then redraw particles.
- [first-spawn-step-pass-2026-04-14](first-spawn-step-pass-2026-04-14.md) — First gameplay step after spawn cannot drop piece; gravity threshold needs multiple steps even with held-Down override.
- [gameplay-hud-counter-label-closure-pass-2026-04-21](gameplay-hud-counter-label-closure-pass-2026-04-21.md) — Gameplay-edge HUD gap closes: `0x0ff2 = LINES`, `0x100c = SCORE`, nearby `0x0ead = LEVEL`.
- [high-score-live-redraw-closure-pass-2026-04-21](high-score-live-redraw-closure-pass-2026-04-21.md) — Gameplay HUD `HIGH-SCORE` is bootstrap/static in owned live play; no ordinary `0x09c8` redraw path found.
- [gameplay-resume-early-return-closure-pass-2026-04-21](gameplay-resume-early-return-closure-pass-2026-04-21.md) — Resume can legitimately hit collapse-only or dissolve-only `0x09c8` steps before the first present.
- [rng-piece-selection-port-parity-pass-2026-07-21](rng-piece-selection-port-parity-pass-2026-07-21.md) — Port RNG certified bit-exact vs decompilation: Watcom `rand` `0xeceb`, 8192-dword table fill `0x32b0`, consume+wrap-at-`0x1FFE` `0x3280`, piece `%7`; verified port==reference-model piece sequence per seed. Seed global `0x2d2a3` is non-deterministic entropy (no recovered fixed write).
- [frame-cadence-timer-decode-pass-2026-07-21](frame-cadence-timer-decode-pass-2026-07-21.md) — Frame pacer `0x24d0` = CRTC page-flip + wait `[0x2c607]` ticks/frame + 6-step catch-up; tick counter `0x2d2a3` (= the RNG seed) incremented by a variable-rate event-scheduled PIT timer (`0x69b6` divisor setter). No fixed static cadence constant exists (config/runtime-coupled); exact real-time cadence is capture-only.

## Input

- [input-system-pass-2026-04-14](input-system-pass-2026-04-14.md) — Custom keyboard ISR: gameplay reads pressed-state table; menus read the release latch.
- [input-system-branch-lever-pass-2026-04-17](input-system-branch-lever-pass-2026-04-17.md) — Input still real branch-sensitive family; no tested variant beat known global floor.

## Top-out / Game Over / State 9

- [topout-and-gameover-presentation-pass-2026-04-14](topout-and-gameover-presentation-pass-2026-04-14.md) — Top-out stages failed spawn, alert+slot 5, row dissolve, saved-under overlay, then state-9 entry.
- [topout-exit-hiscore-menu-capture-pass-2026-04-14](topout-exit-hiscore-menu-capture-pass-2026-04-14.md) — Owned transition capture covers gameplay, dissolve, overlay, state-9 bootstrap, and high-score; very high value.
- [state9-bootstrap-capture-support-pass-2026-04-14](state9-bootstrap-capture-support-pass-2026-04-14.md) — High-score family shares title/object presentation; capture supports shared-bootstrap model, not state-9 trigger proof.
- [state9-bootstrap-onset-timing-pass-2026-04-15](state9-bootstrap-onset-timing-pass-2026-04-15.md) — State-9 bootstrap onset at 21.33s in owned capture; crosses from game-over phase; state-9 steady by 22.5s.
- [topout-slot5-fit-vs-state9-pass-2026-04-15](topout-slot5-fit-vs-state9-pass-2026-04-15.md) — Slot 5 top-out one-shot fits 8.44s onset, 0.87s duration, ends 9.31s; state-9 bootstrap 21.33s; large gap.
- [state9-first-visible-frame-closure-pass-2026-04-20](state9-first-visible-frame-closure-pass-2026-04-20.md) — First visible state-9 frame closes as short shared-bootstrap interval, not instant settled table.
- [topout-first-visible-failed-spawn-closure-pass-2026-04-21](topout-first-visible-failed-spawn-closure-pass-2026-04-21.md) — Failed-spawn frame still presents preview/HUD churn, colliding spawned piece, and first alert reveal; dissolve starts on the next gameplay step.

## Untracked Helper Reachability (`0x1765a`)

- [untracked-pixel-helper-reachability-pass-2026-04-14](untracked-pixel-helper-reachability-pass-2026-04-14.md) — `0x1765a` swaps pixel unconditionally with dirty-3 mark, no bounds checks; likely internal cut code.
- [untracked-helper-callgraph-reachability-pass-2026-04-15](untracked-helper-callgraph-reachability-pass-2026-04-15.md) — `0x1765a` unreachable from entry-point callgraph; neighboring primitives normally reachable.
- [untracked-helper-indirect-reachability-pass-2026-04-15](untracked-helper-indirect-reachability-pass-2026-04-15.md) — `0x1765a` not found in indirect calls, pointer slots, registers, far calls, or trampolines in full binary scan.
- [1765a-reachability-closure-pass-2026-04-20](1765a-reachability-closure-pass-2026-04-20.md) — `0x1765a` is specific enough to pause as a documented non-required primitive.
- [1765a-closure-propagation-pass-2026-04-20](1765a-closure-propagation-pass-2026-04-20.md) — `0x1765a` closure now propagates through renderer/spec artifacts; no active required-family gap remains.

## Runtime Debugger Tooling (DOSBox-X / WSL / Win64)

- [runtime-address-probe-pass-2026-04-15](runtime-address-probe-pass-2026-04-15.md) — DOSBox-X runtime address-trace channel not detected in current build mode; executable activity confirmed.
- [runtime-debugbox-command-availability-pass-2026-04-15](runtime-debugbox-command-availability-pass-2026-04-15.md) — Linux DOSBox-X package lacks DEBUGBOX command; Win64 portable provides it.
- [runtime-debugger-capability-pass-2026-04-15](runtime-debugger-capability-pass-2026-04-15.md) — DOSBox-X debugger surface not engaged in tested mode; `-break-start` and trace files not produced.
- [runtime-win64-debugbox-command-pass-2026-04-15](runtime-win64-debugbox-command-pass-2026-04-15.md) — Win64 DOSBox-X portable reports DEBUGBOX command with debugger-entry purpose text.
- [runtime-win64-debugger-entry-gate-pass-2026-04-15](runtime-win64-debugger-entry-gate-pass-2026-04-15.md) — Break-start effective; debuggerrun=watch no trace; DEBUGBOX /C not accepted.
- [runtime-win64-debugger-command-injection-pass-2026-04-15](runtime-win64-debugger-command-injection-pass-2026-04-15.md) — Can land in debugger-start state on Win64; command injection not executing yet.
- [runtime-wsl-native-debugger-feasibility-pass-2026-04-16](runtime-wsl-native-debugger-feasibility-pass-2026-04-16.md) — Packaged Linux runtime insufficient; source-built debug build staged locally.
- [runtime-wsl-native-debugger-surface-pass-2026-04-16](runtime-wsl-native-debugger-surface-pass-2026-04-16.md) — Source-built Linux debug binary viable; LOGCPU.TXT captured but ATET executed.
- [runtime-wsl-protected-mode-breakpoint-pass-2026-04-16](runtime-wsl-protected-mode-breakpoint-pass-2026-04-16.md) — Native WSL can re-enter live ATET with VRT; stable protected-mode selector exposed.
- [runtime-wsl-protected-mode-setup-branch-pass-2026-04-16](runtime-wsl-protected-mode-setup-branch-pass-2026-04-16.md) — Setup-mode owned probe path; variants stayed later than plain-launch best floor.
- [runtime-wsl-protected-mode-earliest-object2-threshold-pass-2026-04-16](runtime-wsl-protected-mode-earliest-object2-threshold-pass-2026-04-16.md) — Narrow object-2 budget island near 0x3C0000; earliest landing 0868:000178B1.
- [runtime-wsl-protected-mode-late-object2-gate-pass-2026-04-16](runtime-wsl-protected-mode-late-object2-gate-pass-2026-04-16.md) — Best gate late-object2 gate; 0x178CE-0x1790C landing already past target helpers.
- [runtime-wsl-protected-mode-steering-axis-elimination-pass-2026-04-16](runtime-wsl-protected-mode-steering-axis-elimination-pass-2026-04-16.md) — Staged LOGC and LOGL did not improve earliest object-2 floor; current best unchanged.
- [runtime-wsl-protected-mode-external-steering-pass-2026-04-16](runtime-wsl-protected-mode-external-steering-pass-2026-04-16.md) — Menu routing and held-Down real branch levers; neither beat current best floor.
- [runtime-wsl-protected-mode-config-state-branch-pass-2026-04-16](runtime-wsl-protected-mode-config-state-branch-pass-2026-04-16.md) — Setup-state real startup branch; best overall floor unchanged at 0868:000178B1.
- [runtime-state-snapshot-pass-2026-04-17](runtime-state-snapshot-pass-2026-04-17.md) — Early object-1 window running under different selector family than later flat 0868 frontier.

## Config Schema

- [config-schema-closure-pass-2026-04-17](config-schema-closure-pass-2026-04-17.md) — Signature validity only proven startup gate; audio fields real runtime controls, not startup selectors.

## 0x26D000 Selector Investigation (Active)

Timeline of the post-VRT bridge selector chase. This is the currently active line of RE work.

- [branch-family-batch-harness-pass-2026-04-17](branch-family-batch-harness-pass-2026-04-17.md) — Selector split between bounded 0870 cold lane and flat 0868 frontier now reproducible.
- [cold-lane-bifurcation-isolation-pass-2026-04-17](cold-lane-bifurcation-isolation-pass-2026-04-17.md) — Inspection matters at 0x26C000; does not fully explain flat-side cluster above 0x26F000.
- [object2-entry-call-window-pass-2026-04-17](object2-entry-call-window-pass-2026-04-17.md) — Early object-2 entries sit inside clear/project/draw/flush frame family; record-local latch decisive.
- [selector-transition-breakpoint-refinement-pass-2026-04-17](selector-transition-breakpoint-refinement-pass-2026-04-17.md) — Late cliff happens inside selector 0868, not at selector transition itself.
- [boundary-orientation-repeat-pass-2026-04-17](boundary-orientation-repeat-pass-2026-04-17.md) — Seam does not follow simple inspected-vs-uninspected or first-vs-second ordering rule.
- [lower-upper-edge-boundary-isolation-pass-2026-04-17](lower-upper-edge-boundary-isolation-pass-2026-04-17.md) — Split exists at 0x26C800 and 0x26D000; converges on flat 0868 by 0x26D800.
- [upper-edge-crossover-stability-pass-2026-04-17](upper-edge-crossover-stability-pass-2026-04-17.md) — Upper edge stable flat 0868 cluster; conflicts prior fine-grained results, seam not one threshold.
- [0824-to-0868-emergence-mapping-pass-2026-04-17](0824-to-0868-emergence-mapping-pass-2026-04-17.md) — Selector crossover bounded to LOGC 0x260000-0x270000; lands in object 3, not early object 2.
- [fine-grained-0824-to-0868-crossover-bracket-pass-2026-04-17](fine-grained-0824-to-0868-crossover-bracket-pass-2026-04-17.md) — Selector crossover not smooth monotonic threshold; behaves like jittery upper-edge seam.
- [threshold-state-correlation-pass-2026-04-17](threshold-state-correlation-pass-2026-04-17.md) — Current threshold lane lands in frontend glyph draw helper 0x178b0, not object-2 entry.
- [pre-threshold-breakpoint-family-pass-2026-04-17](pre-threshold-breakpoint-family-pass-2026-04-17.md) — Object-1 pre-threshold window real but neither 0x03DAC nor 0x03DDC fired in tested budgets.
- [26d000-probe-session-progression-pass-2026-04-17](26d000-probe-session-progression-pass-2026-04-17.md) — Session progression not monotonic; external state can reopen bridge-adjacent object-1 families.
- [26d000-session4-anomaly-correlation-pass-2026-04-17](26d000-session4-anomaly-correlation-pass-2026-04-17.md) — Ordinary late-session progression insufficient to recreate rare bridge-side 0824 family.
- [26d000-dosbox-environment-freshness-pass-2026-04-17](26d000-dosbox-environment-freshness-pass-2026-04-17.md) — DOSBox env freshness alone does not restore bridge; signal looks like session progression.
- [26d000-rare-0008-reproduction-pass-2026-04-17](26d000-rare-0008-reproduction-pass-2026-04-17.md) — Known-positive replay failed; near-miss control succeeded, tied to rarer mixed opening family.
- [26d000-bridge-side-0824-reproduction-pass-2026-04-17](26d000-bridge-side-0824-reproduction-pass-2026-04-17.md) — Rare 0824 bridge family does not reopen under clean fresh-clone bridge isolation alone.
- [26d000-anomaly-seeded-replay-pass-2026-04-17](26d000-anomaly-seeded-replay-pass-2026-04-17.md) — Post-VRT delay is stronger bridge lever than exec-break; 0.2s delay seeded 0008:0000149A.
- [26d000-batch-position-permutation-pass-2026-04-17](26d000-batch-position-permutation-pass-2026-04-17.md) — Fallback does not require absolute final position; early slot-2 already flipped to object-1.
- [single-budget-26d000-carry-isolation-pass-2026-04-17](single-budget-26d000-carry-isolation-pass-2026-04-17.md) — Bounded fallback reproduced once; not inspect-specific, batch-position-dependent.
- [26d000-no-inspect-cluster-characterization-pass-2026-04-17](26d000-no-inspect-cluster-characterization-pass-2026-04-17.md) — Pure no-inspect depth alone not sufficient for object-1; 0008 family needs refined model.
- [26d000-inspect-after-inspect-0008-refinement-pass-2026-04-17](26d000-inspect-after-inspect-0008-refinement-pass-2026-04-17.md) — Opening no-inspect appears important for object-1 outcome; inspect-after-no-inspect not required.
- [26d000-opening-noinspect-object1-drift-pass-2026-04-17](26d000-opening-noinspect-object1-drift-pass-2026-04-17.md) — Live selector seam is bridge-slot drift problem, not pure final-probe problem.
- [26d000-bridge-slot-object1-drift-pass-2026-04-17](26d000-bridge-slot-object1-drift-pass-2026-04-17.md) — Shape B can place rare 0008 on bridge but unstable; Shape A drift stays pre-bridge.
- [26d000-0008-family-confirmation-pass-2026-04-17](26d000-0008-family-confirmation-pass-2026-04-17.md) — Selector 0008 not explained by inspect-slot or opening-slot rules; deeper family model needed.
- [26d000-shape-b-bridge-family-stabilization-pass-2026-04-17](26d000-shape-b-bridge-family-stabilization-pass-2026-04-17.md) — Shape B bridge decayed from 0008 to 0870 to flat 0868; not stable direct selector.
- [26d000-shape-b-carry-decay-isolation-pass-2026-04-17](26d000-shape-b-carry-decay-isolation-pass-2026-04-17.md) — Interruption did not reset Shape B carry-decay; effect probably not just in-process runtime.
- [26d000-shape-b-fresh-clone-reset-pass-2026-04-17](26d000-shape-b-fresh-clone-reset-pass-2026-04-17.md) — Freshness reopens early Shape B families non-deterministically; not file mutation.
- [26d000-shape-b-fresh-clone-order-sensitivity-pass-2026-04-17](26d000-shape-b-fresh-clone-order-sensitivity-pass-2026-04-17.md) — Simple batch slot position insufficient; remaining factor tied to DOSBox/process freshness.
- [26d000-post-vrt-delay-refinement-pass-2026-04-20](26d000-post-vrt-delay-refinement-pass-2026-04-20.md) — Useful bridge window real but discontinuous; 0.10 and 0.20 hit 0008, 0.15 fell back.
- [26d000-dense-post-vrt-window-confirmation-pass-2026-04-20](26d000-dense-post-vrt-window-confirmation-pass-2026-04-20.md) — Live window real but unstable; 0.18s shifted strongest lead to 0870:00000823.
- [26d000-delay-0p18-bridge-family-stabilization-pass-2026-04-20](26d000-delay-0p18-bridge-family-stabilization-pass-2026-04-20.md) — Bridge-side 0870 at 0.18 is real but sparse; nearby 0.175 and 0.185 stay flat.
- [26d000-delay-0p18-precursor-correlation-pass-2026-04-20](26d000-delay-0p18-precursor-correlation-pass-2026-04-20.md) — Across 13 exact 0.18 runs, no opening/second selector predicts bridge-side object 1.
- [26d000-bridge-slot-delay-decoupling-pass-2026-04-20](26d000-bridge-slot-delay-decoupling-pass-2026-04-20.md) — Varying only bridge-slot delay leaves all six runs flat 0868; pure bridge-local timing is insufficient.
- [26d000-determinism-floor-pass-2026-04-20](26d000-determinism-floor-pass-2026-04-20.md) — Same-fixture repeats showed 10-20% bridge-family variance; ladder now sits near the noise floor.
- [26d000-opening-second-delay-seeding-pass-2026-04-20](26d000-opening-second-delay-seeding-pass-2026-04-20.md) — Final coarse seeding stayed 0868-dominant; runtime ladder closes and hands off to static analysis.
- [26d000-caller-seam-static-pass-2026-04-20](26d000-caller-seam-static-pass-2026-04-20.md) — Shared frontend seam ends at 0x17719 -> 0x24d0; next static target is post-present caller tails.
- [26d000-post-present-caller-tail-pass-2026-04-20](26d000-post-present-caller-tail-pass-2026-04-20.md) — Post-0x24d0 tails either loop locally or rejoin 0x3830 shared fade-out and restore.
- [26d000-dispatch-state-return-pass-2026-04-20](26d000-dispatch-state-return-pass-2026-04-20.md) — 0x3ee4 and 0x3830 now close the late lane as a named state-return problem.
- [26d000-handler-return-state-pass-2026-04-20](26d000-handler-return-state-pass-2026-04-20.md) — Only main menu and sound setup can emit gameplay-facing exits `2` or `4`.
- [26d000-gameplay-facing-exit-edge-pass-2026-04-20](26d000-gameplay-facing-exit-edge-pass-2026-04-20.md) — Gameplay-facing exits redispatch through 0x3830; only state `4` has a unique post-restore fork.
- [26d000-state2-state4-caller-context-pass-2026-04-20](26d000-state2-state4-caller-context-pass-2026-04-20.md) — State `2` depends on caller context; state `4` stays the strongest unique fork.
- [26d000-state4-fork-first-frame-pass-2026-04-20](26d000-state4-fork-first-frame-pass-2026-04-20.md) — State `4` uniquely uploads a partial new-game view before later staged redraws finish.
- [26d000-state4-surface-vs-flat-0868-pass-2026-04-20](26d000-state4-surface-vs-flat-0868-pass-2026-04-20.md) — Flat `0868` is not the state-`4` all-pages upload; overlap starts only at deferred flush/present.
- [26d000-state4-first-gameplay-flush-pass-2026-04-20](26d000-state4-first-gameplay-flush-pass-2026-04-20.md) — First gameplay flush is mixed; state-`4` uniqueness is only the staged bootstrap slice.
- [26d000-state4-bootstrap-contributor-pass-2026-04-20](26d000-state4-bootstrap-contributor-pass-2026-04-20.md) — State-`4` bootstrap now resolves to exact counters, preview, piece stats, and alert reset.
- [26d000-state4-alert-split-pass-2026-04-20](26d000-state4-alert-split-pass-2026-04-20.md) — State-`4` alert reset is unique; later ordinary alert update is usually inert on a fresh run.
- [26d000-state4-bootstrap-layout-pass-2026-04-20](26d000-state4-bootstrap-layout-pass-2026-04-20.md) — Bootstrap image is now spatially grounded in captures and specific enough to pause.
- [26d000-state4-pre-live-piece-visibility-closure-pass-2026-04-21](26d000-state4-pre-live-piece-visibility-closure-pass-2026-04-21.md) — Completed bootstrap/no-piece image closes as offscreen-only; state-4 new-game seam now implementation-safe.
