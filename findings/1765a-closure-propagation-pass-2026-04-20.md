# `0x1765A` Closure Propagation Pass

Date: 2026-04-20

## Summary

This pass propagated the earlier `0x1765a` reachability closure into the live renderer and transition artifacts that still carried stale "unresolved helper" wording.

Main result:

- `0x1765a` is no longer described as an active unresolved renderer family in the living preservation artifacts
- renderer, gameplay-present-order, gameplay-edge, and transition-spec materials now all agree on the same rule:
  - real helper body
  - no owned shipped reachability
  - documented non-required primitive only

That makes the preservation model internally consistent again.

## New Owned Artifact

- [1765a-closure-propagation.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/1765a-closure-propagation.json)

This artifact records:

- which living artifacts still had stale unresolved wording
- how each one was tightened
- the shared closure rule now applied across the renderer/present-order model

## Updated Living Artifacts

- [renderer-primitive-map.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/renderer-primitive-map.json)
- [gameplay-present-order.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-present-order.json)
- [gameplay-edge-paths.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/gameplay-edge-paths.json)
- [transition-preservation-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/transition-preservation-spec.md)

## Key Artifacts Reused

- [1765a-reachability-closure-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/1765a-reachability-closure-pass-2026-04-20.md)
- [renderer-primitive-separation-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/renderer-primitive-separation-pass-2026-04-14.md)
- [gameplay-present-order-and-resume-delta-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-present-order-and-resume-delta-pass-2026-04-14.md)
- [gameplay-edge-paths-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-edge-paths-pass-2026-04-14.md)

## Findings

### 1. The Renderer Primitive Map No Longer Needs An “Unresolved Domain” Bucket For `0x1765a`

Before this pass, [renderer-primitive-map.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/renderer-primitive-map.json) still labeled the `untracked_pixel_swap` family as:

- domain `unresolved`
- confidence `medium`
- open question based on missing callers

That wording was stale after the 2026-04-20 reachability closure.

The updated rule is now:

- domain `documented_non_required_primitive`
- status `statically_unreachable_in_owned_shipped_binary`
- closure rule:
  keep it provisional and preserved, but do not treat it as a required shipped renderer family

That is a much better fit for the current evidence than leaving it in an active unresolved bucket.

### 2. Gameplay Present-Order Artifacts No Longer Treat `0x1765a` As A Required First-Frame Unknown

Before this pass, both living gameplay-side artifacts still carried open questions of the form:

- whether `0x1765a` participates in first-frame presentation

That was now too strong an uncertainty label.

After the reachability closure, the right preservation rule is narrower:

- first-frame ordering should be modeled without `0x1765a`
- `0x1765a` remains documented separately as a provisional primitive only

So the updated artifacts now:

- remove the old first-frame open question
- add an explicit closure-propagation note explaining why `0x1765a` is not part of the required first-frame model

### 3. The Transition Spec Now Uses The Same Port Rule

The live preservation spec had also drifted behind the closure.

Its renderer-family section now says:

- one documented non-required untracked pixel swap primitive
- `0x1765a`
- keep as a provisional preserved primitive only until a real caller or runtime lane appears

That means the implementation guidance now matches the reachability verdict instead of implying that the renderer stack still has one actively unresolved required family.

### 4. This Was A Real Closure-Propagation Pass, Not Just Cosmetic Rewording

The useful change is not the wording by itself.
It is that the project’s living preservation model is now internally consistent again:

- findings doc says `0x1765a` is specific enough to pause
- renderer map says it is non-required
- gameplay present-order model no longer treats it as a required first-frame variable
- transition spec no longer implies a missing canonical family

That prevents future handoffs from accidentally reopening `0x1765a` as if it were still a central renderer gap.

## Practical Porting Impact

The future port should continue to preserve the canonical renderer families as:

- frontend object pixels
- gameplay tracked transient pixels
- gameplay block blits
- shared dirty-cell flush backend

And it should keep `0x1765a` only as:

- documented
- provisional
- non-required unless new evidence appears

That is a safer and more faithful implementation boundary than leaving it in the active unknown bucket.

## Next Strongest Move

Pause the `0x1765a` propagation branch here unless one of these becomes newly useful:

1. a new caller family or binary variant actually reaches `0x1765a`
2. a runtime steering lane becomes strong enough to make a breakpoint result meaningful
3. a port implementation task needs a provisional untracked single-pixel dirty-write fallback

If decompilation keeps moving now, a different unresolved subsystem is still a stronger use of time.

## Bottom Line

The important closure is:

- the `0x1765a` reachability verdict now propagates cleanly through the live renderer and transition artifacts, so the preservation model no longer contradicts itself
