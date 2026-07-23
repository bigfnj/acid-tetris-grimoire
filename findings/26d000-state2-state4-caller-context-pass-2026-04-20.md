# `0x26D000` State-2 vs State-4 Caller-Context Pass

Date: 2026-04-20

## Summary

This pass closed the caller-context meaning of the gameplay-facing dispatcher exits.

Main result:

- state `2` is **caller-dependent**
- state `4` is **caller-stable**
- the unique state-`4` fork through `0x2d88` and `0x05e0` is therefore still the strongest remaining unique static lead on the branch

That matters because the late flat-`0868` lane should now be compared against one stable fork, not against a state `2` path whose visible meaning changes by caller.

## New Owned Artifact

- [26d000-state2-state4-caller-context.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-state2-state4-caller-context.json)

This artifact records:

- the startup-side `0x3830` caller
- the session-loop-side `0x3830` caller
- what state `2` and state `4` mean in each context
- why state `4` remains the strongest unique split

## Key Artifacts Reused

- [26d000-gameplay-facing-exit-edge-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-gameplay-facing-exit-edge-pass-2026-04-20.md)
- [26d000-handler-return-state-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-handler-return-state-pass-2026-04-20.md)
- [ATET.EXE.flat-relocated.bin.00000018.FUN_00000018.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00000018.FUN_00000018.c)
- [ATET.EXE.flat-relocated.bin.0000023b.FUN_0000023b.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.0000023b.FUN_0000023b.c)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [ATET.EXE.flat-relocated.bin](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/disassembly/pmodew/extracted/flat/ATET.EXE.flat-relocated.bin)

## Findings

### 1. Startup Calls `0x3830` In A Different World Than The Session Loop

The startup path at `0x00000018` has two relevant frontend entries:

- optional early setup detour:
  - `0x177`
    `EAX = 5`
  - `0x17c`
    `CALL 0x3830`
- normal cold-start main-menu entry:
  - later startup continues with audio init, SFX loads, startup splashes, gameplay resource loads, and track start
  - only after that does it enter `0x3830(1)` for the normal main menu

So on the early setup detour:

- state `2` does **not** mean "resume live gameplay"
- it means "leave sound setup and return to startup continuation"

That matches the owned dispatcher hypothesis exactly:

- on the early startup state-`5` detour, state `2` restores the zero-filled pre-resource snapshot and hands control back to startup continuation

### 2. In The Live Session Loop, State `2` Means Resume, While State `4` Means New Game

The session loop path at `0x0000023b` reaches the frontend here:

- `0x4c8`
  optional finished-game-over cleanup through `0x23ac`
- `0x4d4` / `0x4d6`
  `EAX = 1` or `9`
  then `CALL 0x3830`

After `0x3830` returns, the loop immediately does:

- `0x4df`
  `0x184cb = 1`
- `0x4e5`
  `CALL 0x24d0`
- then resumes its normal gameplay update path

That means inside the live session loop:

- state `2`
  resumes the outer gameplay loop without calling `0x05e0`
- state `4`
  returns through the same caller, but only after the dispatcher tail has added:
  - `0x2d88`
  - `0x05e0`

So from the session loop perspective:

- state `2` = resume existing run snapshot
- state `4` = re-seed a fresh run before resuming gameplay flow

### 3. State `2` Is Structurally Ambiguous, State `4` Is Not

Across the two callers now owned:

- **state `2`**
  - startup caller:
    continue startup after early setup
  - session-loop caller:
    resume existing gameplay snapshot
- **state `4`**
  - session-loop caller:
    clear dirty-region map
    start new game session
    then continue through the gameplay reveal tail

That makes the state split asymmetric:

- state `2` is a shared dispatcher exit with caller-dependent meaning
- state `4` is a unique new-game fork with caller-stable meaning

This is the strongest current reason not to overweight state `2` as the late-lane candidate.

### 4. The State-`4` Fork Remains The Best Unique Candidate Surface

After the shared restore tail:

- state `2`
  immediately falls into the common gameplay-palette reveal and returns to its caller
- state `4`
  alone executes:
  - `0x2d88`
    clear dirty-region map
  - `0x05e0`
    start new game session

The important structural consequence is:

- if the late flat-`0868` lane belongs to a unique gameplay-facing handoff difference, the best current static candidate is still the state-`4` path

That is the only gameplay-facing branch we own that is both:

- unique
- semantically stable across caller contexts

## Practical Porting Impact

The frontend exit semantics are now precise enough to preserve directly:

- state `2` is a shared "continue / resume" exit whose exact effect depends on caller context
- state `4` is a true new-game exit with mandatory reinitialization work

That is a critical distinction for a faithful port.

## Next Strongest Move

Do a focused state-`4` fork pass:

1. trace the exact visible and data-side effects of `0x2d88`
2. trace the exact first-frame consequences of `0x05e0`
3. compare that state-`4` path against the state-`2` resume path only where they actually diverge

## Bottom Line

The important closure is:

- state `2` is caller-dependent, but state `4` is caller-stable

So state `4` remains the strongest unique static lead on the `0x26D000` branch.
