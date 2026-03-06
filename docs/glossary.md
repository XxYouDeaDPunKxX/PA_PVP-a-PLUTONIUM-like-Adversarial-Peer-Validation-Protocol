---
title: "Glossary"
description: "Closed vocabulary to reduce semantic drift across AI-to-AI iterations."
---

# PA_PVP Glossary (Operational, Closed Vocabulary)

This glossary exists to reduce semantic drift across AI<->AI iterations.
Prefer these exact terms and avoid synonyms.

[Home](./)

## Core Outputs

- Verdict: `DO NOW` | `DO LATER` | `DISCARD`
- Impact: `LOW` | `MED` | `HIGH`
- Urgency: `LOW` | `MED` | `HIGH`
- Profile: `DIY` | `FULL`
- Mode: `PLAN` | `ARTIFACT` | `TARGET`
- State: `DRAFT` | `ACTIVE` | `STANDBY` | `PROBING` | `CLOSED` | `EXPIRED` | `DISCARDED`
- item_exec_mode / `ds`: `OK` | `SLOW` | `NO_TOOL` (derived, per-item)
- `terminal`: `YES` | `NO` (derived from state; terminal states are `CLOSED`, `DISCARDED`, `EXPIRED`)
- `closure_tier` / `ct`: `SIMULATED` | `REAL` | `-` (derived; meaningful only when `state` becomes `CLOSED`)
- `dominant_gate` / `gate`: single-cause trace tag (derived, one value)
  - `FAIL_FAST_CUT` | `FALSIFICATION` | `DEBT_CEILING` | `FRAGILITY` | `DEPENDENCY_BLOCK` | `CONFIDENCE_CAP` | `EXTERNAL_BLOCK` | `REDESIGN_TRIGGER` | `SCORE_TRIAGE` | `NONE`
- `events`: derived event trace (max 3) used as causal anchors in DELTA runs
  - `TRIGGER_<name>` | `PROBE_<probe_id>_STATE_CHANGE` | `DRIVER_<name>_CONFIDENCE_CHANGE` | `EVIDENCE_TIER_UPGRADE`
- `derived_version`: derivation ruleset id used to compute derived fields (audit compatibility)

## Trigger Tags (Item Panel)
`trigger` is the primary deterministic rule tag that caused the item transition.
It is a closed vocabulary, but it is intentionally not duplicated in full here (see the canonical kernel for the complete list).

Common trigger tags you will see in practice:
- `DEPENDENCY_BLOCK`: waiting on unresolved dependency / resource constraint
- `CIRCULAR_DEPENDENCY`: cycle detected in `depends_on` (structural modeling error; requires redesign)
- `REDESIGN_TRIGGER`: anti-stagnation / invalid plan / redesign required
- `BLOCKED_EXTERNAL`: blocked by external I/O/auth/timeout (gate `EXTERNAL_BLOCK`)

## Batch Tags (Input)

- `<<<B ...>>>`: batch header (defaults like `expire_days`, `keep_open`)
- `exec_capability`: runtime availability profile
  - batch: `<<<B ... exec_capability=NO_RUNTIME|RUNTIME_OK>>>` (default `NO_RUNTIME`)
- `close_policy`: closure policy
  - batch: `<<<B ... close_policy=SIM_OK|REAL_ONLY>>>` (default `SIM_OK`)
- `ssot_scale`: output noise control (output-only; SSOT semantics do not change)
  - batch: `<<<B ... ssot_scale=MIN|DEBUG>>>` (default `MIN`)
- `resource_pool`: optional batch resource capacity list (for concurrency)
  - batch: `<<<B ... resource_pool=R1:1,R2:2>>>`
- `<<<I ...>>> ... <<<END>>>`: one decision item (must have stable `id`)
- `ask_user`: external-data question mode
  - batch: `<<<B ... ask_user=NONE|ALLOW>>>` (default `NONE`)
  - per-item: `<<<I ... ask_user=INHERIT|NONE|ALLOW>>>` (default `INHERIT`)
 - `uses`: optional per-item resource usage list (must refer to `resource_pool` ids)
   - item: `<<<I ... uses=R1:1,R2:1>>>`
 - `reversibility`: optional per-item reversibility class
   - item: `<<<I ... reversibility=REVERSIBLE|COSTLY_REVERSIBLE|IRREVERSIBLE>>>`
- Inside each item exactly one:
  - `<<<PLAN>>>`: bullet plan or numbered steps
  - `<<<ARTIFACT>>>`: artifact to audit/optimize (prompt/code/doc/etc)
  - `<<<TOOL>>>` + `<<<TARGET>>>`: tool audits a target artifact (must not invert)
- Optional:
  - `<<<PREV>>>`: previous machine state for the item (enables delta-only ping-pong)
  - `<<<NEW_EVIDENCE evidence_tier=HISTORICAL|REAL|EXPERIMENT>>>`: new evidence object (required to revisit CLOSED items with the same id)
  - `<<<CHANGE>>>`: short change declaration (allowed to revisit SIMULATED-closed items without new evidence)

## Evidence

- rating labels: `LOW` | `MED` | `HIGH`
- confidence labels: `HIGH` | `MED` | `LOW` | `VERY LOW`
- evidence_tier: `SIMULATED` | `DERIVED` | `HISTORICAL` | `REAL` | `EXPERIMENT`
- `best_evidence_tier`: derived max tier seen for the item (used for caps and event trace)
  - `SIMULATED` | `DERIVED` | `HISTORICAL` | `REAL` | `EXPERIMENT`

Interpretation:
- `SIMULATED` / `DERIVED` are weak evidence in Complex/Chaotic domains.
- `REAL` / `EXPERIMENT` are the default path to HIGH confidence when uncertainty matters.

## Probes

Probe status:
- `ACTIVE` | `INCONCLUSIVE` | `VALIDATED` | `FALSIFIED` | `EXPIRED`

Probe fields (must be explicit for debt protection):
- `target_driver`
- `kill_switch`
- `decision_context` and the LITE fields:
  - `decision_context_id`
  - `decision_context_driver`
   - `decision_context_metric`
   - `constraint_signature` (canonical signature; required for new probes)
   - `context_match`: `YES` | `NO` | `UNKNOWN`
   - `representativeness`: `HIGH` | `MED` | `LOW`
     - optional structural override token (audit-only): `OVERRIDE_REPR(owner)`
       - format: `representativeness: MED OVERRIDE_REPR(peerA)` (base bucket unchanged)
   - `expires_at` (TTL)
   - `adopted_at` (per adopter timestamp)

Probe log margin fields (optional but structured):
- `kill_switch_threshold`: number | `UNKNOWN`
- `kill_switch_value`: number | `UNKNOWN`
- `kill_switch_margin`: `PASS_WIDE` | `PASS_NARROW` | `FAIL` | `UNKNOWN`

`constraint_signature` canonical form (keys and order are fixed):
`ENV=<prod|staging|dev|sim>|SCALE=<XS|S|M|L|XL>|DATA=<R|S|H>|TIME=<1m|10m|60m|1d>|CONSTRAINTS=<id1,id2,...>`

## Ping-Pong

- `DELTA-only`: when `<<<PREV>>>` exists, only output changes (one BREAK + one PATCH).
- `contested`: `YES` means the next DELTA must address the contested point explicitly.

## Panels

- `[USER_PANEL]`: human-readable summary at the top (derived-only, no new semantics)
- `[PANEL]`: machine panel (short, strict)
- `[HUMAN_TABLE]`: markdown table after the code block (derived-only from `[QUEUE]`, for human scanning; UI-only, must not be used as SSOT)
- `[HUMAN_TABLE_OPS]`: optional second markdown table after `[HUMAN_TABLE]` (derived-only from `[QUEUE]` + `[ITEM_PANEL]` + `[DERIVED]` + `[PROBE_LOG]` when present)
- `[RISK_DASHBOARD]`: optional derived-only risk hotspot table after `[HUMAN_TABLE]` (derived-only from `[QUEUE]` + `[ITEM_PANEL]` short fields)

## Consumer Commands (Human Render)

- `REPORT`: consumer-layer command. Paste a PA_PVP snapshot and get a derived-only full human report.
- `REPORT LITE`: consumer-layer command. Paste a PA_PVP snapshot and get a 1-page derived-only human report.

Hard rules:
- Reports MUST be derived-only from the pasted snapshot (no new facts, no changes to verdict/gates/steps).
- If something is missing, output must say `missing in snapshot`.
- If the snapshot is missing or malformed, output must be `ERROR: missing or invalid snapshot` and stop.
- Reports MUST NOT include any code blocks.

AskUser output (panel-level, single question max):
- `AskUser: NONE`
- or `AskUser: <id>: <one minimal evidence request>`

## Steps (Execution Logging)

Step 1 priority (hard, deterministic):
- If multiple rules require a specific "Step 1 MUST ..." action (reopen mitigation, cycle redesign, probes, dependency blocks, conservation redesign), the kernel selects Step 1 by a fixed priority order.
- Lower-priority requirements that remain relevant must appear as the earliest non-conflicting later step (not dropped).

Probe executability constraint (hard):
- If an item enters `PROBING` with unresolved dependencies, the Step 1 probe/acquisition must be dependency-independent (must not require dependency output or blocked dependent execution).

Cheap-First Probe note:
- "Cheap-First" is a design constraint for probe steps (timebox<=10m, binary), not a scheduling override over Step 1 priority.

Step status markers (inline on step lines):
- `status=COMPLETED`: step executed in this run and produced an output
- `status=COMPLETED_SIMULATED`: step completed using simulated/derived outcome (required in `exec_capability=NO_RUNTIME` runs)
- `status=FAILED`: step executed but hit `fail_if` (must trigger the relevant gate deterministically)
- `status=FAILED_EXCEPTION`: step failed due to runtime exception in `exec_capability=RUNTIME_OK` runs (must use `EXCEPTION_*` output token)

Runtime exception output tokens (closed vocabulary, RUNTIME_OK only):
- `EXCEPTION_IO`
- `EXCEPTION_TIMEOUT`
- `EXCEPTION_AUTH`
- `EXCEPTION_INVARIANT`
- `EXCEPTION_DEPENDENCY`

## Derived Counters (Optional Memory)

`counters` is a derived, scan-friendly memory line inside `[DERIVED]`:
- `tb=<timebox_total_m>/<timebox_budget_m>`
- `cx=<complexity_score>/<complexity_budget>`
- `pc=<probe_cycle_total>`
- `nm=<near_miss_total>`
- `nmd=<deterministic_near_miss_total>`
- `inc=<probe_inconclusive_total>`
- `ovr=<override_repr_total>`
- `rb=<reopen_block_total>`

Reopen anti-loop memory (derived, authoritative):
- `reopen_total`: how many times the item has been reopened after being `CLOSED` (carried across `<<<PREV>>>`). When `reopen_total>=1`, reopening is blocked (`trigger=REOPEN_BLOCKED_BUDGET`).

Counters are log-derived:
- `counters:` is scan-only and may be omitted. If omitted, derive the same quantities from `[PROBE_LOG]` using the kernel definitions.

Debt and complexity:
- If `falsification_debt=OPEN(...)`, it increases complexity tax (soft disincentive) in addition to any hard debt ceiling guardrail.

## Anti-drift Rules (Vocabulary)

Avoid:
- "depends" as a verdict
- vague steps ("improve", "optimize", "make better") without verifiable action

Prefer:
- explicit drivers
- explicit cause/effect in the impact graph
- explicit probes with kill-switch and representativeness

## Emission Precedence (Output-only)

- `MIN_SURFACE`: if `profile=DIY` or `impact=LOW` and the item is not entering `PROBING`, the item output should stay minimal:
  - `[ITEM_PANEL]` + `[DERIVED]` + `[STEPS]` + `[NEXT]`
  - probe harness blocks are emitted only when the item is actually `PROBING`

`[NEXT]` semantics:
- If `new_state` is `ACTIVE/PROBING`, `[NEXT]` is the next executable/measurable action.
- If `new_state` is `STANDBY`, `[NEXT]` may be omitted; if present, treat it as "next planned action on revive" (not necessarily executable now).
