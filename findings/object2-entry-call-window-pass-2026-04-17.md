# Object-2 Entry Call-Window Pass

Date: 2026-04-17

## Summary

This pass tightened the static object-1 to object-2 handoff around the frontend pointfield helper family.

Main result:

- the important early object-2 entries are no longer just isolated helper addresses
- they now sit inside a specific recurring object-1 clear/project/draw/flush frame family
- the decisive branch condition is the record-local `+0x18` draw-success latch, not a separate hidden state bit

That matters for runtime closure:

- reaching object `2` late at `0x178B1` or later is still too late to say whether the earlier object-1 call window already exercised `0x175C5` and `0x17613`
- the runtime search target should therefore be refined from "earlier object 2" to "object-1 phase immediately before the clear/draw subloops that call object 2"

## New Owned Artifact

- [object2-entry-call-window.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/object2-entry-call-window.json)

This artifact ties together:

- PMW1 object boundaries
- active chunk-7 record fields
- direct object-1 to object-2 relocation entries
- the exact clear / draw / flush call-window structure in the menu entry and exit transition family

## Key Artifacts Reused

- [raw-09c8-0ae0.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-present-order-pass/raw-09c8-0ae0.asm)
- [raw-175c5-17719.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-record-followup/raw-175c5-17719.asm)
- [ATET.EXE.pmw1.manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/disassembly/pmodew/extracted/ATET.EXE.pmw1.manifest.json)

## Findings

### 1. `0x3DAC` And `0x3DDC` Are The Core Object-1 To Object-2 Pointfield Window

The direct raw sequence around `0x3d94 .. 0x3de5` now reads cleanly:

- `0x3d94`
  object-1 clear pass start
- `0x3da0`
  test `[record+0x18] == 1`
- `0x3da6`
  load `y` from `[record+0x10]`
- `0x3da9`
  load `x` from `[record+0x0c]`
- `0x3dac`
  call `0x17613`
- `0x3dc0`
  object-1 draw pass start
- `0x3dcd`
  load `y` from `[record+0x10]`
- `0x3dd1`
  load `x` from `[record+0x0c]`
- `0x3dd5`
  load color from `[record+0x14]`
- `0x3ddc`
  call `0x175c5`
- `0x3de1`
  store returned `EAX` back into `[record+0x18]`

That closes the local contract:

- `+0x0c` is projected `x`
- `+0x10` is projected `y`
- `+0x14` is draw shade or color
- `+0x18` is the previous-frame draw-success latch

So `+0x18` is **not** a static record enable bit.
It is the stored success result from `0x175c5`, which is exactly why the clear pass branches on it while the draw pass does not.

### 2. `0x3E19` Is Part Of A Full Entry-Transition Frame Loop, Not A Standalone Edge

The earlier focus on `0x3e19` by itself was too narrow.

Inside `0x3df4`, the object-2 window is:

1. inline clear loop through `0x3e19 -> 0x17613`
2. eight-row text reveal through `0x60cc`
3. one or more catch-up projection steps through `0x39c4`
4. inline draw loop through `0x3eab -> 0x175c5`
5. dirty flush through `0x3ebc -> 0x17719`
6. present through `0x24d0`

The frame loop uses the same catch-up counter at `0x184cb` as the rest of the known presentation family.

That means the early object-2 helper entries are embedded inside a normal presented-frame transition cadence, not only inside startup-only or one-shot logic.

### 3. The Exit Transition Repeats The Same Object-1 To Object-2 Window

The matching exit helper `0x3ee4` repeats the same object-2 seam:

- `0x3f09 -> 0x17613`
- `0x3f9c -> 0x175c5`
- `0x3fad -> 0x17719`

The main difference is only the text-reveal parameter:

- entry uses an increasing clamp to `0x30`
- exit uses `max(0x2f - frame_index, 0)`

So the pointfield object-1 to object-2 seam is not unique to one transition direction.
It is a reusable frontend frame family.

### 4. The PMW1 Relocation Table Confirms The Seam Explicitly

Object boundaries are now clean:

- PMW1 object `1`: starts at `0x0`
- PMW1 object `2`: starts at `0x175C5`

The decoded object-1 relocation blocks include direct `near_call_jmp` entries from:

- `0x3dac` to `0x17613`
- `0x3ddc` to `0x175c5`
- `0x3e19` to `0x17613`
- `0x3eab` to `0x175c5`
- `0x3ebc` to `0x17719`
- `0x3f09` to `0x17613`
- `0x3f9c` to `0x175c5`
- `0x3fad` to `0x17719`

So this is not just a disassembly-reading convenience.
The PMW1 object graph itself confirms a real object-1 to object-2 call window around the frontend pointfield presenter family.

### 5. Why This Changes The Runtime Search Target

The current best runtime landings remain around:

- `0x178B1`
- `0x178BF`
- `0x178C1`
- `0x178CE`
- `0x178E9`

Those are all later than:

- `0x175C5`
- `0x17613`
- `0x17719`

After this pass, the interpretation is sharper:

- those later landings do not merely miss "early object 2" abstractly
- they miss a specific already-structured object-1 frame window that has probably already performed clear, draw, and flush work by then

So the better runtime objective is now:

- land before the object-1 clear or draw subloops in the frontend frame family
- or catch the object-1 decision point that leads into those subloops

That is more precise than only asking for an earlier flat object-2 offset.

## Practical Porting Impact

This pass reinforces one preservation rule:

- the frontend pointfield path should stay modeled as `clear old points -> project records -> draw new points -> flush`

The clear side is conditional on prior draw success.
The draw side always attempts all records and writes back a fresh success latch.

That is a more exact model than a generic "repaint points every frame" description.

## Next Strongest Move

Use this tighter seam to drive the next runtime-facing pass:

1. target the object-1 decision or phase boundary immediately before `0x3d94` / `0x3dc0` / `0x3df4`
2. correlate the current late object-2 landings against this frame family so we know which phase they represent
3. only after that spend more runtime budget on earlier closure attempts for `0x175C5` and `0x17613`

## Bottom Line

The important handoff fact is:

- the early object-2 helper family is now statically closed as part of a real object-1 frontend frame window

So the remaining runtime problem is no longer just "reach object `2` earlier."
It is "reach the object-1 phase that is about to call object `2`."
