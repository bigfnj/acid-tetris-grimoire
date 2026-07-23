# `0x26D000` Gameplay-Facing Exit-Edge Pass

Date: 2026-04-20

## Summary

This pass followed the only remaining gameplay-facing handler exits all the way into the shared dispatcher tail.

Main result:

- `run_main_menu` and `run_sound_setup_menu` do **not** jump directly into gameplay
- they stage `EAX = 2` or `EAX = 4`, return to `0x3830`, and let the dispatcher redispatch once more into the direct exit stubs
- only after that redispatch do states `2` or `4` enter the shared fade-out / restore tail

That gives us the tightest static handoff map on this branch so far.

## New Owned Artifact

- [26d000-gameplay-facing-exit-edge.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-gameplay-facing-exit-edge.json)

This artifact records:

- the exact handler-side staging sites for gameplay-facing exits
- the redispatch through `0x3830`
- the direct exit stubs for states `2` and `4`
- the split between shared tail work and state-`4`-only work

## Key Artifacts Reused

- [26d000-handler-return-state-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-handler-return-state-pass-2026-04-20.md)
- [26d000-dispatch-state-return-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-dispatch-state-return-pass-2026-04-20.md)
- [ATET.EXE.flat-relocated.bin](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/disassembly/pmodew/extracted/flat/ATET.EXE.flat-relocated.bin)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)

## Findings

### 1. `run_main_menu` Has Two Gameplay-Facing Exit Edges

The main menu has two distinct routes that can feed dispatcher exits:

- **New Game**
  - row `0`
  - jump-table entry `0x4290`
  - stages return state `4` into `[esp+0x4]`
  - arms one-shot exit mode through `EBP = 1`
  - later waits for `0x3fd8` one-shot completion
  - only then stores the exit flag and reaches `0x456f -> 0x3ee4`
- **Return to Game**
  - row `7`
  - jump-table entry `0x4402`
  - only active when live-game flag `0x2c72b == 1`
  - stages return state `2` into `[esp+0x4]`
  - also uses the same one-shot exit gate before `0x3ee4`

There is also a fast gameplay-facing interrupt:

- **Esc in main menu**
  - `0x4232 .. 0x425b`
  - only active when the live-game flag is `1`
  - stages return state `2`
  - clears the release latch through `0x94c`
  - skips straight to the end-of-frame counter path, but still reaches the same `0x3ee4` exit transition before returning

So the main menu emits:

- state `4` on New Game
- state `2` on Return to Game and gameplay-only Esc

### 2. `run_sound_setup_menu` Has One Gameplay-Facing Exit Class

The sound-setup handler feeds gameplay-facing control only through state `2`:

- **Play Game**
  - row `4`
  - jump-table entry `0x5da1`
  - stages return state `2` into `[esp+0x28]`
  - arms exit mode through `[esp+0x24] = 1`
  - later waits for the one-shot `0x3fd8` gate and stores the exit flag in `[esp+0x20]`
  - then reaches `0x5ee1 -> 0x3ee4`
- **Esc**
  - `0x5c0e .. 0x5c25`
  - stages the exact same return state `2`
  - uses the same exit-mode and later the same `0x3ee4` path

So sound setup never emits state `4`.
Its gameplay-facing exit is only:

- state `2`

### 3. Handlers Return `EAX = 2 / 4`, Then `0x3830` Redispatches One More Time

The crucial handoff is now explicit:

1. handler finishes its local exit transition through `0x3ee4`
2. handler returns with:
   - `EAX = 2`
   - or `EAX = 4`
3. `0x3830` executes:
   - `0x3936`
     `ECX = EAX`
   - `0x3938`
     test `ESI`
   - since the handler itself did not set `ESI = 1`, dispatcher control goes back to `0x38ae`
4. the redispatch lands on the direct exit stub:
   - state `2` -> `0x38c9`
   - state `4` -> `0x3906`
5. only those direct exit stubs set `ESI = 1`
6. `0x3938` then falls into the shared exit tail at `0x3941`

That means the gameplay-facing exit edge is a **two-stage** handoff:

- handler return state
- then dispatcher direct exit stub

not a direct jump from handler into gameplay restoration

### 4. State `2` And State `4` Share Most Of The Tail, But State `4` Has One Unique Fork

After redispatch:

- state `2`
  - stub `0x38c9`
  - writes row index `0x1886f = 7`
  - sets `ESI = 1`
  - enters the shared tail
- state `4`
  - stub `0x3906`
  - sets `ESI = 1`
  - enters the shared tail

The shared tail at `0x3941 .. 0x39bd` is then common until:

- clear release latch
- run frontend fade-out
- save `SETUP.DAT`
- restore saved snapshot into working and visible pages

Only **state `4`** then takes the extra new-game fork:

- `0x399a`
  `cmp ecx, 0x4`
- `0x399f`
  `0x2d88`
  clear dirty-region map
- `0x39a4`
  `0x05e0`
  start new game session

Then both state `2` and state `4` finish with:

- `0x6498`
  gameplay palette fade-in
- `0x2574`
  final palette upload

So if the late flat-`0868` lane belongs to a handoff difference between gameplay-facing exits, the strongest unique static candidate is now:

- the state-`4` new-game fork after `0x399a`

### 5. This Is The Best Remaining Static Split On The Branch

The branch surface is now much smaller than before:

- main-menu state `2`
- main-menu state `4`
- sound-setup state `2`
- dispatcher stub `2`
- dispatcher stub `4`
- shared tail
- state-`4`-only fork through `0x2d88` and `0x05e0`

That is the first point in this line of work where the surviving late-lane candidates are small enough to prioritize confidently.

## Practical Porting Impact

The gameplay-facing frontend exit model is now explicit:

- handler-local exit staging
- visible exit transition
- dispatcher redispatch
- shared restore tail
- optional new-game-only fork

That is a strong fidelity model for future port behavior.

## Next Strongest Move

Do a focused split pass on the shared tail versus the state-`4` fork:

1. trace exactly what state `2` does after snapshot restore
2. trace exactly what state `4` adds through `0x2d88` and `0x05e0`
3. use that split as the strongest static candidate surface for the dominant late flat-`0868` lane

## Bottom Line

The important closure is:

- the gameplay-facing exits are now reduced to a two-stage handler-return plus dispatcher-stub handoff, and only state `4` has a unique post-restore fork

That makes state `4` the strongest unique static lead on the branch.
