# PA_PVP v9.8.0 Release Notes

Release date: 2026-03-08

## Summary

v9.8.0 is a hardening release.
It does not change the philosophical model of PA_PVP and does not introduce external tooling requirements.
It closes structural execution gaps discovered in real ping-pong use:

- batch outputs that could silently omit items
- code blocks contaminated by narrative or draft material
- collisions between delta-only PREV runs and mandatory ARTIFACT/TARGET audit blocks
- invalid outputs that could still look reusable as machine state
- snapshot reuse of invalid outputs
- invention of missing diffs, source text, hashes, or target files

The goal of this release is simple:
make the no-look copy/paste loop more reliable without turning the kernel into a heavier formal parser.

## Scope

This release updates the canonical kernel from:

- `PA_PVP_full_v9.7.1_canonical.txt`

to:

- `PA_PVP_full_v9.8.0_canonical.txt`

Documentation references were updated accordingly.

## Major Changes

### 1) Global output precedence

Added a strict precedence rule for output conflicts:

1. `BATCH_COVERAGE`
2. `CODE_BLOCK_PURITY`
3. `PREV_DELTA_SURFACE`
4. `ITEM_MODE_OBLIGATIONS`
5. `RENDER_BLOCKS`

Effect:

- lower-priority obligations can no longer expand the output surface once a higher-priority rule has already restricted it
- PREV delta-only runs now have an explicit dominance rule over mode-specific emission pressure

### 2) Hard batch coverage for no-look safety

Added `BATCH_COVERAGE (HARD)` with explicit set definitions:

- `input_ids`
- `queue_ids`
- `item_ids`
- `split_ids`
- `expected_ids = input_ids U split_ids`

New guarantees:

- every input id must appear in `[QUEUE]`
- every input id must appear in emitted `[ITEM id=...]`
- no extra ids may appear unless created by an explicit split
- if an item is split, the original id must still remain present

Mode behavior:

- in `CopyPasteMode=NO_LOOK`, any batch coverage violation invalidates the whole output
- in non-`NO_LOOK` flows, legacy partial-progress behavior may survive only if explicitly preserved by older canonical paths

Why this matters:

v9.7.1 could still produce outputs that were locally plausible but globally unsafe to re-paste because one item was missing.

### 3) Code-block purity as a hard invariant

Added `CODE_BLOCK_PURITY (HARD)`.

Inside the machine code block, only canonical PA_PVP blocks and allowed fields may appear.
The following are now explicitly forbidden inside the machine block:

- narrative prose
- explanations
- apologies
- draft text
- diff bodies
- pasted artifact content
- human guidance

Effect:

- the code block is treated more strictly as machine state, not mixed-mode output
- contamination now invalidates the run instead of being tolerated informally

### 4) Strict PREV delta surface

Added `PREV_DELTA_SURFACE (HARD)`.

When `<<<PREV>>>` exists, an item may emit only:

- `[DERIVED]`
- `[DELTA]`
- `[VERDICT_UPDATE]` if changed
- `[STEPS]`
- `[NEXT]` if state allows it

No other item-level blocks may appear under PREV.

This closes the most serious v9.7.1 collision:

- PREV runs were delta-only by contract
- ARTIFACT/TARGET items also had mandatory audit blocks
- the runtime had to arbitrate between two incompatible rules

v9.8.0 resolves that conflict by letting PREV delta-only surface win.

### 5) Audit serialization under PREV

Under PREV, separate audit blocks are now forbidden:

- `[AUDIT_EXACT]`
- `[ARTIFACT_AUDIT]`

If audit duties still apply, they must be serialized inside `[DELTA][PATCH]` using closed keys:

- `audit_exact_defect=...`
- `audit_exact_patch=...`
- `audit_exact_step=...`
- `artifact_audit_equivalence_probe=...`

Important:

- the normal causal delta text is still allowed
- what is forbidden is extra audit structure, not the existing delta contract itself

### 6) Invalid output becomes explicit and non-operational

v9.8.0 introduces a fail-closed invalid-output path without changing the `STATE` header.

State remains:

- `STATE: DECISION.TEST.FINAL`

Invalidity is signaled in `[PANEL]` only, through closed fields:

- `OutputValidity: INVALID`
- `InvalidReason: BATCH_COVERAGE | CODE_BLOCK_PURITY | PREV_SURFACE | OTHER`
- `CoverageDiff: ...` only for `BATCH_COVERAGE`

Why this design was chosen:

- changing the `STATE` line would have collided with the existing snapshot-recognition rules
- keeping `STATE` stable preserves compatibility with current header expectations
- the invalid branch is still parseable, but it is no longer reusable as machine state

### 7) Invalid output structure exception

Added `INVALID OUTPUT STRUCTURE EXCEPTION (HARD)`.

If `[PANEL]` contains `OutputValidity: INVALID`:

- normal `[QUEUE]` requirements are waived
- normal per-item `[ITEM id=...]` requirements are waived
- the machine output collapses to:
  - `STATE: DECISION.TEST.FINAL`
  - `[USER_PANEL]`
  - `[PANEL]`

This closes the formal collision with the older output contract, which otherwise required `[QUEUE]` and `[ITEM]` in every run.

### 8) Minimal invalid USER_PANEL

Added `USER_PANEL INVALID EXCEPTION (HARD)`.

For invalid outputs, `[USER_PANEL]` remains mandatory but is reduced to a minimal diagnostic form derived only from `[PANEL]`.

Allowed minimal fields:

- `BatchID`
- `OutputValidity`
- `InvalidReason`

No free diagnostic prose is allowed there.

### 9) Invalid outputs are rejected as machine-state input

Added explicit snapshot rejection:

- outputs with `[PANEL] OutputValidity: INVALID` must not be treated as valid `PREV_SNAPSHOT`
- no PREV recovery may happen from them
- they are diagnostic only

This works alongside the structural rule that invalid outputs omit `[QUEUE]` and `[ITEM]`, so they no longer match the old snapshot pattern either.

### 10) Missing source material is now a hard anti-invention rule

Added `MISSING_SOURCE_INPUT (HARD)`.

If a step or claim depends on missing source material such as:

- artifact text
- diff body
- original item body
- commit hash
- file target
- referenced source block

the runtime must not invent it.

Allowed responses are now strictly limited to:

- bounded acquisition step
- `STANDBY` with `trigger=MISSING_SOURCE_INPUT`
- `DISCARD` only if already forced by a higher-priority rule

The trigger maps to:

- `dominant_gate=DEPENDENCY_BLOCK`

Important:

`MISSING_SOURCE_INPUT` is a per-item operational trigger only.
It is not a global invalid-output reason.

## Compatibility

This is intended as a compatible hardening release, not a semantic reboot.

Unchanged at the core:

- forced verdict model
- batch-first structure
- snapshot-as-input philosophy
- no-look copy/paste workflow
- Step 1 priority stack
- debt / probe / lifecycle model
- consumer-layer `REPORT` and `REPORT LITE`

Changed behavior:

- invalid outputs are now explicitly marked and must not be reused as machine state
- PREV runs are more tightly constrained
- missing-source situations can no longer be papered over with plausible narrative reconstruction

## Migration Notes

### Replace canonical filename references

Update references from:

- `PA_PVP_full_v9.7.1_canonical.txt`

to:

- `PA_PVP_full_v9.8.0_canonical.txt`

### Treat invalid outputs as dead ends for SSOT input

If a run emits:

- `[PANEL]`
- `OutputValidity: INVALID`

then:

- do not use it as `<<<PREV>>>`
- do not use it as `SNAPSHOT_AS_INPUT`
- use it only as diagnostics, then rerun from valid source state

### PREV + ARTIFACT/TARGET behavior changed

In v9.7.1, this area was ambiguous.
In v9.8.0:

- PREV delta-only surface wins
- audit obligations must be serialized inside `[DELTA][PATCH]`
- separate audit blocks are forbidden under PREV

## Files Updated For This Release

Core:

- `PA_PVP_full_v9.8.0_canonical.txt`

Docs:

- `README.md`
- `QUICKSTART.md`
- `docs/guide.md`
- `docs/overview.md`
- `docs/examples.md`
- `docs/index.html`

## Validation Notes

This release was validated locally for documentation references after the update.

Link check result:

- `0 broken refs`

Recommended regression probes after adoption:

1. `PREV + ARTIFACT`
2. missing item in batch under `NO_LOOK`
3. explicit split with original id retained
4. contaminated code block
5. missing diff/hash/source
6. missing source while a higher-priority Step 1 already exists
7. invalid snapshot pasted back as input

## Release Positioning

This release is versioned as `v9.8.0`, not `v10.0`, because:

- it hardens the existing kernel rather than replacing it
- it does not intentionally break the main philosophical model
- it does not redefine the core decision semantics
- it resolves operational collisions inside the existing contract

## Final Takeaway

v9.8.0 is the release that makes PA_PVP harder to misuse at runtime.

It does not make the protocol broader.
It makes the existing machine surface stricter, cleaner, and safer to reuse in no-look ping-pong.
