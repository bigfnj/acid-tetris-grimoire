# Slot-2 Audio Supersede Wording Cleanup Pass

Date: 2026-04-21

## Summary

This pass cleans up the last stale wording still embedded inside the slot-`2` audio closure artifact.

Main result:

- the live audio model did **not** change
- the slot-`2` closure rule did **not** change
- the stale `supersedes` block in [slot2-sound-map-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/slot2-sound-map-closure.json:1) now correctly reflects the actual pre-closure drift:
  - findings-level wording was still `unresolved`
  - the living artifact-level status had already moved to `unused_small_ui_or_impact_alt`

So the slot-`2` branch is now handoff-safe without any remaining contradiction between its closure note and the live sound map.

## Why This Pass Was Needed

The `2026-04-21` root handoff correctly identified one last stale detail:

- [session-log-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/session-log-2026-04-21.md:1)

The closure pass itself was already directionally right:

- [slot2-sound-map-closure-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/slot2-sound-map-closure-pass-2026-04-20.md:1)

and the living sound map was already right:

- [sound-event-map.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/sound-event-map.json:1)

But the closure artifact still said:

- findings-level slot-`2` status: `unresolved`
- artifact-level slot-`2` status: `unresolved`

That second line was stale.
The artifact-level state had already been closed to:

- `unused_small_ui_or_impact_alt`

So this pass makes the machine-readable closure record match the actual historical drift it was supposed to document.

## Updated Artifact

- [slot2-sound-map-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/slot2-sound-map-closure.json)

The updated record now says:

- `findings_level_slot2_status_before_closure = unresolved`
- `artifact_level_slot2_status_before_closure = unused_small_ui_or_impact_alt`

and adds a short note explaining the correction.

## Findings

### 1. The Live Slot-2 Closure Rule Was Already Correct

The owned live audio map still says:

- slot `2` is startup-loaded
- slot `2` is unplayed in the owned shipped binary
- slot `2` should be preserved but left unbound in a faithful first-pass `SoundEvent` map

That rule remains unchanged by this pass.

### 2. The Stale Detail Was Only In The Closure Artifact’s Historical Summary

The mismatch was not behavioral.
It was historical bookkeeping inside the closure artifact itself.

The actual pre-closure split was:

- findings note still said `unresolved`
- live `sound-event-map.json` had already moved to:
  - `unused_small_ui_or_impact_alt`

So the stale line was:

- `artifact_level_slot2_status = unresolved`

not the forward-looking audio interpretation.

### 3. The Slot-2 Branch Is Now Internally Consistent

After this cleanup:

- the findings doc says slot `2` is closed as shipped-but-unused preserved content
- the living sound map says slot `2` is `unused_small_ui_or_impact_alt`
- the closure artifact now records the real before-pass drift instead of inventing a second unresolved side

That makes future handoffs less likely to reopen slot `2` by mistake.

## Practical Impact

This is a small pass, but it removes the last misleading breadcrumb in the slot-`2` branch.

Future port or RE work should now read the slot-`2` state as fully settled under the current evidence set:

- preserve the asset
- do not bind it to a required shipped event
- treat any remaining uncertainty as historical intent only

## Next Ordered Step

- pause the slot-`2` branch completely
- if decompilation keeps moving now, choose a different bounded subsystem rather than revisiting slot `2`
- the `2026-04-21` root session log should now no longer treat slot-`2` cleanup as pending work

## Bottom Line

The important cleanup is:

- the slot-`2` closure artifact now matches the actual pre-closure drift, so the audio branch no longer contains stale supersede wording
