# Secondary Frontend Exit Contract Closure Pass

Date: 2026-04-20

## Summary

This pass closes the secondary frontend family as a single dispatcher contract instead of a pile of separate handler notes.

Main result:

- states `5..10` now split cleanly into:
  - one gameplay-facing configuration prelude:
    - sound setup
  - one nested frontend-only menu pair:
    - options
    - keyboard setup
  - three one-way return-to-main-menu presentation screens:
    - high-scores display
    - high-score qualification / name entry
    - credits
- only sound setup can leave the secondary family toward a direct dispatcher exit
- options and keyboard setup never leave the frontend loop directly
- credits and high scores are terminal viewers that always collapse back to main menu state `1`

That means the future port can now model the whole secondary frontend branch as an explicit small state graph with known exit semantics.

## New Owned Artifact

- [secondary-frontend-exit-contract.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/secondary-frontend-exit-contract.json)

This artifact records:

- the member states of the secondary frontend family
- the shared secondary menu contract reused by options, keyboard setup, and sound setup
- the exact dispatcher return states each secondary handler can stage
- the contextual meaning of sound setup state `2`
- the terminal return rules for credits and high scores

## Key Artifacts Reused

- [frontend-dispatcher-transition-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-dispatcher-transition-pass-2026-04-14.md)
- [frontend-secondary-state-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-secondary-state-pass-2026-04-14.md)
- [secondary-frontend-scaffold-and-live-audio-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/secondary-frontend-scaffold-and-live-audio-pass-2026-04-14.md)
- [frontend-mutable-rows-and-key-capture-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-mutable-rows-and-key-capture-pass-2026-04-14.md)
- [sound-setup-row-action-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/sound-setup-row-action-pass-2026-04-14.md)
- [sound-setup-return-and-resume-fidelity-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/sound-setup-return-and-resume-fidelity-pass-2026-04-14.md)
- [26d000-dispatch-state-return-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-dispatch-state-return-pass-2026-04-20.md)
- [26d000-handler-return-state-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-handler-return-state-pass-2026-04-20.md)
- [return-to-game-first-frame-closure-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/return-to-game-first-frame-closure-pass-2026-04-20.md)

## Findings

### 1. The Secondary Frontend Family Is Now A Closed Dispatcher Subgraph

The owned dispatcher map already identified these secondary states:

- state `5`
  sound setup
- state `6`
  options
- state `7`
  keyboard setup
- state `8`
  high-scores display
- state `9`
  high-score qualification / name entry
- state `10`
  credits

After this synthesis pass, their outbound rules are now specific enough to treat the whole group as a closed subgraph:

- `5 -> 2 or 3`
- `6 -> 1 or 7`
- `7 -> 1 or 6`
- `8 -> 1`
- `9 -> 1`
- `10 -> 1`

Only one member of the family can break out toward gameplay-facing dispatcher behavior:

- state `5` sound setup through return state `2`

No other secondary handler emits state `2`, `3`, or `4`.

### 2. Options And Keyboard Setup Form A True Nested Frontend-Only Pair

The options and keyboard handlers now read as a self-contained nested menu pair inside the main frontend loop.

Options `0x4584`:

- stays in place for rows `0..1` volume edits
- stages state `7` on `Keyboard Setup`
- stages state `1` on `Back To Main Menu`
- stages state `1` on `Esc`

Keyboard setup `0x49bc`:

- stays in place for rows `0..4` key-capture mode
- stages state `6` on `Back to Options Menu`
- stages state `1` on `Esc`

That means:

- options is not a gateway to gameplay
- keyboard setup is not a modal overlay on top of options
- together they are a reversible `6 <-> 7` nested branch inside the frontend redispatch loop

### 3. Sound Setup Is The Only Secondary Menu With A Direct Exit Out Of The Frontend Loop

Sound setup `0x5a94` is structurally still a sibling of the other secondary menus:

- eight visible row scaffold
- entry transition `0x3df4`
- steady clear -> catch-up -> redraw -> flush -> present loop
- exit transition `0x3ee4`

But its exit contract is different:

- row `4` `Play Game` stages state `2`
- `Esc` also stages state `2`
- row `5` `Exit to Dos` stages state `3`

So sound setup is the only secondary menu that is allowed to terminate the secondary family into a direct dispatcher exit.

That makes it a configuration prelude rather than a pure nested settings branch.

### 4. Sound Setup State `2` Is Contextual, Not A Generic “Resume Game” Label

The secondary-family closure is only correct if sound setup state `2` keeps its caller-context split.

Owned evidence already shows:

- from early startup state `5`, return state `2` means:
  - leave sound setup
  - restore the pre-resource startup snapshot
  - continue the startup path into later backend init, splashes, gameplay-base load, music start, and main menu
- from live main-menu `Return to Game`, return state `2` means:
  - restore the saved gameplay snapshot
  - preserve live run state
  - present one fresh gameplay-owned frame

So the contract is:

- sound setup emits dispatcher state `2`
- the meaning of that state is determined by the caller context around the dispatcher, not by sound setup alone

This is the important reason the future port should name state `2` as a frontend exit code first, then apply caller-specific meaning second.

### 5. Credits And High Scores Are One-Way Presentation Screens

The remaining secondary states are now specific enough to group together as terminal presentation screens.

Credits state `10`:

- runs its internal page timer and page-advance phases
- only stages state `1`
- never branches to another secondary state

High-scores display state `8`:

- runs display and footer-exit flow
- only stages state `1`

High-score qualification state `9`:

- runs insertion, reveal, live name entry, footer exit, and conceal phases
- still only stages state `1`

So `8`, `9`, and `10` are not nested menus.
They are presentation screens with internal subphases but a single outward destination:

- main menu state `1`

### 6. The Shared Secondary Menu Contract Is Now Specific Enough To Preserve Directly

Options, keyboard setup, and sound setup now share a closed common contract:

- eight visible row scaffold
- localized row-band clear and rewrite through `0x61c8`
- steady clear -> catch-up -> redraw -> flush -> present cadence
- release-latch-driven row movement through `0x964`
- one-shot gated row changes and exits through `0x3fd8`
- full 48-frame exit through `0x3ee4`

They differ only in row-local side effects and permitted return states:

- options:
  live volume edits, `1` / `7`
- keyboard:
  key-capture mode, `1` / `6`
- sound setup:
  config editing, `2` / `3`

That is now a real implementation contract, not just a cluster of similar observations.

## Practical Porting Impact

The future SDL port can model the secondary frontend family as:

- redispatch-only nested menus:
  - options
  - keyboard setup
- one configuration-prelude screen with exit authority:
  - sound setup
- terminal presentation screens:
  - credits
  - high scores display
  - high-score qualification / name entry

That is specific enough to drive a faithful state machine without reopening the dispatcher branch every time a screen is implemented.

## Next Strongest Move

Pause the secondary frontend exit-contract branch here unless one of these becomes newly useful:

1. a port milestone wants this family translated into actual code-state enums and transitions
2. a later capture pass needs one secondary screen timed against this now-closed exit graph
3. a static pass finds a previously missed secondary handler that would expand the family

If decompilation keeps moving now, the stronger use of time is another unresolved subsystem rather than more refinement on this family.

## Bottom Line

The important closure is:

- the secondary frontend family is now a fully named small dispatcher graph with known shared menu contract and exact outward exits
