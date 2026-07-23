# Decompilation Trajectory Audit - 2026-05-27

## Verdict

The port trajectory is still aligned with an accurate decompilation-driven rebuild. The current implementation is intentionally not claiming full parity; it is converting closed reverse-engineering facts into explicit runtime state machines, deterministic probes, and visible checkpoints. That is the right direction.

## On-Track Signals

- The port still uses recovered assets directly for title, gameplay, startup splashes, font, chunk-7 frontend objects, chunk-3 HUD/piece overlays, and top-out art.
- New gameplay behavior is represented as explicit state: falling, collapse, top-out, gravity, lock, snapshot, warning, sleepy, high-score, and audio events are not hidden in generic screen swaps.
- Deterministic scripted smokes now cover the riskiest transition surfaces: line clear, pause/resume, top-out to high-score, and high-score exit.
- The checklist continues to mark partial work as `◑` instead of prematurely certifying parity.
- Original `SETUP.DAT` remains immutable; only build-local setup state is mutated.

## Accuracy Risks

- Piece generation is currently a deterministic seeded stand-in, not certified original RNG.
- Line-clear debris uses level-varying placeholder motion and colors rather than recovered chunk-3 ramp sampling and exact helper profiles.
- Snapshot capture now exists, but exact transient exclusion and page/restore ordering are still approximate.
- Audio routing is observable and slot-correct for several events, but tracker playback and recovered SFX playback are not implemented.
- High-score conceal and top-out timings are structured, but not capture-matched frame for frame.
- Frontend chunk-7 target-bank selection is still deterministic cycling, not the recovered randomized target behavior.

## Guardrails For Next Passes

- Keep adding deterministic smokes before broadening behavior; every new edge path should have a scriptable verification route.
- Preserve separate primitive families: frontend chunk-7 pixels, frontend text/highlight, direct gameplay block/HUD blits, gameplay transients, and temporary overlays.
- Do not mark checklist rows `✅` until behavior is capture- or evidence-certified, not merely structurally implemented.
- Avoid collapsing top-out, high-score entry, and menu transitions into generic modal or scene-swap code.
- Prefer small timing constants with documented uncertainty over invented "exact" values.

## Recommended Next Direction

1. Certify or refine piece RNG and spawn/rotation timing.
2. Replace placeholder line-clear debris with recovered row-pixel/color-ramp sampling.
3. Add real SFX playback behind the existing audio-event router.
4. Tighten snapshot/transition frame ordering against `transition-preservation-spec.md`.
5. Continue promoting high-confidence checklist rows from `◻` to `◑`, and only promote to `✅` with owned evidence.
