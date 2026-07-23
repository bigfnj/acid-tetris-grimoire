# ACiD Tetris Behavior Spec

Date: 2026-04-20

This document is a port-facing behavior spec for the shipped game as currently understood from the bundled manual, owned captures, and reverse-engineering notes.

Provenance tags:

- `[Manual]` comes directly from `Original.Game/ATET.DOC`.
- `[Observed]` comes from owned captures under `Decompilation.Effort/captures/`.
- `[RE-derived]` comes from owned findings docs and executable tracing.

This spec is intentionally practical: it should let a reader who has never run the game describe the major screens, player-facing rules, and transition behavior.

## Controls

- `[Manual]` The main selection screen uses the arrow keys plus `Enter` to choose menu items, view high scores, view credits, enter keyboard setup, and change music selection.
- `[Manual]` On the main menu, moving to the `Music:` row and pressing `Enter` switches to the next song.
- `[Observed]` The default keyboard setup screen shows these gameplay bindings:
  - `Down -> Down`
  - `Left -> Left`
  - `Right -> Right`
  - `Rotate Left -> A`
  - `Rotate Right -> S`
- `[RE-derived]` The live gameplay loop reads exactly five configurable gameplay bindings from saved setup data: down, left, right, rotate-left, and rotate-right.
- `[RE-derived]` Selecting one of the first five rows in keyboard setup enters a capture mode where the next accepted key becomes the new binding for that action.
- `[RE-derived]` Left, right, rotate-left, and rotate-right act immediately on press, then repeat independently every 8 gameplay frames while held.
- `[RE-derived]` Down does not use the same repeat rule. It uses a separate soft-drop path plus a hold-lockout field that suppresses carried `Down` input across some line-clear and spawn transitions.
- `[RE-derived]` `Esc` during live gameplay exits to the main menu rather than quitting the program.
- `[RE-derived]` `Esc` after the finished game-over presentation hands control to the high-score qualification or name-entry flow rather than back to the ordinary main menu.

Primary sources: `Original.Game/ATET.DOC`; `captures/screenshots/03-keyboardsetup.png`; `frontend-state-map-2026-04-13.md`; `gameplay-helper-pass-2026-04-13.md`; `gameplay-input-timing-pass-2026-04-14.md`; `gameplay-escape-and-transient-render-pass-2026-04-14.md`.

## Scoring

- `[Manual]` The bundled manual does not explain the scoring formula. It only says the game is "pretty self explanitory."
- `[Observed]` The gameplay HUD shows `HIGH-SCORE`, `SCORE`, `LEVEL`, and `LINES`.
- `[Observed]` The high-score table shows player name, score, and a second numeric field that matches total lines cleared in the run.
- `[RE-derived]` The scoring table is:
  - `1 line -> 100`
  - `2 lines -> 300`
  - `3 lines -> 600`
  - `4 lines -> 1200`
- `[RE-derived]` The awarded score for a clear is `(current_level + 1) * base_clear_value`.
- `[RE-derived]` Total cleared lines are accumulated during the run and copied into the saved high-score record if the run qualifies.
- `[RE-derived]` During ordinary in-progress gameplay, owned redraw evidence updates `SCORE`, `LINES`, and conditional `LEVEL`, but not the gameplay HUD `HIGH-SCORE` field. `HIGH-SCORE` is bootstrap/static in the owned shipped gameplay paths unless new evidence appears.
- `[RE-derived]` The game keeps only five saved high-score entries.

Primary sources: `captures/screenshots/04-hiscores.png`; `captures/screenshots/07-gameplay.png`; `gameplay-helper-pass-2026-04-13.md`; `highscore-nameentry-and-helper-boundary-pass-2026-04-14.md`.

## Levels-And-Speed

- `[Manual]` The main menu exposes a `Level` row, which is the player's visible pre-game level setting.
- `[Observed]` The title screen capture shows `Level 0` as the selected displayed start level.
- `[RE-derived]` Pressing `Enter` on the `Level` row increments the stored starting level and wraps after `9`.
- `[RE-derived]` The live level increases after each additional 10 cleared lines.
- `[RE-derived]` Gravity is seeded as `(current_level << 9) + 0x200`, which gives a nominal natural descent interval of `128 / (level + 1)` gameplay frames per row.
- `[RE-derived]` A newly spawned piece starts with gravity accumulator `0`, so the first live gameplay step after a fresh spawn normally does not drop the piece by gravity yet.
- `[RE-derived]` The game uses 10 level-themed row-clear effect helpers chosen by `current_level % 10`, so level affects clear presentation as well as speed.

Primary sources: `Original.Game/ATET.DOC`; `captures/screenshots/01-title.png`; `gameplay-helper-pass-2026-04-13.md`; `piece-and-board-pass-2026-04-13.md`; `gameplay-input-timing-pass-2026-04-14.md`; `first-spawn-step-pass-2026-04-14.md`.

## Setup-Flow

- `[Manual]` First-time setup is performed by running `SETUP.BAT` or the provided Windows shortcut for setup. After that, the player runs `ATET.EXE` to play.
- `[Manual]` The manual describes the first-run screen as a sound-card selection screen and recommends `Autodetect` when appropriate.
- `[RE-derived]` Cold startup actually has two possible frontend phases:
  - optional early sound setup when explicit `setup` mode is requested or `SETUP.DAT` is missing or invalid
  - later normal main-menu entry after startup splashes and resource loads
- `[RE-derived]` Normal launch with valid setup follows this visible order:
  - `DDD` splash
  - warning splash
  - title and main menu
  - gameplay after `New Game`
- `[RE-derived]` Setup or first-run launch follows this visible order:
  - title-backed sound setup screen
  - if the user continues instead of exiting
  - `DDD` splash
  - warning splash
  - title and main menu
- `[RE-derived]` The first-run sound setup rows are:
  - sound device
  - mixing rate
  - stereo or mono
  - bit depth
  - `Play Game`
  - `Exit to Dos`
- `[RE-derived]` `Esc` in first-run sound setup means "leave setup and continue," not "exit to DOS."
- `[RE-derived]` Sound-setup edits rewrite the saved configuration values immediately in memory, but the screen does not live-preview or reinitialize the audio backend while the user is editing.
- `[RE-derived]` The startup backend init can fall back to device `None` if the configured backend fails, but the presentation flow still continues into the main menu.

Primary sources: `Original.Game/ATET.DOC`; `startup-sequence-correction-pass-2026-04-14.md`; `sound-setup-row-action-pass-2026-04-14.md`; `sound-setup-return-and-resume-fidelity-pass-2026-04-14.md`; `startup-path-predicate-pass-2026-04-17.md`.

## Menus

### Main Menu

- `[Observed]` The main menu sits under a large `ACiD TETRIS` logo on a black background, with a moving green chunk-7 decorative object behind the menu text.
- `[Observed]` The visible rows are:
  - `New Game`
  - `Options`
  - `Music:"..."`
  - `Level N`
  - `High Scores`
  - `Credits`
  - `Exit Game`
  - `Return to Game`
- `[RE-derived]` All eight rows are always drawn. `Return to Game` is visible even when no live game exists, but activation is ignored unless a resumable live game is active.
- `[RE-derived]` The cold-boot menu presentation is staged:
  - title and logo base appears first
  - floating chunk-7 objects fade in
  - the eight menu rows reveal afterward
  - only then does the steady selected-row pulse begin
- `[RE-derived]` The initial selected row on a normal cold boot is `New Game`.

### Options

- `[Observed]` The options screen shows four active rows:
  - `Music Volume`
  - `Sound FX Volume`
  - `Keyboard Setup`
  - `Back To Main Menu`
- `[RE-derived]` Options still uses the shared eight-row frontend scaffold; the four unused rows are blank padding rather than a different dialog type.
- `[RE-derived]` Row movement is release-gated, but value edits use held-state checks, so the screen feels like a menu with held controls rather than a static form.
- `[RE-derived]` Changing `Music Volume` applies immediately to the currently playing music.
- `[RE-derived]` Changing `Sound FX Volume` rewrites the stored value immediately, but its audible effect is only heard on later sounds; there is no dedicated live SFX preview helper.
- `[RE-derived]` Moving between option rows plays the normal menu navigation sound using the current SFX-volume setting.

### Keyboard Setup

- `[Observed]` The keyboard screen lists five bindings and a `Back to Options Menu` row.
- `[RE-derived]` Like options and sound setup, keyboard setup uses the shared eight-row scaffold with blank padded rows at the bottom.
- `[RE-derived]` The first five rows are binding-capture rows; the sixth returns to options.

### High Scores And Name Entry

- `[Observed]` The steady high-score screen keeps the shared title and logo base plus the floating green object behind the table text. The footer reads `Return to Main Menu`.
- `[Observed]` The capture shows five rows of names with score and line totals.
- `[RE-derived]` There are two related states:
  - state `8`: show the saved high-score table
  - state `9`: qualify a new score, insert it if needed, then allow name entry
- `[RE-derived]` Qualifying a score does not open an instant text field. The visible sequence is:
  - row-by-row reveal of the table
  - live row-local name entry for the inserted record
  - one-shot commit gate
  - footer exit gate
  - conceal back to the main menu
- `[RE-derived]` Name entry uses a local temporary buffer, caps visible names at 10 characters, supports backspace, and has a case-toggle behavior.

### Credits

- `[Observed]` Credits uses the same title-backed presentation family as the other frontend screens, but displays centered text pages instead of a selectable menu.
- `[Observed]` The captured pages include `Acid Tetris`, `Copyright 1997`, programming and graphics credits, and protected-mode extender credits.
- `[RE-derived]` Credits is a six-page fixed pager with eight lines per page, not a continuous scroll.
- `[RE-derived]` Each page's text is drawn once, then left in place while only the shared floating-object system and a page timer continue running.
- `[RE-derived]` `Esc` interrupts credits immediately and returns to the main menu using the animated frontend exit path.

Primary sources: `captures/screenshots/01-title.png`; `captures/screenshots/02-options.png`; `captures/screenshots/03-keyboardsetup.png`; `captures/screenshots/04-hiscores.png`; `captures/screenshots/05-credits.png`; `captures/screenshots/05.1-credits.png`; `captures/screenshots/05.2-credits.png`; `frontend-state-map-2026-04-13.md`; `main-menu-row-gating-pass-2026-04-14.md`; `cold-boot-menu-and-held-input-pass-2026-04-14.md`; `secondary-frontend-scaffold-and-live-audio-pass-2026-04-14.md`; `highscore-nameentry-and-helper-boundary-pass-2026-04-14.md`; `credits-raw-coverage-pass-2026-04-14.md`; `topout-exit-hiscore-menu-capture-pass-2026-04-14.md`.

## Gameplay-Loop

- `[Observed]` The main gameplay screen has a tall central well, a fixed `NEXT` preview housing at left, score and level counters below that preview, a fixed `LINES` band below the well, and a right-side piece-stat panel with fixed tetromino silhouettes plus live numeric counts.
- `[Observed]` The observed gameplay screen uses a themed background with decorative borders around the well and side panels.
- `[RE-derived]` The logical playfield is a 10x20 board.
- `[RE-derived]` Pieces use four orientations, and placement checks allow negative Y so a new piece may begin partly above the visible top of the well.
- `[RE-derived]` A new piece starts at board `X = 4`, `Y = 0`, `rotation = 0`, with gravity accumulator `0`.
- `[RE-derived]` The game keeps both a current piece and a next piece. Promoting the next piece also increments the per-piece statistics panel for the promoted piece type.
- `[RE-derived]` The fixed gameplay HUD labels and framing belong to the authored gameplay base screen: runtime gameplay helpers redraw the digits and the preview tetromino inside those fixed regions, but not the `NEXT` / `HIGH-SCORE` / `SCORE` / `LEVEL` / `LINES` text or the preview housing itself.
- `[RE-derived]` The fixed tetromino silhouettes in that right-side piece-stat panel belong to the authored gameplay base screen; runtime gameplay helpers redraw only the numeric counts on top of them.
- `[RE-derived]` The authored gameplay base screen also owns the decorative well frame; runtime gameplay helpers clear and repopulate only the `80x160` well interior at screen `x = 109`, `y = 21`, using palette-zero clears for empty cells and chunk-3 tetromino tiles for occupied cells.
- `[RE-derived]` Piece draw and piece erase are separate helpers. The game restores the old footprint before moving or rotating, then redraws the piece in its new position.
- `[RE-derived]` When one or more rows clear, the game:
  - records the clear count
  - updates score and total lines
  - conditionally updates level on ten-line rollover
  - runs the current level's row-clear visual helper, where level selects motion profile while sampled row pixels choose debris colors through shared chunk-3 ramps
  - enters a multi-frame collapse path that temporarily bypasses normal movement and spawn logic
- `[RE-derived]` Line-clear debris is intentionally dense, queue-capped, and allowed to overlap the collapse frames rather than being a tiny one-shot particle effect.
- `[RE-derived]` Rising stack height triggers escalating warning visuals and warning sounds before top-out. The highest occupied row determines which warning band is active.
- `[RE-derived]` Pressing `Esc` during live play snapshots the clean gameplay image, opens the main menu, and leaves the current music playing.
- `[RE-derived]` Choosing `Return to Game` restores the saved gameplay snapshot and preserved run state rather than starting a partial rebootstrap.
- `[RE-derived]` The first resumed gameplay-owned frame is usually an ordinary live step, but if pause happened during pending line-clear collapse or active top-out dissolve, it can resume into collapse-only or dissolve-only work before normal block blits return.
- `[RE-derived]` The first visible frame after `New Game` is already live in engine terms, but it usually shows the newly seeded scene rather than a gravity-dropped piece. Held movement or rotation can still affect that first frame.

Primary sources: `captures/screenshots/06-newgame.png`; `captures/screenshots/07-gameplay.png`; `gameplay-helper-pass-2026-04-13.md`; `piece-and-board-pass-2026-04-13.md`; `gameplay-input-timing-pass-2026-04-14.md`; `first-spawn-step-pass-2026-04-14.md`; `gameplay-escape-and-transient-render-pass-2026-04-14.md`; `transition-snapshot-and-music-continuity-pass-2026-04-14.md`; `line-clear-particle-overlap-pass-2026-04-15.md`; `line-clear-style-source-closure-pass-2026-04-20.md`; `gameplay-screen-asset-composition-pass-2026-04-20.md`; `gameplay-piece-stat-icon-ownership-closure-pass-2026-04-21.md`; `gameplay-hud-base-art-ownership-closure-pass-2026-04-21.md`; `gameplay-well-frame-ownership-closure-pass-2026-04-21.md`.

## Topout

- `[Observed]` The captured early game-over frame shows a large red `GAME OVER` overlay over the playfield and a yellow face graphic on the left side of the gameplay presentation.
- `[Observed]` The top-out capture confirms that the post-game-over path continues into the shared title and logo family before the high-score table fully settles.
- `[RE-derived]` The original top-out path is a staged sequence, not an instant jump from failed spawn to score table:
  - failed spawn triggers alert face `6` and the top-out sound
  - that failed-spawn frame can still show the colliding spawned piece plus the first visible alert reveal
  - the gameplay image dissolves row by row
  - a late saved-under `GAME OVER` overlay appears
  - the overlay is later removed
  - only then does the shared high-score bootstrap and reveal begin
- `[RE-derived]` The game-over overlay is a real saved-under overlay phase, not just text drawn as the first top-out frame.
- `[RE-derived]` After the finished game-over sequence, the session loop restores the overlay background and then hands control to the high-score qualification or entry state.
- `[RE-derived]` The high-score family later conceals and returns to the main menu rather than staying in a dead-end game-over screen.

Primary sources: `captures/screenshots/08-gameover.png`; `captures/video/topout-exit-hiscore-menu.mkv`; `topout-and-gameover-presentation-pass-2026-04-14.md`; `topout-exit-hiscore-menu-capture-pass-2026-04-14.md`; `gameplay-escape-and-transient-render-pass-2026-04-14.md`; `post-gameover-audio-tail-pass-2026-04-14.md`.

## Audio-Triggers

- `[Manual]` The bundled manual names six songs and tells the player to use the `Music:` menu row to cycle tracks.
- `[Manual]` The shipped track list is:
  - `Continuum`
  - `Tearing Up SpaceTime`
  - `Inner Walls Released`
  - `Costumed`
  - `I See It Now`
  - `Simple Song`
- `[Manual]` If the game feels sluggish, the manual recommends lowering the mixing rate and only disabling music as a last resort.
- `[Observed]` The capture set preserves six extracted music references that match the manual's credited songs.
- `[Observed]` The title capture shows the default visible track as `Continuum`.
- `[RE-derived]` Startup selects and starts the current music track before the main menu rows appear.
- `[RE-derived]` The current music track continues across:
  - `New Game`
  - `Return to Game`
  - the top-out to high-score handoff
  - return from high scores to the main menu
- `[RE-derived]` The only normal track-change actions are:
  - cold startup's initial selected track load
  - explicit `Music:` row activation in the main menu
- `[RE-derived]` Track changes go through the loader at `0x6544`, which stages the switch as a music-volume ramp: fade the current track out over `0x20` ticks (ramp mode `2`), wait for the ramp to finish, load the new track, then fade it in over `0x20` ticks (ramp mode `1`). The ramp system is `0x699e` (`schedule_music_volume_ramp(mode, duration_ticks)`), `0x69b0` (ramp-active query), and `0x6965` (per-tick step: clamp progress to the duration and scale linearly against the configured user music volume at `0x2c6eb`).
- `[RE-derived]` `Exit Game` is the audio event that schedules a fade-out before shutdown. The state-`3` exit path (`0x3830`) waits for any active ramp to finish, schedules a `0x40`-tick fade-out (mode `2`), runs the frontend fade-out, saves `SETUP.DAT`, then calls the final DOS-exit helper `0x05cc` — so the fade completes before the runtime shuts down.
- `[RE-derived]` High-confidence sound effects are:
  - menu row move -> slot `1`
  - low stack warning -> slot `3`
  - medium or high stack warning -> slot `4`
  - top-out -> slot `5`
  - sleepy no-clear timeout -> slot `6`
  - 1-line clear -> slot `9`
  - 2-line clear -> slot `7`
  - 3-line clear -> slot `7`
  - 4-line clear -> slot `10`
  - repeated back-to-back 4-line clears -> slot `11`
  - waking from the sleepy state with a 1-line clear -> slot `8`
- `[RE-derived]` All sound effects play through the mixer entry point `0x6817`, whose caller controls four parameters: `EAX` the slot index, `EDX` a coarse voice/playback-class offset (0 for UI, 1 for warnings), `ECX` a pan value where `0x80` is centered and offsets bias left/right, and `EBX` the saved user-facing SFX percent, which `0x6817` scales into the engine mixer range `0..0x40` before mixing. Known caller values: menu navigation (slot `1`) is centered, class `0`; the stack warnings (slots `3`/`4`) are centered, class `1`; the piece-lock/board-impact sound (slot `0`, chunk `15`, caller `0x0d4e`) is class `0` with a pan derived from the piece's board column around `0x80` (the exact column-to-pan curve is not recovered).
- `[RE-derived]` Warning sounds are cooldown-gated by stack-height band, and those cooldowns pause during line-clear collapse even though alert visuals continue aging.
- `[RE-derived]` The first-run sound setup screen is configuration-only. It rewrites saved settings but does not live-preview device changes or live-reinitialize the backend while the user edits rows.

Primary sources: `Original.Game/ATET.DOC`; `captures/audio`; `captures/screenshots/01-title.png`; `sound-event-mapping-pass-2026-04-14.md`; `transition-snapshot-and-music-continuity-pass-2026-04-14.md`; `alert-sound-trigger-cadence-pass-2026-04-15.md`; `audio-slot-call-closure-pass-2026-04-15.md`; `uni-music-conversion-closure-pass-2026-04-20.md`.
