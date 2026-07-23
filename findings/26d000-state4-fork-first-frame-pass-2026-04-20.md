# `0x26D000` State-4 Fork First-Frame Pass

Date: 2026-04-20

## Summary

This pass traced the exact first-frame consequences of the state-`4` new-game fork and compared them against the shared state-`2` resume path.

Main result:

- state `2` resumes a restored gameplay snapshot
- state `4` reinitializes a new run before returning to the same outer gameplay loop
- the strongest unique visible consequence of state `4` is the early full-page gameplay upload inside `0x05e0`, which happens before later HUD and piece-bootstrap redraws are fully flushed

That makes the state-`4` fork the strongest remaining unique behavioral surface on the branch.

## New Owned Artifact

- [26d000-state4-fork-first-frame.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-state4-fork-first-frame.json)

This artifact records:

- the shared state-`2` resume path
- the state-`4`-only work through `0x2d88` and `0x05e0`
- the first outer gameplay-loop consequences after `0x3830` returns

## Key Artifacts Reused

- [26d000-state2-state4-caller-context-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state2-state4-caller-context-pass-2026-04-20.md)
- [26d000-gameplay-facing-exit-edge-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-gameplay-facing-exit-edge-pass-2026-04-20.md)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [ATET.EXE.flat-relocated.bin.000005e0.FUN_000005e0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-entry-pass/ATET.EXE.flat-relocated.bin.000005e0.FUN_000005e0.c)
- [ATET.EXE.flat-relocated.bin.00002d88.FUN_00002d88.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-entry-pass/ATET.EXE.flat-relocated.bin.00002d88.FUN_00002d88.c)
- [ATET.EXE.flat-relocated.bin.0000023b.FUN_0000023b.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.0000023b.FUN_0000023b.c)

## Findings

### 1. State `2` Returns To The Outer Gameplay Loop With No New-Game Reinitialization

Inside the session-loop caller:

- `0x3830` restores the saved gameplay snapshot into the working screen and all three VGA pages
- state `2` does **not** call:
  - `0x2d88`
  - `0x05e0`
- after `0x3830` returns, the session loop does:
  - `0x4df`
    `0x184cb = 1`
  - `0x4e5`
    `0x24d0`
  - then resumes the normal gameplay outer loop

At the top of the next gameplay loop iteration, the already-owned session-loop path begins with:

- `0x48f`
  `0x17875`
  restore tracked particle pixels

So the shared resume path is:

- restore snapshot
- palette reveal
- return to session loop
- one normal gameplay-present cycle resumes from the prior run state

### 2. State `4` Adds Two Unique Operations Before Rejoining The Same Loop

Only state `4` adds:

- `0x2d88`
  clear the entire coarse dirty-region map at `0x1ad97`
- `0x05e0`
  start a fresh new game session

That new-game session does all of the following before control returns to the same session-loop caller:

- reset current score to `0`
- reset cleared-line count to `0`
- promote selected starting level into the live level
- clear the logical 10x20 board buffer through `0x1330`
- choose and stage piece state through `0x1348`
- reset gameplay repeat timers through `0x09c8(EAX=1)`
- recompute gravity and write it to `0x2c717`
- reset the alert tile through `0x206c(EAX=1)`
- set the live-game flag at `0x2c72b = 1`

So state `4` does not merely "resume and then tweak a few globals."
It constructs a fresh run state before the outer gameplay loop resumes.

### 3. The Strongest Unique Visible Consequence Is The Early Full-Page Upload In `0x05e0`

The key visual detail inside `0x05e0` is earlier than many later redraw helpers:

- it clears the board and preview rectangles in the working screen through `0x6274`
- then immediately uploads that partially reset gameplay image into:
  - `0x2c6ab`
  - `0x2c6bb`
  - `0x2c6bf`
  via `0x2938`

Only later in `0x05e0` and the resumed gameplay flow do the rest of the staged updates finish:

- HUD counter redraws
- preview piece redraw
- per-piece counter update
- alert reset
- later dirty flushes

This is why the existing owned hypothesis about restart visuals is structurally sound:

- a restart can briefly reveal the previous run's HUD around a newly cleared board and preview until the later dirty flush completes the new-run presentation

That early all-pages upload is a uniquely state-`4` surface.

### 4. State `2` And State `4` Converge Again Only After The New-Game Work Is Already Done

After state `4` finishes `0x2d88` and `0x05e0`, both paths converge again at:

- `0x6498`
  simple gameplay palette fade-in
- `0x2574`
  final palette upload
- return to the session loop
- `0x184cb = 1`
- `0x24d0`
- next outer gameplay iteration

So the comparison is now very clean:

- state `2`
  restored snapshot -> reveal -> resume old run
- state `4`
  restored snapshot -> clear dirty map -> build fresh run -> reveal -> resume new run

### 5. The Tracked-Pixel Restore Path Is Shared, Not State-`4`-Unique

One important non-difference:

- neither the shared exit tail nor the state-`4` fork currently shows a separate clear of the tracked restore queue
- the next outer gameplay iteration still begins at `0x17875`

So tracked transient cleanup is not the strongest unique state-`4` discriminator.
The unique discriminator is still the new-game reinitialization work itself, especially the early full-page upload.

## Practical Porting Impact

This gives a high-fidelity behavioral split for frontend exits:

- state `2`
  restore and resume
- state `4`
  restore, then rebuild a fresh run before resuming

That distinction needs to remain explicit in any faithful port.

## Next Strongest Move

Do a focused static pass on the unique state-`4` surfaces only:

1. isolate the early all-pages upload in `0x05e0`
2. isolate the subsequent staged HUD / preview / counter redraws that lag behind it
3. compare those uniquely state-`4` effects against the late flat-`0868` lane

## Bottom Line

The important closure is:

- the state-`4` fork is not just a logical flag difference; it has a unique early visible presentation path through `0x05e0`

That remains the strongest unique static lead on the `0x26D000` branch.
