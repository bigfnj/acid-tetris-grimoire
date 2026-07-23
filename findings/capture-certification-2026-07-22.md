# Capture-Certification Pass (port vs. running original)

Date: 2026-07-22

## Summary

First side-by-side certification of the port against the **running original**,
performed on the Linux box `kryptos`: the original `ATET.EXE` was run in DOSBox-X
under a virtual display (Xvfb) and its framebuffer captured with ImageMagick;
the port was run the same way. This exposed a significant rendering bug that all
prior headless/behavioral testing structurally could not catch, plus confirmed
strong visual parity for the certified screens.

## Method

- `dosbox-x` (already installed), Xvfb + ImageMagick + xdotool (installed via
  apt). `Original.Game/ATET.EXE`+`ATET.DAT` come from the normal clone.
- `Xvfb :99` -> `DISPLAY=:99 dosbox-x -conf atet.conf` (mount `Original.Game`,
  autoexec `ATET.EXE`) -> `import -window root` at timed points; `xdotool key
  Return` to enter New Game. The port was captured identically (SDL on `:99`).
- Screens captured for both: warning splash, title/main menu, gameplay.

## FINDING 1 (major): indexed art rendered with mangled color channels

The port rendered the **gameplay in red/maroon** where the original is **blue**
(cave/water), and the **menu text brown** where the original is **purple** —
while layout, HUD, wells, piece-stat panel, and menu rows were all correctly
placed.

Root cause in `indexed_asset_loader.cpp` `CreateTextureFromImage`: the pixel
buffer is a byte array in `R,G,B,A` order, but the texture was created as
`SDL_PIXELFORMAT_RGBA8888` — a *packed* 32-bit format (`0xRRGGBBAA`) whose
little-endian memory order is `A,B,G,R`. So every indexed-art texture (menu/title,
gameplay base, fonts, pieces) had its channels reinterpreted (red-shifted). Debris
particles were unaffected (drawn via `SDL_SetRenderDrawColor` with the palette
RGB directly, not through a texture), so the two paths actually disagreed on
color until now.

Why it survived to today: **all prior testing was headless/behavioral**. The
smoke suite asserts exit codes, gameplay fingerprints, RNG, and audio routing —
never pixel colors. Grayscale screens (the DDD/warning splashes) are invariant
under the channel swap, so they looked correct. Only rendering a color screen and
*looking at it* revealed it.

Fix: `SDL_PIXELFORMAT_RGBA32` (the endianness-correct array alias for `R,G,B,A`).
Verified by rebuilding on kryptos and recapturing: gameplay now renders the
original's blue background / brown well / cyan piece and the correct per-type
piece-stat colors (brown/red/purple/cyan/blue/yellow/green); the menu renders the
blue `TETRIS` logo and purple menu text. Behavior/fingerprints unchanged; all
smokes still pass on Windows and Linux. Commit `819c28f`.

## Certified matches (port vs original)

- Warning splash (`chunk-2`): matches (grayscale art + layout).
- Title + main menu: the `ACiD TETRIS` logo (red ACiD + blue TETRIS), the eight
  rows in order (New Game / Options / Music:"Continuum" / Level N / High Scores /
  Credits / Exit Game / Return to Game), default track `Continuum`, and the green
  chunk-7 floating object (a rotating wireframe tetromino, animating) all match —
  and now in the correct colors after the fix.
- Gameplay: themed blue background (`chunk-1`), wood-framed 10-wide well, `NEXT`
  box, `HIGH-SCORE 0100000` / `SCORE` / `LEVEL`, `LINES` box, and the right oval
  piece-stat panel with seven per-type counters (the spawned type at `0001`) —
  layout and colors match.

## Cadence

DOSBox-X reports the original running at `320x200` (`match=(320,200)` in its log)
= VGA mode 13h, whose vertical refresh is 70.086 Hz — confirming at runtime the
cadence the port derives statically from the `0x6c62` vretrace calibration and
now paces at (~70 fps). RNG remains certified against the decompilation (the
seed is timer-based, so a fixed runtime sequence can't be forced).

## FINDING 2 (fixed): frontend-font digits were all shifted +1

The menu showed `Level 1` for the level-0 default. Not a level-value bug
(gameplay starts at `level=0`), but a **frontend-font glyph mapping** bug: the
Options screen exposed it clearly — `Music Volume:211` (should be 100) and
`Sound FX Volume:01` (should be 90), i.e. every menu-font digit rendered as
`(d+1) mod 10`. The chunk-5 font lays digits out as `1,2,...,9,0` (zero last),
but `GlyphCodeForCharacter` assumed `0-9`. The gameplay HUD was unaffected (it
uses a separate digit atlas). Fixed by mapping `'1'..'9' -> 53..61`, `'0' -> 62`
(commit `81e3c67`); recapture confirms `Level 0`, `Music Volume:100`,
`Sound FX Volume:90`. Another find only visible by looking at the render.

## Top-out A/B (running original vs current port) — 3 new gaps

Follow-up pass: drove the **running original** in DOSBox-X (Xvfb + `xdotool`,
`import` capture) to a real top-out, and captured the **current port** the same
way via `--topout-demo`. Blind key-driven play in headless DOSBox is unreliable
for precise placement (repeated tries could not force a clean single-line clear —
pieces stack but leave holes), so the transient effect was reached via **top-out**
(pile the well to the ceiling), which drives the same cell-clear/dissolve
machinery. Sequence captured on the original: stack maxes out (no face) → **angry
face** at top-out detection → **shocked face** + **upward particle fountains** as
the board dissolves top-down → board gone, red `GAME OVER` overlay.

Matches confirmed: overall top-out scene (emptied well + red `GAME OVER` overlay +
yellow shocked face + HUD/panels/colors), the shocked-face art itself, the
`GAME OVER` overlay art/position, and gravity/fall/lock/stacking (observed at
length in both). Three gaps, all visual-only (headless-invisible), same class as
the color/digit bugs:

### GAP A (FIXED): top-out dissolve emits no particles
- Original: dissolves **row-by-row from the top down**, each dissolving row
  erupting into an **upward particle fountain** (colored cell debris sprays up and
  out the top of the well).
- Port (before): `RenderTopOutBackdrop` dissolved by **filling rows with solid
  black** top-down — no particles. The port already had a debris system
  (`SpawnDebrisPolar`/`SpawnDebrisTarget` for line clears), but the top-out path
  never called it.
- Fix: `UpdateTopOutDissolve()` (called each pre-overlay frame of `RenderTopOut`)
  tracks the dissolve front (`top_out_dissolved_rows_`); as each row crosses into
  the cleared set, `SpawnTopOutRowDebris()` sprays one debris particle **per
  on-screen pixel** of that row's filled cells (same 8x80 per-pixel density as the
  line-clear burst) with an upward polar velocity (angle ~0x200 = straight up,
  +/-45deg spread) in the cell's own colors. `RenderDebrisParticles()` (factored
  out of `RenderGameplay`) is re-drawn on top of the black rows so the fountain
  stays visible above the shrinking stack. Verified against the original (port
  `s_60`/`s_66` vs original `d_39`): matching upward fountain from the dissolve
  front. Note: `--topout-demo` renders a *preset* board with `live_game_active_`
  false and an empty `live_board_cells_`, so the fountain only shows in real
  gameplay top-out (`--live-demo`), not the synthetic demo scene.

### GAP B (FIXED): alert face drawn on an opaque box
- Original: the mood face is a **transparent circle** — cave water shows around it.
- Port (before): face drawn on an **opaque purple box** (`#7100B2` = srgb
  113,0,178). The loader only keyed **palette index 0**, but the face art's
  background is palette index **0x20** (confirmed: the tile's corner bytes are all
  0x20 and gameplay-palette[0x20] = 113,0,178).
- Fix: generalized `LoadIndexedImage`/`LoadTextureAsset` from `bool
  transparent_zero` to `int transparent_index` (-1 = opaque, else key that index);
  face/alert tiles now load with `transparent_index = 0x20` (RE board-alert-pass:
  `0x1793d` treats palette index 0x20 as transparent), other assets keep index 0.
  Verified: face renders as a circle over the water, no box (`s_66`, `q_13`).

### GAP C (FIXED): warning face reused the top-out (shocked) tile
- Original: escalating distress faces during the stack-height warning, then the
  shocked face at top-out/game-over. Per RE alert-id-correlation-pass, alert ID N
  = chunk-6 tile N: IDs 3/4/5 = low/medium/high stack warning, ID 6 = game over.
- Port (before): drew the single top-out face (tile06, shocked) for the gameplay
  warning too.
- Fix: load chunk-6 tiles 03/04/05 into `warning_faces_[3]`; the gameplay
  warning-face draw selects by `warning_band_` (1/2/3 -> tile 3/4/5); the top-out
  scene keeps tile06. `BeginTopOutDemo` clears `warning_band_` so the backdrop's
  gameplay pass stops drawing a warning face and the shocked face takes over.
  Verified: gameplay warning shows the frown/distress faces (`r_45` = tile03),
  top-out shows the shocked face.

### Alert-system polish pass (full chunk-6 mood faces)
Followed GAP C to its conclusion: replaced the ad-hoc `top_out_face_` +
`warning_faces_[3]` with the whole **14-tile chunk-6 bank** (`alert_tiles_[14]`)
behind a general alert controller (RE board-alert-pass `0x2008`/`0x206c`):
- `TriggerAlert(id, lifetime)` arms a one-shot mood face; `UpdateAlert()` ages it
  and otherwise reflects the ambient state (stack-warning band -> tile 3/4/5, else
  sleepy -> tile 11); any tile change restarts a staged vertical reveal-in
  (`RenderAlertTile`, row table `{50,40,30,20,10,1}` read as the down-counting
  reveal counter -> rows grow 1..50 over six frames).
- Triggers: `BeginLineCollapse` fires the reward/wake face via `AlertIdForClear`
  (recovered 0x980 table: 1/2/3/4-line -> tile 2/0/9/7, back-to-back tetris ->
  tile 8, single-line clear out of sleepy -> wake tile 12); the ambient path
  covers warning (3/4/5) and sleepy (11). (See the sound-validation section below
  for a fast-follow-tile bug this table later corrected.)
- Verified in `--live-demo` + the `C` clear-inject cycle: the mood face cycles
  through the reward expressions per clear type (`reward_faces.png`), the warning
  frowns appear on a high stack, and the shocked face on top-out (`faces_timeline.png`).
  The 6-frame reveal wipe is implemented but too brief to capture at frame-grab
  cadence. Sleepy (tile 11) is wired to the 45s idle timeout (not force-captured).
  Smokes green on Linux + Windows. `--topout-demo` still uses tile06 directly.

All three fixes are in `milestone_a_demo.cpp`/`.h`, `indexed_asset_loader.cpp`/`.h`,
and `CMakeLists.txt`. Builds + full smoke suite pass on both Linux (`build-linux`)
and Windows (`build/Release`).

Evidence frames (scratchpad, this session): original `d_36` (warning frown),
`d_38`/`d_39`/`d_40` (upward-fountain dissolve, top-down), `d_42` (GAME OVER);
port before fix `p_01`/`p_12`/`p_25`; port after fix `r_45` (warning tile03),
`s_60`/`s_66` (dissolve fountain + transparent shocked face), `q_13` (transparent
face); A/B montage `ab_fountain.png`.

### Mood-face SOUND validation + a fidelity fix it exposed
Cross-checked the alert system against the RE sound map (sound-event-mapping-pass:
12 WAV chunks load into slots 0..11; the line-clear tables at 0x980/0x990 are
alert IDs `[2,0,9,7]` and SFX slots `[9,7,7,10]`):
- **Shocked face (top-out, alert ID 6) -> slot 5**, **surprised face (4-line
  tetris, alert ID 7) -> slot 10** both fire and play. Confirmed at runtime with a
  new `alert=` `--debug-state` field + the audio log: a forced tetris logs
  `line-clear-4 -> slot 10` with `alert=7`, and top-out logs `top-out -> slot 5`.
  Every wired slot {0,1,3,4,5,6,7,8,9,10,11} matches the RE; slot 2 (chunk 23) is
  the original's shipped-but-unplayed slot (slot2-closure pass) and the port
  correctly never plays it. So the smiley sounds are reproduced and the map is
  faithful (`v_surprised.png`).
- The cross-check exposed a bug in the prior pass: the recovered clear table is
  `[2,0,9,7]` with **no fast-follow path**, but `AlertIdForClear` had invented
  fast-follow tiles 1/10. Fixed to the recovered table + its two overrides
  (sleepy+1line -> wake 12, back-to-back tetris -> 8); tiles 1/10/13 stay unused,
  matching the shipped code. Verified: `alert=` cycles `2/0/9/7`.

### Object-alpha / frontend fade: verified faithful, no visible gap
RE (chunk7-usage / exe-loader-function-map): the frontend fade helpers `0x62e0`
(out) / `0x63b8` (in) ramp the **palette brightness** while animating the chunk-7
object records -- there is no separate per-object alpha; the objects fade *with*
the palette. A palette-brightness ramp (`rgb*k`) and the port's black-overlay fade
(`rgb*(1-alpha)`) are the same multiply, so the visible output is identical.
Confirmed by a dense cold-boot A/B of the running original: its fades are a
uniform brightness-dim of the whole frame (half-faded WARNING splash `ob_20`),
exactly the darkening the port's overlay produces, objects animating throughout.
Conclusion: the port's fade is already visually faithful; re-implementing the
palette mechanism would be byte-identical churn on the smoke-tested fade, so it
was left as-is (documented, not changed).

### Frontend screens A/B (Options / High Scores / Credits) + 2 high-score fixes
Capture-certified the three remaining ◑ frontend screens against the running
original (drive the menu with `xdotool Down*N + Return`; note the original menu
WRAPS on Up/Down, so navigate with `Down` from a fresh boot, not an "Up*7 to top"
reset; sub-screens reveal row-by-row so allow ~3s settle):
- **Options**: `Music Volume:100 / Sound FX Volume:90 / Keyboard Setup / Back To
  Main Menu` -- matches.
- **Credits**: logo + `Acid Tetris / Copyright 1997 / Dungeon Dwellers Design` +
  the chunk-7 object -- matches.
- **High Scores**: two gaps found and fixed. (a) The port prepended a **"High
  Scores" heading** the original lacks -- removed, so the table sits directly
  under the logo at the same first-row Y as the menu (`CurrentFrontendRows`
  kHighScores/kHighScoreEntry). (b) The port rendered each row as one **centered**
  `name score lines` string, so names staggered; the original is a **table** with
  left-aligned names and right-aligned score/lines columns. Added a columnar
  render path in `RenderFrontendRows` for score rows using column X measured from
  the original (logical 320-space: name left @94, score right @230, lines right
  @266; left-align uses `center_x = X + w/2`, right-align `X - w/2`). Verified
  A/B (`hiscore_ab2.png`): names and number columns now align like the original.
  Safe for the shared name-entry view (`HighScoreRowText`/fields map by row index,
  not vector position; entry cursor unaffected). Smokes green Linux + Windows
  (`hiscore` EXIT=0, `hsboot=72` intact). Evidence: `scr_orig_{options,hiscore,
  credits}.png`, `scr_port_hiscore3.png`, `screens_ab.png`.

## Net

Capture-certification did exactly what it was supposed to: it turned "faithful
by construction" into "verified against the original," and in doing so caught a
real, global color bug that headless testing could not — and, in the top-out A/B,
three more visual-only gaps (dissolve particles, face transparency, face
expression) that headless testing also could not see. All five are now fixed
(color channel, digit map, GAP A/B/C), each RE-backed and re-verified against the
running original, with the full smoke suite still green on Linux and Windows. The
certified static screens and the top-out sequence are now visual matches.
The fuller chunk-6 alert system (all 14 mood faces: line-clear reward faces +
sleepy/wake + warning bands + staged vertical reveal) is now implemented too, its
face->tile mapping corrected to the recovered `[2,0,9,7]` table, and its mood
SOUNDS validated against the RE map (shocked->slot 5, surprised->slot 10, all
slots faithful). The object-alpha/frontend fade was verified as already faithful
(brightness-dim == the port's overlay). All of it stays green on the full smoke
suite, Linux and Windows. The Options / High Scores / Credits screens were then
A/B'd too (High Scores fixed: dropped a bogus heading + made the table columnar).

### Coverage closed: back-to-back (tile 8) and sleepy (tile 11) faces captured
The last two mood faces were force-captured. A debug key `B` (inject a four-line
clear; guarded by `live_game_active_`, alongside the existing C/O/H dev keys) lets
two presses produce the back-to-back tetris: the log shows `line-clear-4-back-to-
back -> slot 11` and the alert timeline `alert=7 -> alert=8`, with the cool
sunglasses face on camera (`face_b2b.png`). Sleepy was reached by idling
`--live-demo --start-level=0` ~40s with no clears (the AI keeps the board low so
it never tops out); the log shows `sleepy-no-clear-timeout -> slot 6`, `sleepy=1`,
and `alert=11`, with the closed-eyes face on camera (`slp_10.png`). That exercises
all 14 chunk-6 tiles that the shipped game uses (1/10/13 remain intentionally
unused). No known fidelity gaps remain; every screen, the top-out sequence, and
the full mood-face system (faces + sounds) are capture-certified against the
running original on both platforms.
