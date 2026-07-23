# Alert Lifetime Under Line-Clear Load Pass - 2026-04-15

This pass tightened one specific question:

- what happens to alert lifetime while line-clear collapse and debris are actively running

## New Artifact

- [alert-lifetime-under-lineclear-load.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/alert-lifetime-under-lineclear-load.json)

## Main Result

During pending clears, alert lifetime keeps advancing, but stack-warning refresh is suppressed.

That means line-clear-driven alerts can legitimately age out during collapse frames, even while debris remains active.

## Evidence Tightening

### 1. Line-clear alerts are triggered on the clear-detect step

In the clear-detect branch of `0x09c8`, after rows are identified and collapse state is seeded through `0x1d04(1)`, the code dispatches alert IDs and SFX in the same step through:

- `0x2008`
- `0x6817`

The line-clear trigger path consistently passes `EDX = 0xa0` into `0x2008`, so these are finite-lifetime alert effects.

### 2. Pending-clear steps short-circuit `0x09c8`

At the top of `0x09c8`, if `0x184df > 0`, it branches to:

- call `0x1d04`
- return immediately

This bypasses:

- `0x21c4` stack-warning updates
- the later line-clear alert dispatch block

So while clears are pending:

- no new stack-warning refresh from `0x21c4`
- no repeated line-clear alert re-trigger from `0x09c8`

### 3. Alert lifetime still ticks during collapse

Outer gameplay frame order remains:

1. `0x2e18`
2. `0x09c8`
3. `0x206c`
4. `0x2f24`
5. `0x17719`
6. `0x24d0`

So even in collapse-only `0x09c8` steps, `0x206c` still runs afterward and continues:

- lifetime decrement
- reveal progression
- expiry restore/clear when lifetime reaches zero

This is the critical fidelity detail:

- line-clear collapse does not pause alert animation timing

## Practical Consequence

Under line-clear-heavy frames, we should preserve that alerts can:

- continue revealing
- continue counting down
- and potentially expire mid-collapse

before normal stack-warning logic resumes (after pending clears drop back to zero).

## Updated Files

- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [transition-preservation-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/transition-preservation-spec.md)

## Next 10 Strongest Moves

1. Tighten spawn-collision and first visible failed-spawn presentation one more step.
2. Search for indirect or computed dirty-map writers that could refine mark-`3` propagation.
3. Revisit high-score footer-exit conceal rhythm against captures.
4. Build the renderer contract note that consolidates page ring, dirty propagation, transient cleanup, and overlap rules.
5. Expand owned raw artifact coverage for the remaining gameplay edge helpers adjacent to collapse and top-out.
6. Correlate a line-clear-heavy gameplay capture against the queue-cap and alert-lifetime model.
7. Refresh the root session log after the next cluster so restart context stays sharp.
8. Keep `0x1765a` provisional unless new caller evidence appears.
9. Tighten credits-to-main-menu conceal cadence with capture-assisted correlation.
10. Continue translating these rules into direct C++23/SDL3 implementation constraints.
