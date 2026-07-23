# Raw Evidence Boundary Pass

Date: 2026-04-14

## Summary

This pass tightens four closely related confidence questions:

- how hard we should lean on the current `state 9` trigger model
- which dirty-map writers are directly confirmed to write value `3`
- which current paths really bypass the three-page incremental presentation model
- how strong the initial displayed-page ownership claim is around video-mode init

The main payoff is not a new gameplay behavior.
It is a cleaner boundary between:

- directly confirmed executable facts
- strong structural readings
- places where we should still keep the wording a little humble

That is exactly the kind of maintenance pass that keeps a faithful port honest.

## 1. `State 9` Still Stays At Medium Confidence On Trigger Semantics

The current `state 9` trigger story still holds up well enough to keep using:

- finished game-over reaches the high-score flow through the same gameplay-to-frontend handoff seam
- that seam is best currently read as the released-`Esc` branch specialized by `0x184db == -2`

But this pass is important because it did **not** find a better on-disk late-session export than the already used executable-side reasoning.

What I rechecked:

- the currently exported `0x023b` first-pass file on disk only covers the early startup/resource section
- there is no second owned export on disk that shows the later `0x04a3 .. 0x04d6` branch window directly
- the machine-readable maps and findings already keep this path at medium confidence

So the right accuracy move here is:

- keep the current `state 9` trigger reading
- keep it explicitly marked as medium-confidence / executable-strong / capture-light
- do **not** silently promote it to "fully raw-proven" until we recover a later branch artifact

That is not backtracking.
It is good evidence hygiene.

## 2. The Current Direct Dirty Writers All Support Value `3`

This pass rechecked the dirty-map writers against the owned assembly exports instead of the higher-level summaries.

### `0x2d30`

`0x2d30` is now directly grounded as a rectangle marker that loads:

- `MOV CL,0x3`

and then writes that same byte into the coarse dirty map at `0x1ad96`.

So this is not just "some writer usually marks dirty."
It is a direct on-disk `3` writer.

### `0x175c5`

The chunk-7/object draw helper writes:

- `MOV byte ptr [... + 0x1ad97],0x3`

after plotting a previously empty pixel into the working screen.

### `0x17613`

The chunk-7/object clear helper writes:

- `MOV byte ptr [... + 0x1ad97],0x3`

after restoring the corresponding working-screen pixel to `0`.

### What That Lets Us Say Safely

We can now say with stronger precision:

- the currently confirmed direct dirty writers all use mark value `3`
- that fits the three-page propagation model very well

We should **not** overstate it as:

- "every dirty writer in the executable uses only 3"

because this pass did not prove that globally.

The accurate wording is:

- all currently confirmed direct writers support the value-`3` propagation model

That is a strong statement and it is enough to guide the port.

## 3. The Splash Presenter Still Looks Like The Only Confirmed Single-Page Bypass Path

I re-surveyed every currently owned exported `CALL 0x2938` site.

Current call-site set on disk:

- startup gameplay-base upload in `0x00000018`
- splash presenter `0x00002998`
- frontend entry/exit in `0x00003830`
- new-game bootstrap in `0x000005e0`

The practical split is now very clean:

### Triple-Seed Paths

These all use `0x2938` three times against:

- `0x2c6ab`
- `0x2c6bb`
- `0x2c6bf`

Confirmed current triple-seed paths:

- startup gameplay-base upload
- frontend entry
- frontend exit
- early new-game bootstrap

### Single-Page Path

The only currently confirmed single-page `0x2938` path is:

- `0x2998`

and it explicitly uploads to:

- `0x2c6bf`

That keeps the earlier architectural reading on solid footing:

- ordinary scene transitions synchronize the whole page ring
- the fullscreen splash presenter is the owned confirmed bypass path that writes only the currently displayed page

Could another single-page path exist elsewhere in the executable?
Possibly.
But within the owned export set right now, `0x2998` is the only confirmed one.

## 4. Initial Displayed-Page Ownership Is Strong, But Not Fully Closed At The Video-Mode Layer

The startup page-root assignments are fully direct:

- `0x2c6bf = 0x0a0000`
- `0x2c6bb = 0x0a4b00`
- `0x2c6ab = 0x0a9600`

And the later ring behavior is fully direct:

- `0x17719` flushes into `0x2c6ab`
- `0x24d0` rotates the ring and programs the CRTC from the new `0x2c6bf`

So the later runtime ownership model is strong.

The only place where I want to keep the wording slightly softer is the exact first displayed-page ownership immediately after video-mode init.

Why:

- the owned `0x2438` export is short and only directly shows BIOS mode set plus palette clear
- it does **not** currently give us a later explicit page-root or CRTC assignment inside that same export
- so "initial displayed page is `0x0a0000`" is still best read as:
  - strongly consistent with the startup assignments
  - strongly consistent with VGA default expectations
  - not yet independently re-proven inside the visible `0x2438` export itself

That means the right wording is:

- **high-confidence runtime reading**

not:

- mathematically closed from every layer of startup/video-mode evidence

That distinction matters more for note quality than for implementation risk, but it is worth keeping straight.

## Porting Impact

For the future `C++23 + SDL3` port, this pass sharpens the constraints nicely:

- keep the `state 9` trigger model, but keep its implementation note marked as medium-confidence until we recover the late branch artifact
- treat dirty value `3` as the confirmed common propagation value for the currently owned direct writers
- treat single-page direct presentation as a special-case splash behavior, not a normal transition behavior
- preserve the later page-ring ownership model confidently
- keep one small confidence note around the exact first displayed page right after raw video-mode init

That is a good balance between fidelity and honesty.

## Bottom Line

This was a cleanup pass, but it was a valuable one.

- `state 9` stays supported, but carefully worded
- dirty mark value `3` is now more directly evidenced
- `0x2998` remains the only owned confirmed single-page bypass path
- initial page ownership is strong, with one small video-init caution still noted

That is the kind of disciplined edge-trimming that makes later implementation safer.

## 5 Next Strongest Moves

1. Recover a later raw disassembly slice for the session-loop handoff so we can either confirm or deliberately downgrade the current `state 9` trigger model with direct branch text.
2. Tighten whether any additional gameplay-side helpers write dirty values other than `3`, especially outside the currently exported rectangle and chunk-7 paths.
3. Resolve whether any other direct-to-`0x2c6bf` uploads exist outside the currently owned export set, or lock `0x2998` in as the only shipped single-page presenter we need to preserve.
4. Tighten the exact first-visible state-`9` presentation against captures if we can recover or create a direct transition sequence.
5. Keep expanding the preservation spec so these evidence boundaries become implementation notes instead of only research notes.
