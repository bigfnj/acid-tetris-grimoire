# Session Loop Handoff Confirmation Pass

Date: 2026-04-14

## Summary

This pass closes three high-value evidence gaps together:

- the late session-loop branch into frontend state `9`
- the direct whole-binary dirty-map writer set
- the direct whole-binary reference set for `0x2c6bf`

The biggest result is simple and important:

- the `state 9` handoff is no longer just a best current reading
- it is now directly confirmed from the flat-relocated binary

This same whole-binary sweep also strengthened two renderer claims:

- tracked-particle plot/restore helpers are additional direct dirty-`3` writers
- `0x2998` is still the only confirmed single-page direct-display presenter, now against the whole current flat-relocated binary sweep rather than only the exported decompilation subset

That is a strong fidelity win.

## 1. The `State 9` Trigger Is Now Directly Confirmed

The critical raw sequence in the flat-relocated binary is:

- `0x0488`
  `MOV EBP,0x1`
- `0x048f`
  `CALL 0x17875`
- `0x04a3`
  `LEA EDX,[ESP + 0x10]`
- `0x04a7`
  `MOV EAX,[0x2c227]`
- `0x04ae`
  `ADD EAX,0x1f`
- `0x04b1`
  `CALL 0x0e6c8`
- `0x04b8`
  `JNE 0x04ef`
- `0x04ba`
  `CALL 0x094c`
- `0x04bf`
  `CMP [0x184db],0xfffffffe`
- `0x04c8`
  `CALL 0x23ac`
- `0x04cd`
  `MOV EAX,0x9`
- `0x04d6`
  `CALL 0x3830`

That means the finished game-over handoff path is now directly executable-proven:

- compare the release-latch tail
- if it matches, clear the latch
- if `0x184db == -2`, restore the `GAME OVER` underlay
- call `0x3830(9)`

There is no longer a reason to keep the basic control-flow seam at medium confidence.

## 2. Why That Compare Is Specifically A Released-`Esc` Check

This pass also closes the one remaining semantic gap in that compare.

At the very start of the main runtime entry:

- address `0x00000010` contains byte `0x01`
- address `0x00000011` begins the embedded string `"setup"`

The startup code at `0x0036` copies that literal byte `0x01` into:

- `[ESP + 0x10]`

Then a direct raw sweep of the combined `0x0018 .. 0x04d6` region shows:

- no later writes to `[ESP + 0x10]`
- the next use of that stack location is the compare at `0x04a3`

At the compare site:

- `EBP` is already `1`
- `0x0e6c8` is a byte-range compare helper
- `EAX` points to `[0x2c227] + 0x1f`
- `EDX` points to `[ESP + 0x10]`
- `EBX` receives `EBP`, so compare length is exactly `1`

And we already know:

- `0x0964` reads the release-latch tail from `[0x2c227] + 0x1f`
- `0x094c` clears that same `0x20`-byte release latch

So the best wording now is stronger:

- the session-loop branch directly compares the most recent released scan-style code against literal byte `0x01`
- that is the released-`Esc` seam

That is no longer just a plausible interpretation.
It is the direct executable story.

## 3. The Whole-Binary Dirty-Map Sweep Strengthens The Mark-`3` Model

I re-ran the dirty-map survey against the whole flat-relocated binary, not only the exported helper files.

Current direct references to the dirty maps now include:

- `0x2d30`
  rectangle dirty marker writes `0x03` into `0x1ad96`
- `0x2d88`
  clears `0x1ad97`
- `0x175c5`
  chunk-7/object plot path writes `0x03`
- `0x17613`
  chunk-7/object clear path writes `0x03`
- `0x17821`
  tracked-particle plot path writes `0x03`
- `0x17875`
  tracked-particle restore path writes `0x03`
- `0x17719`
  flush backend decrements non-zero dirty bytes

There is also one additional direct mark-`3` store at:

- `0x1767c`

inside the same nearby object/particle helper cluster.

So the direct writer picture is now stronger than before:

- multiple unrelated localized-update systems write `3`
- one clear helper zeroes the map
- the flush backend decrements the value

That makes the three-page propagation model even healthier.

I still would not overstate this as:

- "every dirty writer in the whole executable uses only 3"

But the direct evidence set is now much broader.

## 4. The Whole-Binary `0x2c6bf` Sweep Leaves `0x2998` As The Only Confirmed Single-Page Presenter

The direct whole-binary references to `0x2c6bf` now resolve cleanly to:

- `0x007b`
  startup root assignment
- `0x03c0`
  startup gameplay-base triple seed
- `0x064f`
  new-game triple seed
- `0x2500 / 0x250c / 0x2512`
  page-ring rotation in `0x24d0`
- `0x2a03`
  fullscreen splash upload in `0x2998`
- `0x388b`
  frontend-entry triple seed
- `0x398a`
  frontend-exit / gameplay-restore triple seed

That is a very nice result.

It means the current stronger wording is justified:

- `0x2998` is the only confirmed single-page direct-display upload path in the whole current flat-relocated binary sweep

Everything else that meaningfully seeds a new scene either:

- assigns the page root
- rotates the ring
- or seeds all three pages

That keeps the renderer model crisp.

## Porting Impact

For the future `C++23 + SDL3` port, this pass sharpens three important rules:

- treat finished game-over to state `9` as a directly confirmed released-`Esc` handoff seam
- preserve dirty value `3` as the dominant localized-update propagation value across multiple systems, not only one helper family
- preserve single-page direct presentation as a splash-only special case, not a general scene-transition technique

These are exactly the kinds of details that separate a faithful port from one that only looks approximately right.

## Bottom Line

This pass materially improved the confidence profile of the project.

- `state 9` trigger semantics are now directly confirmed
- the direct dirty-writer set is broader and stronger
- `0x2998` stays the lone confirmed single-page presenter, now against the whole flat-relocated binary sweep

That is a good step forward for both reverse engineering and future implementation safety.

## 5 Next Strongest Moves

1. Recover or export a dedicated owned artifact for the full `0x023b` session-loop region so future passes do not rely on ad hoc `objdump` slices for this seam.
2. Tighten the unlabeled helper cluster around `0x17660 .. 0x17684` so we can name its exact role in the transient/object presentation path.
3. Keep searching for any indirect or computed dirty-map writes that would materially weaken the current mark-`3` propagation model.
4. Tighten the first visible `state 9` handoff against captures if we can produce or recover a direct transition sequence.
5. Continue converting these now-direct control-flow findings into implementation constraints in the preservation spec and machine-readable transition artifacts.
