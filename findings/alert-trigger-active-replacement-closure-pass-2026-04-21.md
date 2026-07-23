# Alert Trigger Active-Replacement Closure Pass

Date: 2026-04-21

## Summary

This pass closes one remaining ambiguity inside the board-alert trigger helper `0x2008`:

- what exactly happens when a new alert request arrives while another alert is still active

Main result:

- the fresh-start path in `0x2008` does **not** draw immediately
- it only stores:
  - requested effect ID
  - requested lifetime
  - reveal counter `6`
- the active path stores the requested effect ID **before** the full draw
- so both:
  - same-effect refresh
  - different-effect replacement while active
  immediately draw the **requested** tile full-frame through `0x1793d`
- that active path still does **not** reseed the six-step reveal counter

So the stronger preservation rule is now:

- new idle-start alerts enter through staged reveal
- active replacements swap to the new requested tile immediately at full height
- same-effect refreshes keep the current tile visible while only refreshing lifetime

## Why This Pass Was Needed

The older alert lifetime model was already close, but one phrase stayed too blurry:

- "refresh or replacement while active redraws the full current tile"

The raw helper export at:

- [raw-2008-2294.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/alert-particle-edge-pass/raw-2008-2294.asm:1)

is now specific enough to tighten that wording.

The important instruction order is:

1. `mov esi, eax`
2. `mov [0x2c76f], eax`
3. `call 0x1793d`

That means the active path draws the requested effect, not the previously active one.

## New Owned Artifact

- [alert-trigger-active-replacement-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/alert-trigger-active-replacement-closure.json)

## Updated Living Artifacts

- [alert-and-tracked-particle-lifetimes.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/alert-and-tracked-particle-lifetimes.json)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [transition-preservation-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/transition-preservation-spec.md)

## Findings

### 1. The Fresh-Start Path Seeds State But Does Not Draw Immediately

The idle-start branch is:

- `cmp eax, [0x2c76f]`
- if different
- `cmp [0x2c75f], 0`
- if lifetime is `0`
- store:
  - `[0x2c76f] = eax`
  - `[0x2c75f] = edx`
  - `[0x2c767] = 6`
- return directly

There is no call to:

- `0x1793d`
- `0x17983`
- `0x2d30`

inside that branch.

So `0x2008` itself does not make a fresh idle-start alert visible immediately.
It only stages the state that `0x206c` will later reveal during the gameplay-owned update step.

### 2. Active Same-Effect Refresh Draws The Requested Tile Fully And Refreshes Lifetime

When:

- `eax == [0x2c76f]`

the helper enters the active path directly.

That path:

- copies `eax` into `esi`
- stores `eax` back into `[0x2c76f]`
- calls `0x1793d`
- marks the alert block dirty through `0x2d30`
- stores the new requested lifetime into `[0x2c75f]`

So a same-effect refresh:

- keeps the same effect ID
- redraws the same tile fully right now
- refreshes lifetime
- does not restart the six-step reveal

### 3. Active Different-Effect Replacement Also Draws The Requested Replacement Tile Fully

When:

- `eax != [0x2c76f]`
- but `[0x2c75f] != 0`

the helper still enters the same active path.

Because it stores:

- `[0x2c76f] = eax`

before:

- `call 0x1793d`

the full draw is already using the new requested effect ID.

So an active replacement is **not**:

- redraw old tile now, swap later

It is:

- store new requested effect now
- full-draw the new requested tile now
- refresh lifetime now
- keep the old reveal-counter state without reseeding it

That is the exact closure this pass needed.

### 4. The Reveal Counter Rule Is Now More Specific

The active path never writes:

- `[0x2c767]`

So the reveal-counter rule is now:

- fresh idle-start:
  - seed reveal counter `6`
- active same-effect refresh:
  - do not reseed reveal counter
- active different-effect replacement:
  - do not reseed reveal counter

Practical consequence:

- a replacement alert can appear immediately at full height even if the old alert had partially or fully completed its reveal

### 5. The Update Helper Still Owns The First Visible Sliver Of A Fresh Idle-Start Alert

This pass does **not** change the already-closed `0x206c` rules.

What it sharpens is the boundary between the two helpers:

- `0x2008`
  stages fresh idle-start state only
- `0x206c`
  owns the first visible reveal-step draw for that staged start

That distinction matters for frame-order fidelity:

- fresh-start alert visibility belongs to the gameplay step's later alert-update stage
- active refresh/replacement visibility can happen immediately in the trigger helper itself

## Practical Porting Impact

The future port should preserve three trigger modes for board alerts:

1. fresh idle-start:
   - store requested effect
   - store lifetime
   - seed reveal counter
   - no immediate full draw
2. same-effect refresh while active:
   - full-draw the requested tile immediately
   - refresh lifetime
   - keep reveal counter as-is
3. different-effect replacement while active:
   - switch to the new effect immediately
   - full-draw the requested replacement tile immediately
   - refresh lifetime
   - keep reveal counter as-is

That is a materially better fidelity rule than the older shorthand about "full current tile."

## Next Ordered Step

- pause this `0x2008` replacement-semantics branch unless one of these becomes newly useful:
  - a capture pass wants alert swaps timed against one gameplay-owned frame
  - a port milestone wants the alert subsystem translated into explicit code-state transitions
  - another alert-side edge case becomes the next strongest bounded target

## Bottom Line

The important closure is:

- active alert replacement in `0x2008` swaps to and full-draws the requested new tile immediately; it does not redraw the old tile first and it does not restart staged reveal
