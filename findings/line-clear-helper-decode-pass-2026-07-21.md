# Line-Clear Helper Decode Pass

Date: 2026-07-21

## Summary

This pass decompiles the ten level-selected line-clear helpers
(`0x14b4`..`0x1c40`) and the two particle spawners they call (`0x2f78`,
`0x3034`), directly from the preserved flat-relocated binary
(`research/disassembly/pmodew/extracted/flat/ATET.EXE.flat-relocated.bin`, where
function address == file offset) using capstone. It unblocks the branch that
[line-clear-style-source-closure-pass-2026-04-20](line-clear-style-source-closure-pass-2026-04-20.md)
paused for lack of decompilation: the helper motion families, per-particle RNG
consumption, and the particle physics are now recovered and implementable.

## Dispatch

`0x1414` computes `current_level % 10` (via `[0x2c6df]`, idiv 10) and jumps
through the ten-entry table at `0x13ec` to one of:

`0x14b4 0x158c 0x1694 0x1770 0x1830 0x192c 0x19dc 0x1a9c 0x1b54 0x1c40`

(An eleventh slot dispatches to `0x1488`.) Each helper receives the cleared-row
index in `EAX`.

## Shared Helper Structure

Every helper walks the cleared row's 8-pixel-tall band as an outer loop of 8
pixel-rows and an inner loop of 80 columns (the 10-cell well is 80px wide):

- row screen offset = `row*8*320 + 0x1a40` (320 = mode width, `0x1a40` = well
  origin), advanced by `0x140` (320) per pixel-row.
- for each column it samples the current on-screen pixel from the board/screen
  buffer at `[0x2c727]` (this sampled palette index is the particle's color,
  stored as `pixel<<2` and later run through the chunk-3 four-stage ramps).
- it spawns one particle per column via `0x2f78` or `0x3034`.
- after each pixel-row it calls `0xe6b0` (fill 80 px) to blank the sampled row.

So a helper emits ~640 particles per cleared row and erases the row band.

## Particle Physics

### `0x2f78` — polar-velocity spawn

Inputs: `EAX` start position (column), `EBX` speed magnitude, `ECX` angle
(0..0x7ff), `EDX` start row; stack: lifetime, color byte.

- particle cap: `[0x2ca97]` must be `< 0x1000` (4096) or the spawn is dropped;
  slots come from a 4096-entry ring (`[0x2ca9f]` index, `[0x2ca9b]` pool).
- position: `[+4] = startX<<16`, `[+8] = startY<<16` (16.16 fixed-point).
- velocity is polar, using the sine table at `[0x2c6cf]` (period 0x800):
  - `[+0xc] = -(sine[(angle+0x200)&0x7ff] * speed) >> 7`  (X, cosine)
  - `[+0x10] = -(sine[angle&0x7ff] * speed) >> 7`  (Y, sine)
- `[+0x14] = 0` (age), `[+0x18] = 0x400 / lifetime` (decay rate),
  `[+0x1c] = color << 2`.

### `0x3034` — target-velocity spawn

Same pool/cap/decay/color, but velocity aims at a target point over `lifetime`
frames instead of a polar launch:

- `[+0xc] = (targetX - startX)<<16 / lifetime` (X)
- `[+0x10] = (targetY - startY)<<16 / lifetime` (Y)

This is the "target-point behavior" noted in the style-source closure: helpers
using `0x3034` converge/implode particles toward computed points.

## The Ten Helper Families

All emitters loop `col = 0..0x4f` (inner) and `row8 = 0..7` (outer); each column
draws its RNG (if any) and spawns one particle. Spawn args (`0x2f78`: EAX=startX,
EBX=speed, ECX=angle, EDX=startY; `0x3034`: EAX=startX, EBX=targetX, ECX=targetY,
EDX=startY; both push color then lifetime). Exact per-emitter parameters
(`life`=lifetime, angles/indices in the 0x800-period table):

| # | addr | spawn | draws/particle | angle / target | speed | life |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | `0x14b4` | polar | 3 | `rand&0x7ff` | `(rand&0x1ff)+0x80` | `(rand&0x3f)+0x10` |
| 1 | `0x158c` | polar | 2 | `row&1 ? 0x400 : 0` | `(rand&0x1ff)+0x80` | `(rand&0x3f)+0x10` |
| 2 | `0x1694` | polar | 2 | `0x188 + col*3` | `(rand&0x1ff)+0x80` | `(rand&0x3f)+0x10` |
| 3 | `0x1770` | polar | 0 | `0xc0 + col*8` | `0x100` | `0x40` |
| 4 | `0x1830` | target | 0 | target `(0x1f + col*3, 8*row+0xf)` | — | `0x40` |
| 5 | `0x192c` | target | 0 | target `(0x95, 0xd5)` (converge point) | — | `0x30` |
| 6 | `0x19dc` | polar | 0 | `0x600` (fixed dir) | `0x214 + col*4` | `0x30` |
| 7 | `0x1a9c` | target | 0 | target `(0x94, row*8+0x18)` (converge line) | — | `0x30` |
| 8 | `0x1b54` | polar | 0 | `col<0x28 ? 0 : 0x400` | `0x80` | `0x30` |
| 9 | `0x1c40` | polar | 0 | `col<0x28 ? 0x100 : 0x500` | `0xa0` | `0x30` |

Draw order for the RNG families is: `#0` life, angle, speed; `#1`/`#2` life,
speed (angle is deterministic/row-parity). The color byte is the sampled pixel
(pushed before the draws). Two families emerge: RNG-driven bursts (`#0`/`#1`/`#2`)
and fully deterministic sprays (`#3`-`#9`). RNG consumption is therefore
level-family dependent: only levels whose `%10` selects `#0`/`#1`/`#2` draw from
the shared gameplay random table during a clear (see
[rng-piece-selection-port-parity-pass-2026-07-21](rng-piece-selection-port-parity-pass-2026-07-21.md)),
at draws-per-particle x 640 columns x cleared-rows.

## Sine Table

The polar spawner uses the runtime-built table at `[0x2c6cf]` (2048 dwords,
generated at `0x27b8`): `sine[i] = round(sin(i * 2*pi/2048) * 32769)` — a full
signed-16-bit-amplitude sine over a 0x800 period. Velocity is
`v = -(sine[idx] * speed) >> 7` (X uses `idx=(angle+0x200)&0x7ff` = cosine, Y
uses `idx=angle&0x7ff` = sine), giving ~0.5-2.5 px/tick for the observed speeds.

## Port Implication (ready to implement)

The port's current `SeedLineClearDebris` is a modeled approximation (integer
`dx = (jitter%7)-3` scatter). A faithful implementation is now fully specified:

- one particle system with 16.16 fixed-point position, a velocity vector, an age,
  a `0x400/lifetime` decay, and a `pixel<<2` color into the chunk-3 ramps; cap
  4096.
- two spawn constructors: polar (`speed`,`angle` via a 0x800-period sine table)
  and target (`(target-start)*65536/lifetime`).
- ten helper emitters selected by `level%10`, sampling the cleared row's pixels,
  with the RNG-burst families drawing from the certified gameplay RNG in the
  exact 2/3/4-draws-per-particle order.

This has now been implemented in the port (`milestone_a_demo.cpp`):
`DebrisParticle` is a 16.16 fixed-point position + velocity with a
`0x400/lifetime` decay; `SpawnDebrisPolar`/`SpawnDebrisTarget` reproduce
`0x2f78`/`0x3034` (polar velocity uses the existing `sine_table_`, whose
amplitude is `0x10000` vs the original's `0x8001`, so the shift is `>>8` rather
than `>>7`); `SeedLineClearDebris` sweeps the cleared row's 8x80 band and
dispatches `EmitLineClearParticle` per column for the ten `level%10` families
with their exact parameters; the RNG-burst families draw from the certified
gameplay RNG in the exact per-particle order, and the pool caps at 4096.
Verified: builds clean, all five smokes `EXIT=0`, debris caps at 4096, a level-0
4-line clear consumes 7680 RNG draws (4x640x3), and the top-out fingerprint is
unchanged. The `#4`/`#7` target emitters are exact: their target-Y
(`8*row+0xf` and `8*row+0x18`) and target-X (`0x1f+col*3` and `0x94`) were
re-derived from `0x1830`/`0x1a9c` and match the port's mapping (the port's
`kGameplayClearX=0x6d` / `kGameplayClearY=0x15` origins equal the original's, so
start/target coordinates line up); levels 4 and 7 spawn debris and consume 0 RNG
during the clear, as expected for the deterministic families.

## Method Note

No Ghidra run was needed: the flat-relocated binary is preserved in the repo and
`FUN_xxxx` addresses equal its file offsets, so capstone
(`CS_ARCH_X86/CS_MODE_32`) disassembles any function directly. This is the
lightweight path for future decode passes when the Ghidra `.rep` DB is absent.
