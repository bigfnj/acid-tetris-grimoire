# Config Schema Closure Pass

Date: 2026-04-17

## Summary

This pass closes the practical `SETUP.DAT` schema question that was left open after the runtime config-state and audio-matrix work:

- which persisted fields are true startup/control-flow levers?
- which ones only feed backend init or later runtime behavior?

Main result:

- the file layout is now cleanly closed
- signature validity is the only `SETUP.DAT`-resident field currently proven to participate in the early startup gate
- device/rate/stereo/bit-depth are startup backend-init inputs, not setup-gate bytes
- music volume, SFX volume, and track index are real runtime/audio-state controls, but not currently proven startup-branch selectors
- key bindings and high-score records are persistence fields, not current startup-control targets

## Updated Owned Artifacts

Regenerated setup analysis with schema classification:

- [setup-dat.analysis.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/raw/setup-dat/setup-dat.analysis.json)

New machine-readable closure artifact:

- [setup-dat-schema-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/setup-dat-schema-closure.json)

Updated decoder:

- [decode_setup_dat.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/decode_setup_dat.py)

## File Layout Closure

The runtime file traces and the decoder now line up exactly on the shipped `178`-byte layout:

- `5` bytes
  signature
- `4` bytes
  sound device index
- `4` bytes
  mixing rate
- `4` bytes
  stereo flag
- `4` bytes
  bit-depth flag
- `5` bytes
  gameplay key bindings
- `4` bytes
  music volume
- `4` bytes
  SFX volume
- `4` bytes
  current music track index
- `140` bytes
  five fixed-size high-score records

Observed DOSBox-X file traces show exactly that read shape, and frontend-exit saves rewrite the same logical order with a leading truncate/open write.

So there is no longer a meaningful “unknown middle config region” inside the shipped file.

## Field Role Closure

### 1. Signature Validity Is The Startup Gate

The first `5` bytes are the embedded `AciD` signature.

Current closure:

- `0x36dc` validates these bytes on load
- missing or invalid setup falls back to seeded defaults through `0x358c`
- that fallback result is OR-ed with the explicit `setup` command-line flag
- the combined result is what forces the early `0x3830(5)` sound-setup frontend entry

So signature validity is the only `SETUP.DAT` field family currently proven to participate in the early startup gate.

### 2. Device / Rate / Stereo / Bit Depth Feed Backend Init, Not The Gate

The four dword audio-format fields at offsets:

- `0x05`
- `0x09`
- `0x0D`
- `0x11`

have a now-clean role:

- `0x668c` consumes them during concrete backend initialization
- startup retries with device index `4` `None` if the configured backend fails
- the sound-setup screen edits these fields in memory, but does not live-reinitialize audio while the user is still inside that menu

That means these bytes are startup-relevant, but not in the same way as the signature:

- they shape backend init and later runtime audio behavior
- they do not currently define the known early setup-screen gate

### 3. Music Volume / SFX Volume / Track Index Are Runtime Controls

The three dwords at:

- `0x1A`
- `0x1E`
- `0x22`

now sit in a clearer category.

They matter:

- `0x36dc` normalizes the current track index modulo `6`
- `0x6544` uses the selected track during cold startup and later menu-side music cycling
- runtime probing proved that stereo, SFX volume, music volume, and track index can bias the late scout lane

But this pass closes an important boundary:

- these are real runtime/audio-state levers
- they are not currently proven startup-branch selectors in the same sense as signature validity

### 4. Key Bindings Are Gameplay / Menu Input State

The `5` binding bytes at `0x15` are now fully bounded as:

- persisted input bindings
- direct indexes into the live `0x100`-byte key-state table
- relevant to gameplay and menu handling
- not current startup-gate evidence

That means they remain important for a faithful port, but they are the wrong target for startup-branch hunting.

### 5. High Scores Share The File, But Not The Startup Role

The final `140` bytes are the five fixed-size high-score records:

- `20` bytes name
- `4` bytes score
- `4` bytes total lines

They are saved through the same `0x3604` file write path, but this pass closes their practical role:

- durable persistence data
- not a current startup-control lever

## Why This Matters

This pass turns the earlier config-state experiments into a cleaner decision rule.

We no longer need to think of `SETUP.DAT` as one vague family of “maybe startup-relevant state.”
It now separates into:

- startup gate
  signature validity only
- startup backend-init inputs
  device / rate / stereo / bit depth
- runtime audio controls
  music volume / SFX volume / track index
- gameplay/menu persistence
  key bindings
- persistence-only score records
  high-score table

That is the closure we needed before spending more runtime passes in this area.

## Recommended Next Move

The strongest next move is now:

- **Step 5: Startup-Path Predicate Pass**

Specifically:

- tighten the executable-side startup logic around `0x36dc`, `0x358c`, the explicit `setup` argv flag, and the later `0x668c` / `0x6544` startup sequence
- identify whether any still-unbounded predicate family exists between “signature validity gate” and “normal startup backend init”
- avoid more broad config mutation unless it targets one of those specific remaining predicates

## Bottom Line

`SETUP.DAT` is now practically closed as a schema:

- signature validity is the startup gate
- audio-format fields are backend-init inputs
- music/SFX/track are runtime control fields
- key bindings and high scores are persistence state, not startup selectors

That means the next best move is no longer more blind config-state fishing.
It is startup-path predicate closure.
