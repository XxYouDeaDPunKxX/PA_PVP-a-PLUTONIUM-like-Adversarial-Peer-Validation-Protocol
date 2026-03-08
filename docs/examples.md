---
title: "Examples"
description: "PA_PVP examples pack used as an equivalence probe for protocol changes."
---

# PA_PVP Examples (Validation Pack)

These examples are designed to validate that PA_PVP remains executable, batch-safe, and stable across revisions.
They are also intended as "equivalence probes" when proposing changes to the protocol.

[Home](./)

## How to Use

1) Copy one example input into your runtime.
2) Verify the invariants listed under it (do not overfit to exact wording).
3) For protocol changes: run the same examples before/after and confirm invariants still hold.

## Example 1 [starter]: Trivial PLAN (DIY, compress)

Input:
```text
<<<I id=E1 title="Choose lunch" expire_days=7>>>
<<<PLAN>>>
- Option A: eat at place X (5 min walk)
- Option B: eat at place Y (15 min walk)
<<<END>>>
```

Invariants:
- Output is ONE code block and starts with `STATE: DECISION.TEST.FINAL`.
- Includes `[USER_PANEL]` at the top (human-readable, derived-only).
- Uses DIY or compresses output due to LOW impact.
- Includes `[PANEL]` and `[QUEUE]` with `ds:`, `term:`, `ct:`, `gate:`, and `exp:` shown.
- Output includes a derived-only `[HUMAN_TABLE]` markdown table after the code block.

## Example R1 [starter]: REPORT LITE (consumer-layer, derived-only)

This is NOT a kernel run. It is a human rendering pass that reads a snapshot.

Input:
```text
REPORT LITE
<paste ONE PA_PVP snapshot here (the code block; extra derived text after it is OK)>
```

Invariants:
- Output is NOT a PA_PVP snapshot (no `[PANEL]`/`[QUEUE]` emission); it is a human report only.
- Output contains no code blocks.
- Report is derived-only: it must not change any verdict/gate/trigger/steps; it must not invent facts.
- If a value is not present in the snapshot, it must say `missing in snapshot`.

## Example R2 [intermediate]: REPORT (FULL, consumer-layer, derived-only)

Input:
```text
REPORT
<paste ONE PA_PVP snapshot here (the code block; extra derived text after it is OK)>
```

Invariants:
- Includes all items, with per-item details derived from the snapshot.
- Any percentages are mechanical utilization (timebox/complexity/completion) and are printed only if derivable; otherwise `missing in snapshot`.
- Output contains no code blocks.

## Example 2 [stress case]: Complex PLAN triggers PROBING (simulation-first)

Input:
```text
<<<I id=E2 title="Ship refactor with unknown risk" expire_days=60>>>
<<<PLAN>>>
- Refactor module A into service B
- Remove feature flag C
- Deploy by Friday
<<<END>>>
```

Invariants:
- If domain is Complex and critical driver confidence is LOW/MED, protocol proposes a Reality Probe.
- Probe includes kill-switch, representativeness, and decision_context LITE fields.
- If `context_match=YES`, decision_context should include a canonical `constraint_signature` (when available) to prevent context swapping across iterations.
- If the item enters `PROBING`, the probe must be adversarial-by-construction and include an explicit `kill_switch` and canonical `constraint_signature`.
- If `[PROBE_LOG]` includes `kill_switch_margin=PASS_NARROW` repeatedly under weak evidence, the item should trigger `REDESIGN_TRIGGER` (near-miss loop).
- In Deterministic domain, any `PASS_NARROW` should cap confidence (avoid `HIGH`) and force a mitigation step (second check or threshold redesign).
- If score is fragile near threshold, protocol forces PROBING (not CLOSED).
- If the first N probes are run (N=3 default), output includes a [PROBE_LOG] with required fields.
- `[STEPS]` obeys v9.2 grammar: each step has `-> <OUTPUT>`, `timebox<=...m`, and `fail_if=...` (no meta verbs unless rewritten as a measurable probe).

## Example 3 [starter]: ARTIFACT audit (no "need a plan")

Input:
```text
<<<I id=E3 title="Audit a prompt" expire_days=30>>>
<<<ARTIFACT>>>
<paste any prompt/protocol artifact here>
<<<END>>>
```

Invariants:
- Does not fail-closed asking for a plan.
- Includes `[ARTIFACT_AUDIT]` with the 3 checks: completeness/coherence/operationality.
- Includes `[AUDIT_EXACT]` with exactly: defect / patch / step.

## Example 4 [intermediate]: TOOL + TARGET (must not invert)

Input:
```text
<<<I id=E4 title="Tool audits target">>>
<<<TOOL>>>
<tool artifact>
<<<TARGET>>>
<target artifact>
<<<END>>>
```

Invariants:
- Mode is TARGET analysis and roles are not inverted.
- Output is in PA_PVP format (not a generic essay).

## Example 5 [intermediate]: Dependencies in batch

Input:
```text
<<<B expire_days=30>>>

<<<I id=E5A title="Decide architecture" expire_days=60>>>
<<<PLAN>>>
- Choose between Option 1 and Option 2
<<<END>>>

<<<I id=E5B title="Implement" depends_on=E5A>>>
<<<PLAN>>>
- Implement the chosen option
<<<END>>>
```

Invariants:
- `[QUEUE]` shows `exp:` for both.
- For E5B, Step 1 resolves dependency or explicitly waives it.

## Example 5c [stress case]: Circular dependencies (DAG-only)

Input:
```text
<<<B expire_days=30>>>

<<<I id=E5X title="Cycle A" depends_on=E5Y>>>
<<<PLAN>>>
- Proceed
<<<END>>>

<<<I id=E5Y title="Cycle B" depends_on=E5X>>>
<<<PLAN>>>
- Proceed
<<<END>>>
```

Invariants:
- The batch detects the cycle by fixed rule (no waiting).
- Both items are parked (`new_state=STANDBY`, verdict `DO LATER`).
- Both items use `trigger=CIRCULAR_DEPENDENCY` and `gate=REDESIGN_TRIGGER`.
- The earliest non-conflicting step for each item must break the cycle (split/refactor ids, remove/replace an edge, waive one edge with debt, or discard a dominated item).
- Because the items are `STANDBY`, `[NEXT]` may be omitted; if present, it is a "next planned action on revive" (must not be interpreted as execute-now).

## Example 5d [stress case]: PROBING blocked by unresolved dependency (probe must be dependency-independent)

Input:
```text
<<<B expire_days=30>>>

<<<I id=E5D_A title="Upstream output needed">>>
<<<PLAN>>>
- Produce output X (not yet closed)
<<<END>>>

<<<I id=E5D title="Probe would require upstream output" depends_on=E5D_A>>>
<<<PLAN>>>
- Run probe that requires upstream output X -> probe_pass=YES/NO
<<<END>>>
```

Invariants:
- If `E5D_A` is not `CLOSED/DISCARDED`, `E5D` must not enter `PROBING` if the only viable probe requires executing blocked dependent steps.
- `E5D` is parked with `trigger=DEPENDENCY_BLOCK` (gate `DEPENDENCY_BLOCK`) and Step 1 follows dependency resolution/waiver.

## Example 5b [intermediate]: Resource concurrency (no new tags)

Input:
```text
<<<B resource_pool=ENG:1>>>

<<<I id=E5C title="Ship item A" uses=ENG:1>>>
<<<PLAN>>>
- Do A
<<<END>>>

<<<I id=E5D title="Ship item B" uses=ENG:1>>>
<<<PLAN>>>
- Do B
<<<END>>>
```

Invariants:
- If both items would be `DO NOW`, the kernel must demote one to `DO LATER` due to capacity.
- The demoted item uses `trigger=DEPENDENCY_BLOCK` (no new tags).

## Example 6 [intermediate]: Delta-only ping-pong + contestation

Input (second turn):
```text
<<<I id=E6 title="Ping-pong" expire_days=30>>>
<<<PLAN>>>
- Do X then Y
<<<PREV>>>
<paste ONLY the previous [ITEM id=E6] block (not PANEL/QUEUE, not other items)>
<<<END>>>
```

Invariants:
- Output for E6 is DELTA-only (one BREAK + one PATCH).
- DELTA output includes a short `[DERIVED]` block (derived-only fields).
- `[DERIVED]` includes `derived_version`.
- `[DERIVED]` includes `events` (max 3) as causal anchors.
- If `contested=YES` in PREV, BREAK addresses the contested point explicitly.
- DELTA includes at least one explicit causal reference (driver/relationship/probe_id/evidence).

## Example 7 [intermediate]: Revisit CLOSED requires NEW_EVIDENCE

Input:
```text
<<<I id=E7 title="Reopen closed decision">>>
<<<PLAN>>>
- Do X
<<<PREV>>>
<paste ONLY the previous [ITEM id=E7] block where state was CLOSED>
<<<NEW_EVIDENCE evidence_tier=REAL>>>
<paste a new outcome/measurement/log>
<<<END>>>
```

Invariants:
- If PREV was `CLOSED` with `ct:REAL`: reopening without `<<<NEW_EVIDENCE>>>` is blocked.
- If PREV was `CLOSED` with `ct:SIM`: reopening is allowed with either `<<<NEW_EVIDENCE>>>` or `<<<CHANGE>>>`.
- If PREV indicates `reopen_total>=1` for the same `id`, reopening is blocked even if `<<<NEW_EVIDENCE>>>` is present (`trigger=REOPEN_BLOCKED_BUDGET`).
- With `<<<NEW_EVIDENCE>>>`, evidence tier is reflected in binding and `ct:REAL` is allowed.
- If PREV was `CLOSED` with `ct:REAL` and the item is reopened, Step 1 must be a mitigation/compensation step.

## Example 7c [stress case]: Step 1 priority (reopen mitigation > dependency)

Input (batch, second turn):
```text
<<<B expire_days=30>>>

<<<I id=E7C_A title="Upstream dependency">>>
<<<PLAN>>>
- Decide X
<<<END>>>

<<<I id=E7C title="Reopen REAL + blocked dependency" depends_on=E7C_A>>>
<<<PLAN>>>
- Continue execution
<<<PREV>>>
<paste ONLY the previous [ITEM id=E7C] block where state was CLOSED with ct:REAL>
<<<NEW_EVIDENCE evidence_tier=REAL>>>
<paste a new outcome/measurement/log>
<<<END>>>
```

Invariants:
- Step 1 follows the fixed priority order: mitigation/compensation comes before dependency resolution.
- The dependency requirement is not dropped: an earliest non-conflicting later step must resolve/waive `E7C_A` before any dependent execution.

## Example 7b [intermediate]: Irreversible items cannot SIM-close

Input:
```text
<<<B close_policy=SIM_OK exec_capability=NO_RUNTIME>>>

<<<I id=E7B title="Irreversible decision" reversibility=IRREVERSIBLE>>>
<<<PLAN>>>
- Do irreversible action
<<<END>>>
```

Invariants:
- Item must not end `CLOSED` with `ct:SIM`.
- If it would close without real evidence, it should park with `CLOSE_BLOCKED_REAL_ONLY` (or equivalent protocol-consistent block).

## Example 8 [stress case]: Operational pressure (CUT MODE + split on multi critical driver)

Input (batch):
```text
<<<B expire_days=30>>>

<<<I id=E8A title="CUT MODE after 2 FAIL-FAST">>>
<<<PLAN>>>
- Make it better
<<<PREV>>>
[ITEM id=E8A]
[ITEM_PANEL]
id: E8A
title: CUT MODE after 2 FAIL-FAST
mode: PLAN
profile: DIY
state: prev_state=None, new_state=DRAFT
verdict: DO LATER
impact: LOW
urgency: LOW
score: 30
scoring_version: DIY=anchor_based
contested: NO
depends_on:
missing_ratings: NO
fail_fast_count: 1
why: FAIL-FAST triggered previously (missing minimal specs).
critical_driver: missing minimal specs
dominant_tradeoff: speed vs clarity
falsification_debt: NONE
next_uncertainty: goal state / boundary / constraint missing
trigger: FAIL_FAST_PLAN_SNR
[STEPS]
S1: Provide minimal specs -> specs_v1 | timebox<=5m | fail_if=specs_missing_any_required_line
[NEXT]
S1: Provide minimal specs -> specs_v1 | timebox<=5m | fail_if=specs_missing_any_required_line
<<<END>>>

<<<I id=E8B title="Split independent high-impact drivers">>>
<<<PLAN>>>
- Decide shuttle purchase for staff commute (capex + maintenance risk)
- Decide meal vendor for daily staff meals (quality + health risk)
<<<END>>>
```

Invariants:
- E8A triggers FAIL-FAST again and is CUT: `verdict=DISCARD`, `trigger=FAIL_FAST_CUT`, `new_state=DISCARDED` (no further questions).
- E8B MUST be split into 2 new items (new ids) in the same output:
  - original E8B shows `trigger=MULTI_CRITICAL_DRIVER_SPLIT` and `split_into=[...]` and is DISCARDED
  - each new item includes `split_from=E8B` and has exactly one `critical_driver`
- `[QUEUE]` includes the newly created items (first-class batch items).
- The newly produced items include steps that obey the executable grammar (OUTPUT + timebox + fail_if).

## Example 9 [starter]: Snapshot-as-input (no-look paste full output)

Input:
```text
<paste the entire previous assistant output code block>
```

Invariants:
- Protocol detects PREV_SNAPSHOT and accepts it as valid input (SNAPSHOT_AS_INPUT).
- Protocol auto-recovers per-item `<<<PREV>>>` from `[ITEM id=...]` blocks and proceeds delta-only.
- Protocol MUST NOT ask the user to cut/rewrap anything to resume.
- If a [NEXT] step is self-executable in the current environment, the protocol may advance it and record `status=COMPLETED | result=...`.
- Default in `CopyPasteMode: NO_LOOK`: `AskUser` is `NONE` (simulation-first).
- To enable external-data questions: set `ask_user=ALLOW` in `<<<B ...>>>` (or per-item override).
- If the pasted prior output includes extra text after the code block (e.g., `[HUMAN_TABLE]`, `[HUMAN_TABLE_OPS]`, footer lines), SNAPSHOT-AS-INPUT MUST ignore it.

## Example 10 [intermediate]: Derived confidence caps (overclaim blocked)

Input (second turn, with PREV):
```text
<<<I id=E10 title="Derived confidence caps block overclaim" expire_days=60>>>
<<<PLAN>>>
- Continue
<<<PREV>>>
[ITEM id=E10]
[ITEM_PANEL]
id: E10
title: Derived confidence caps block overclaim
mode: PLAN
profile: FULL
domain_type: Complex
state: prev_state=None, new_state=PROBING
verdict: DO NOW
impact: HIGH
urgency: HIGH
score: 82.9
scoring_version: FULL=v9.1_normalized
contested: NO
depends_on:
missing_ratings: NO
critical_driver_confidence: HIGH
hedge_action: NO
fail_fast_count: 0
why: Prior peer overclaimed HIGH confidence on weak evidence.
critical_driver: confidence_manipulability
dominant_tradeoff: expressivity vs objectivity
falsification_debt: NONE
next_uncertainty: whether caps apply by fixed rule
trigger: PROBE_ITERATION
[DERIVED]
derived_version: v9.8.0
item_exec_mode: SLOW
terminal: NO
closure_tier: -
best_evidence_tier: DERIVED
events: [TRIGGER_PROBE_ITERATION]
counters: tb=20/120 cx=2/7 pc=0 nm=0 nmd=0 inc=0 ovr=0 rb=0
effective_expire_days: 60
output_mode: FULL
[STEPS]
S1: Run equivalence probe -> derived_conf_pass=YES/NO | timebox<=20m | fail_if=breaks_contract
[NEXT]
S1: Run equivalence probe -> derived_conf_pass=YES/NO | timebox<=20m | fail_if=breaks_contract
<<<END>>>
```

Invariants:
- Because `domain_type=Complex` and `best_evidence_tier=DERIVED`, confidence MUST be capped at `MED` (no `HIGH` overclaim).
- `critical_driver_confidence` MUST be <= `MED` after cap application.
- Output includes a causal note / trigger indicating the cap was applied (e.g., `CONFIDENCE_CAP_APPLIED`).

## Example 11 [stress case]: Dependency waiver + debt (explicit)

Input (batch):
```text
<<<B expire_days=30>>>

<<<I id=E11A title="Decide architecture" expire_days=60>>>
<<<PLAN>>>
- Choose between Option 1 and Option 2
<<<END>>>

<<<I id=E11B title="Implement with dependency waiver" depends_on=E11A expire_days=60>>>
<<<PLAN>>>
- Waive dependency E11A and proceed anyway (time critical)
<<<END>>>
```

Invariants:
- For `E11B`, Step 1 explicitly waives dependency OR resolves it.
- If waived, protocol MUST treat as blind-risk acceptance and open debt:
  - `falsification_debt=OPEN(critical_driver)` and `critical_driver_confidence` capped to `LOW` for that run.
- `E11B` MUST include a probe/acquisition step to reduce uncertainty caused by the waiver (unless impact is LOW).
- If no higher-priority Step 1 applies, the waiver action should be Step 1 and the waiver-required probe/acquisition should appear at the earliest non-conflicting later step.
- With `falsification_debt=OPEN(...)` under weak evidence (`SIMULATED/DERIVED`), complexity tax should be harsher (debt contributes `+1` to `complexity_score`), making redesign/splitting more likely if the plan is already near budget.

## Example 12 [stress case]: Context-match stall exit (2x UNKNOWN/NO => redesign A/B/C)

Input (second turn, with PREV):
```text
<<<I id=E12 title="Context-match stall exit" expire_days=60>>>
<<<PLAN>>>
- Continue probing
<<<PREV>>>
[ITEM id=E12]
[ITEM_PANEL]
id: E12
title: Context-match stall exit
mode: PLAN
profile: FULL
domain_type: Complex
state: prev_state=None, new_state=PROBING
verdict: DO NOW
impact: MED
urgency: MED
score: 60
scoring_version: FULL=v9.1_normalized
contested: NO
depends_on:
missing_ratings: NO
critical_driver_confidence: MED
hedge_action: NO
fail_fast_count: 0
why: Context match cannot be confirmed.
critical_driver: driver_x
dominant_tradeoff: speed vs validity
falsification_debt: NONE
next_uncertainty: context_match observability
trigger: PROBE_ITERATION
[PROBE_LOG]
- probe_id: E12_PRB_01
  target_driver: driver_x
  context_match: UNKNOWN
  representativeness: MED
  evidence_tier: SIMULATED
  status: INCONCLUSIVE
- probe_id: E12_PRB_02
  target_driver: driver_x
  context_match: UNKNOWN
  representativeness: MED
  evidence_tier: SIMULATED
  status: INCONCLUSIVE
[STEPS]
S1: Run probe in declared context -> context_match=YES/NO | timebox<=20m | fail_if=context_match!=YES
[NEXT]
S1: Run probe in declared context -> context_match=YES/NO | timebox<=20m | fail_if=context_match!=YES
<<<END>>>
```

Invariants:
- Two consecutive `INCONCLUSIVE` with `context_match in {UNKNOWN,NO}` triggers `PROBE_STALL_CONTEXT_MATCH`.
- Output MUST choose exactly one redesign path A/B/C (probe redesign, split, or STANDBY/DISCARD) with no questions.

## Example 13 [intermediate]: Peer convergence (A/B, consumer-level)

Input:
```text
<<<I id=E13 title="Peer convergence compare" expire_days=60>>>
<<<PLAN>>>
- Run the same input twice (two independent reviewers/hosts)
- Compare the two outputs by fixed rules (consumer-level)
<<<END>>>
```

Invariants:
- Both outputs remain valid PA_PVP snapshots (ONE code block, batch-safe).
- Steps must keep stable output tokens (`S#: ... -> <TOKEN> | ...`) so a consumer can compare the two runs without NLP.
- A consumer can classify the item as `CONVERGED` / `SOFT_DIVERGENCE` / `HARD_DIVERGENCE` using:
  - verdict agreement
  - gate agreement (`gate:` token)
  - overlap on the first 3 step tokens + `[NEXT]` token (see [`docs/guide.md`](guide.md)).

## Example 14 [intermediate]: RISK_DASHBOARD (derived-only, batch hotspots)

Input (batch):
```text
<<<B expire_days=30 ssot_scale=MIN>>>

<<<I id=E14A title="Driver X risky (1)" expire_days=60>>>
<<<PLAN>>>
- Do thing A
<<<END>>>

<<<I id=E14B title="Driver X risky (2)" expire_days=60>>>
<<<PLAN>>>
- Do thing B
<<<END>>>
```

Invariants:
- If emitted, `[RISK_DASHBOARD]` MUST be outside the code block and MUST be derived-only (rendering).
- SNAPSHOT-AS-INPUT MUST ignore `[RISK_DASHBOARD]` (it must not affect parsing).
- The dashboard MUST be derived only from `[QUEUE]` + `[ITEM_PANEL]` short fields (no new semantics).

## Example 15 [intermediate]: Runtime exceptions (RUNTIME_OK, closed vocabulary)

Input (batch):
```text
<<<B exec_capability=RUNTIME_OK close_policy=SIM_OK>>>

<<<I id=E15 title="Runtime exception encoding" expire_days=30>>>
<<<PLAN>>>
- Run a step that may hit a runtime exception (I/O, auth, timeout, invariant, dependency)
<<<END>>>
```

Invariants:
- In `exec_capability=RUNTIME_OK`, if a step fails due to a runtime exception (not `fail_if`), the step output token MUST be one of:
  - `EXCEPTION_IO`, `EXCEPTION_TIMEOUT`, `EXCEPTION_AUTH`, `EXCEPTION_INVARIANT`, `EXCEPTION_DEPENDENCY`
- Such a step MUST be marked `status=FAILED_EXCEPTION` and mapped to existing gates (no new tags).
