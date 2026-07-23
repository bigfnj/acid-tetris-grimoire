# `0x26D000` State-4 Alert Split Pass

Date: 2026-04-20

## Summary

This pass separated the state-`4` alert reset inside `0x05e0` from the later ordinary `0x206c` call in the first live gameplay step.

Main result:

- `0x206c(EAX = 1)` on the state-`4` path is a true visible bootstrap contributor:
  - restore alert background
  - clear active effect
  - zero warning-side cooldown state
- the later ordinary `0x206c` call in the first live step is usually a no-op on a normal fresh run
- it only becomes visibly different if the first real `0x09c8` step triggers a new alert in that same frame

So the alert side is now tighter:

- the clean alert-region reset belongs to the state-`4` bootstrap slice
- later alert motion belongs to ordinary gameplay edge cases, not to the default new-game bootstrap

## New Owned Artifact

- [26d000-state4-alert-split.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-state4-alert-split.json)

This artifact records:

- the exact reset behavior of `0x206c(EAX = 1)`
- the ordinary non-reset behavior of `0x206c`
- why the first live-step alert update is usually inert on a normal fresh run
- the narrow edge cases that can override that clean split

## Key Artifacts Reused

- [26d000-state4-bootstrap-contributor-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-contributor-pass-2026-04-20.md)
- [board-alert-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/board-alert-pass-2026-04-13.md)
- [alert-and-tracked-particle-lifetime-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/alert-and-tracked-particle-lifetime-pass-2026-04-14.md)
- [gameplay-edge-paths-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-edge-paths-pass-2026-04-14.md)
- [first-spawn-step-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/first-spawn-step-pass-2026-04-14.md)
- [ATET.EXE.flat-relocated.bin.0000206c.FUN_0000206c.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/startup-followup-pass/ATET.EXE.flat-relocated.bin.0000206c.FUN_0000206c.c)

## Findings

### 1. The State-`4` Alert Reset Is A Direct Visible Bootstrap Write

The reset call inside `0x05e0` uses:

- `0x206c(EAX = 1)`

Owned `0x206c` behavior for `EAX = 1`:

- clear active alert lifetime at `0x2c75f`
- clear reveal counter at `0x2c767`
- restore the saved alert background through `0x2138`
- set active effect id `0x2c76f = -1`
- zero the warning-side cooldown fields
- mark the alert region dirty through the restore path

That means the state-`4` path contributes one specific visible alert-region result to the staged bootstrap:

- clean restored alert background with no active icon

This is a real screen contribution, not just internal bookkeeping.

### 2. The Later Ordinary `0x206c` Call Usually Has Nothing Left To Do

After `0x3830` returns, the first live gameplay frame still executes its normal order:

- `0x2e18`
- `0x09c8`
- ordinary `0x206c`
- `0x2f24`
- `0x17719 -> 0x24d0`

But after the earlier state-`4` reset, the ordinary `0x206c` path sees:

- lifetime `0`
- reveal counter `0`
- active effect id `-1`

And the ordinary non-reset `0x206c` logic does:

- if lifetime is `0`
  - return without restore or reveal work

So on a normal fresh run, the first gameplay-step `0x206c` contributes no second alert-region change.

### 3. That Makes The Clean Alert Region Part Of The Unique State-`4` Slice

Because the later ordinary `0x206c` is usually inert, the default first-flush alert-region contribution belongs entirely to the earlier state-`4` reset.

So the alert side of the first gameplay-side flush now splits cleanly:

- state-`4`-unique default contribution:
  - clean restored alert background
- shared gameplay-edge contribution:
  - only if the first real gameplay step activates or refreshes an alert

This is better than the earlier generic wording "alert reset surface."
We now know what that surface really is.

### 4. The First Live Step Can Still Override The Clean Split In Specific Edge Cases

This pass does not claim the later ordinary `0x206c` is always irrelevant.

Owned edge cases still matter:

- `0x09c8` can trigger alert effect `6` on immediate spawn collision / top-out entry
- other gameplay-side alert triggers can refresh or start an alert if the first live step reaches them

In those cases, the first live-step `0x206c` is no longer inert:

- it can reveal or refresh a live alert icon in the same overall presented frame

But that is now clearly an edge-path override, not the normal fresh-run bootstrap.

### 5. The State-`4`-Unique First-Flush Slice Is Smaller Again

After this pass, the unique state-`4` staged bootstrap left dirty by `0x05e0` is even more explicit:

- `HIGH-SCORE`
- `SCORE`
- `LEVEL`
- `LINES`
- seven-piece statistics panel with one promoted-piece count at `1`
- next-piece preview
- clean alert-region restore

And the later shared first-frame work now excludes the default alert reset case.

That is the smallest still-visible state-`4` bootstrap slice we have owned so far.

## Practical Porting Impact

For the future port:

- the new-game bootstrap should explicitly restore the alert tile background as part of the staged restart image
- the first live gameplay-step alert update should usually do nothing on an ordinary fresh run
- only edge cases like immediate spawn failure should cause that later alert stage to visibly override the clean reset in the same frame

That is a subtle sequencing rule, but it is now solid enough to preserve.

## Next Strongest Move

Do a focused static pass on the remaining exact `0x05e0` bootstrap image:

1. tie the clean alert-region restore and the named HUD counters back to their fixed gameplay-screen panel layout
2. compare that fully named state-`4` bootstrap image against owned gameplay captures
3. decide whether the `0x26D000` static branch is now specific enough to pause in favor of another subsystem

## Bottom Line

The important closure is:

- the default alert contribution in the first gameplay-side flush after state `4` is the clean background restore from `0x206c(EAX = 1)`, while the later ordinary `0x206c` call is usually inert unless the first live step triggers a new alert
