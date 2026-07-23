# Highscore Name-Entry Ownership Closure Pass

Date: 2026-04-21

## Summary

This pass closes a stale local ambiguity around the high-score handler `0x50b0`:

- what `0x50b0` itself now owns directly
- what the nearby raw helper family no longer needs to imply about `0x1765a`

Current best closure:

- `0x50b0` now has a clean owned renderer contract through:
  - `0x17613`
  - `0x60cc`
  - `0x39c4`
  - `0x175c5`
  - `0x17719`
  - `0x24d0`
- `0x1765a` is **not** part of the required `0x50b0` ownership model
- the live high-score artifact should no longer frame `0x1765a` as a remaining local helper-boundary question

So the high-score screen is now specific enough to preserve directly without carrying a stale renderer-side caveat.

## Why This Pass Was Needed

The project already had strong visible-behavior closure for `0x50b0`:

- qualification insert
- reveal
- live name entry
- row commit
- footer exit
- conceal

But the machine-readable artifact [highscore-name-entry-behavior.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/highscore-name-entry-behavior.json:1) still ended with a `helper_boundary` block that said:

- behavior around `0x1765a` was locally unresolved

That wording was stale after the later project-wide `0x1765a` closure:

- [1765a-reachability-closure-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/1765a-reachability-closure-pass-2026-04-20.md:1)
- [1765a-closure-propagation-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/1765a-closure-propagation-pass-2026-04-20.md:1)

So this pass updates the high-score branch to match the current project-wide boundary:

- keep `0x1765a` documented
- keep it non-required
- stop implying that `0x50b0` still depends on resolving it

## Findings

### 1. `0x50b0` Uses A Stable Shared Frontend Clear / Project / Draw / Flush Loop

The owned raw export at:

- [raw-50b0-5a93.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/highscore-edge-pass/raw-50b0-5a93.asm:1)

shows the same bounded frontend helper family across the visible loop phases.

Direct helper calls inside `0x50b0`:

- `0x17613` at:
  - `0x5239`
  - `0x5403`
  - `0x561a`
  - `0x5706`
- `0x175c5` at:
  - `0x5357`
  - `0x55d1`
  - `0x56bd`
  - `0x5828`
- `0x17719` at:
  - `0x5368`
  - `0x55e2`
  - `0x56ce`
  - `0x5839`
- `0x39c4` at:
  - `0x5320`
  - `0x5472`
  - `0x5673`
  - `0x57f4`

That gives the handler a repeatable owned loop skeleton:

1. clear previous chunk-7 object pixels through `0x17613`
2. redraw text or row-local content through `0x60cc` and related local helpers
3. advance frontend object state through `0x39c4`
4. redraw bounded chunk-7 object pixels through `0x175c5`
5. flush dirty cells through `0x17719`
6. present through `0x24d0`

That contract is now strong enough to describe directly without leaning on the broader helper-family ambiguity.

### 2. The Name-Entry Substate Owns Its Local Row Rewrite Helpers Directly

Inside the live name-entry slice, the raw handler shows a concrete row-local ownership family:

- `0x964`
  - released-key poll
- `0x94c`
  - release-latch clear after accepted actions
- `0x61c8`
  - clear only the active row band
- `0x5868`
  - active-row pulse / one-shot commit gate

Those direct calls appear in the expected name-entry region:

- `0x5424`
  - `0x964`
- `0x5439`
  - `0x94c`
- `0x5454`
  - `0x61c8`
- `0x5481`
  - `0x5868(row, 0)` continuous pulse
- `0x54c0`
  - `0x5868(row, 1)` one-shot commit gate
- `0x54ce`
  - second `0x964`
- `0x54e4`
  - second `0x94c`

That means `0x50b0` is not just "some screen near the helper family."
It has a direct owned local rewrite model:

- row-local clear
- row-local string rebuild
- local pulse / commit gate
- shared frontend object redraw
- shared flush / present

### 3. `0x1765a` Is Absent From The Owned `0x50b0` Call Surface

The same raw `0x50b0` export shows:

- direct calls to `0x17613`
- direct calls to `0x175c5`
- direct calls to `0x17719`

But no direct call to:

- `0x1765a`

That matters more now than it did on 2026-04-14, because the project-wide `0x1765a` status is no longer "active unresolved helper boundary."

It is now:

- documented non-required primitive
- behavior understood
- statically unreachable in the owned shipped binary

So the right high-score-side reading is now:

- the nearby raw helper export remains useful as supporting context for:
  - `0x17613`
  - `0x175c5`
  - `0x17719`
- but `0x50b0` itself does **not** need a still-open `0x1765a` ownership caveat

### 4. The High-Score Handler Is Now Closed As A Self-Sufficient Frontend Screen Family

With the stale helper-boundary wording removed, the live high-score artifact now cleanly says:

- `0x50b0` owns:
  - qualification insert logic
  - reveal
  - live name entry
  - row commit
  - footer exit
  - conceal
- its renderer-side presentation uses the same bounded chunk-7 clear / draw / flush family already owned elsewhere
- the remaining `0x1765a` closure belongs to the general renderer-family record, not to the local high-score handler model

That is a much better fit for both preservation and port planning.

## Closure

The current best reading is now:

- `0x50b0` is a fully owned high-score handler with a stable local interaction model and a stable shared frontend render loop
- `0x1765a` is absent from the owned direct `0x50b0` call surface
- the high-score branch should therefore stop carrying a local "helper boundary unresolved" note

So this branch is now specific enough to pause as closed behavior plus closed ownership.

## Artifact

This pass adds:

- [highscore-name-entry-ownership-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/highscore-name-entry-ownership-closure.json)

## What I Now Treat As Resolved

- the live high-score behavior artifact no longer needs to imply a remaining local `0x1765a` ambiguity
- `0x50b0` now has a clean owned helper family for preservation purposes
- the high-score raw index no longer needs to describe `0x1765a` as an active remaining boundary

## Next Ordered Step

- pivot to the next small artifact-consistency cleanup rather than reopen another large subsystem
- current best candidate:
  - stale superseded wording around `slot 2` in the audio-side closure artifacts
