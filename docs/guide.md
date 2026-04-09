---
title: "Guide"
description: "How to run PA_PVP - a PLUTONIUM-like Adversarial Peer Validation Protocol (AI-native kernel, human supervision)."
---

# PA_PVP - a PLUTONIUM-like Adversarial Peer Validation Protocol (Guide)
PA_PVP is a rule-ordered decision protocol optimized for **movement under uncertainty**.
It is designed as an **AI-native decision kernel** with **human supervision**.

[Home](./)

If you want the fastest copy/paste path, start with [QUICKSTART.md](https://github.com/XxYouDeaDPunKxX/PA-PVP/blob/main/QUICKSTART.md).
Read this guide when you want the operating model, the state transitions, and the reasoning behind the protocol mechanics.
It assumes you care about how PA_PVP works, not just how to run the first batch.

## Two layers (do not mix them)
PA_PVP has two ways to use it:

1) **Kernel (protocol)**
   - File: [PA_PVP_full_v9.8.0_canonical.txt](https://github.com/XxYouDeaDPunKxX/PA-PVP/blob/main/PA_PVP_full_v9.8.0_canonical.txt)
   - Use this when you want a rule-ordered, stateful, batch-safe AI-to-AI protocol.

2) **Quickstart (copy/paste template)**
   - File: [QUICKSTART.md](https://github.com/XxYouDeaDPunKxX/PA-PVP/blob/main/QUICKSTART.md)
   - Use this when you want the simplest copy/paste workflow to run the kernel.

We call it the "kernel" because of its internal structure: this choice is intentional to reduce drift and noise: one rule-ordered SSOT core enforces invariants and produces the state transition, while everything else (Quickstart, REPORT, orchestration policy) stays a consumer layer that must not change SSOT semantics.

## What the kernel returns (always)
Each run returns:
- a forced verdict: `DO NOW` / `DO LATER` / `DISCARD`
- one `[NEXT]` action (executable if `ACTIVE/PROBING`; optional/planned-only when `STANDBY`)
- up to five atomic executable steps

For human scanning, the output also includes a derived markdown table (`[HUMAN_TABLE]`) after the code block.
The `[QUEUE]` and `[HUMAN_TABLE]` lines include a `gate:` token (dominant gate) so you can see the primary blocker without opening item blocks.
`[HUMAN_TABLE]` is UI-only: models must not treat it as SSOT or as machine input; analysis must rely on the single code block only.
If `[PANEL]` marks the output invalid, the runtime fail-closes to a minimal diagnostic surface and that output must not be reused as machine state.

If it cannot produce an executable next action, the item is invalid (or must be discarded/parked).

## Orchestrator policy (optional, external)
Keep the kernel guardrails hard and rule-ordered. If you have an orchestrator/runner that chooses what to execute next across items, use this simple rule:
- Among actions allowed by gates and Step 1 priority, choose the next action that maximizes information ROI relative to Cost of Delay (cheap-first when possible).

## REPORT / REPORT LITE (human render, no-look, derived-only)
If you want a human-readable report without adding narrative drift to the SSOT loop, use a second pass:

Input format (consumer-layer command):
- First line: `REPORT` or `REPORT LITE`
- Next lines: paste ONE PA_PVP snapshot from a prior kernel run (the code block). Extra derived text after the code block is OK and will be ignored.

Note:
- The canonical kernel supports these commands as a non-SSOT routing mode, so you can invoke them without loading a separate prompt.

Hard rules:
- The report MUST be derived-only from the pasted snapshot.
- It MUST NOT change verdicts/gates/triggers/steps or introduce new facts.
- If a value is missing in the snapshot, the report MUST say `missing in snapshot` (no invention).
- The report output MUST NOT be used as SSOT input (`<<<PREV>>>`); it is a rendering only.
- If the snapshot is missing or malformed, output: `ERROR: missing or invalid snapshot` and stop.
- The report MUST NOT include any code blocks (to avoid being confused with a PA_PVP snapshot).

`REPORT LITE` (1 page, scan-first) MUST include:
- Batch overview: counts by state; list ids `DO NOW`; list ids blocked (top 5 by `gate:`).
- For each `DO NOW` / `PROBING` item: `id`, `gate`, `trigger`, `critical_driver`, `conf`, and the `[NEXT]` token (if present).
- A short "What to do next" list: execute `[NEXT]` for `ACTIVE/PROBING` items; for `STANDBY`, list the next planned action (if `[NEXT]` present).

`REPORT` (FULL, derived-only) MUST include:
- Everything in `REPORT LITE`, plus per-item details for all non-terminal items:
  - `state`, `verdict`, `gate`, `trigger`, `closure_tier`, `best_evidence_tier`, `falsification_debt`
  - steps list (S1..S5) with statuses when present
  - `[NEXT]` semantics: executable for `ACTIVE/PROBING`; planned-only if `STANDBY`
- Fixed metrics (only if derivable from snapshot):
  - `timebox_utilization_pct = 100 * tb_total / tb_budget` (prefer `counters tb=.../...`; else sum `timebox<=Nm`)
  - `complexity_utilization_pct = 100 * cx_score / cx_budget` (prefer `counters cx=.../...`; else use complexity tax formula)
  - `step_completion_pct = 100 * completed_steps / steps_count` (from `status=COMPLETED*` markers)
  - `probe_resolution_pct = 100 * resolved_probe_ids / probe_ids_total` (from `[PROBE_LOG]` statuses)
  - If a metric cannot be derived, print `missing in snapshot`.

### Optional: HUMAN_TABLE_OPS (scan-friendly console)
In addition to the mandatory `[HUMAN_TABLE]`, the runtime may output an optional second table:
- `[HUMAN_TABLE_OPS]` (outside the code block; derived-only)

It is derived from `[QUEUE]` + per-item `[ITEM_PANEL]`/`[DERIVED]` and is designed to answer quickly:
**"Is anything ready right now?"**

It is ignored by SNAPSHOT-AS-INPUT just like `[HUMAN_TABLE]`.

### Optional: RISK_DASHBOARD (batch hotspots)
The runtime may also emit an optional derived-only block after the code block:
- `[RISK_DASHBOARD]` (outside the code block; derived-only)

It aggregates cross-item risk without introducing new kernel fields:
- derived from `[QUEUE]` + `[ITEM_PANEL]` short fields only (driver, confidence, debt, gates)
- ignored by SNAPSHOT-AS-INPUT (rendering only)

This answers: **"Which drivers are fragile across the batch right now?"**

## SSOTScale (noise control, output-only)
PA_PVP supports an output-only scaling flag (no SSOT semantics change):
- batch: `<<<B ... ssot_scale=MIN|DEBUG>>>` (default `MIN`)

Behavior (rule-ordered):
- `MIN`: prints only the operational console for non-`<<<PREV>>>` items (panel/queue + `ITEM_PANEL`/`DERIVED`/`STEPS`/`NEXT` and probe logs if present).
- `DEBUG`: prints the full item structure.
- In `MIN`, any item with `contested=YES` or `dominant_gate != NONE` MUST be emitted in `DEBUG` (auto-escalation).

## When to use PA_PVP
Use it when:
- the decision has real operational impact
- trade-offs are non-trivial
- you want to prevent analysis loops and force contact with reality

Do not use it for:
- free-form brainstorming
- pure exploration with no actionable leverage

## Operating modes (derived per item)
The kernel derives an execution mode per item:
- `OK`: fast loop (execute now)
- `SLOW_MODE`: systemic decision (probe-first)
- `NO_TOOL`: bypass the deep loop (execute or discard directly)

Terminality is separate from the mode:
- an item can be `CLOSED` (terminal) while its `ds` still reads `OK/SLOW/NO_TOOL` for scanability.

## DIY / LOW is still real (MIN_SURFACE)
To keep the protocol truly scalable from trivial choices to systemic decisions, PA_PVP enforces a **minimal emission surface**:
- If `profile=DIY` **or** `impact=LOW` and the item is **not** entering `PROBING`, the output for that item must stay minimal:
  - `[ITEM_PANEL]` + `[DERIVED]` + `[STEPS]` + `[NEXT]`
  - no probe harness / probe logs unless the item is actually `PROBING`

This prevents "DIY masked as FULL" and keeps low-stakes runs short and executable.

## How the kernel works (copy/paste)
The kernel is batch-first. You paste:
- one batch header (optional): `<<<B ...>>>`
- one or more items: `<<<I ...>>> ... <<<END>>>`

Each item must contain exactly one mode:
- `<<<PLAN>>>` (bullets or numbered steps)
- `<<<ARTIFACT>>>` (paste the artifact to audit/optimize)
- `<<<TOOL>>>` + `<<<TARGET>>>` (tool audits target; must not invert)

Ping-pong is supported:
- If you paste the full previous output, the protocol treats it as `SNAPSHOT_AS_INPUT` and resumes delta-only.

## Simulation-first execution and closure
By default the kernel runs without runtime access:
- Batch defaults: `exec_capability=NO_RUNTIME` and `close_policy=SIM_OK`

This means the protocol is allowed to close decisions based on simulation, but it must label the closure tier:
- `ct:SIM` (simulated close)
- `ct:REAL` (close backed by new real/historical/experiment evidence)

Irreversibility safeguard:
- If an item declares `reversibility=IRREVERSIBLE`, it must not close as `ct:SIM`. It requires `ct:REAL` evidence (or it is parked with `CLOSE_BLOCKED_REAL_ONLY`).

Reopen mitigation (REAL-closed items):
- If an item was previously `CLOSED` with `ct:REAL` and is reopened due to new evidence, Step 1 must be a mitigation/compensation step (rollback/containment/compensation) before continuing execution.

Anti-loop reopen cap (mechanical):
- The kernel tracks `reopen_total` (derived, authoritative) and blocks reopening when `reopen_total>=1` with `trigger=REOPEN_BLOCKED_BUDGET`.

Dynamic priorities (policy, via `<<<CHANGE>>>`):
- When context changes (deadlines, constraints, scope), add a short `<<<CHANGE>>>` declaration so impact/urgency updates are explicit and reviewable across iterations.

## Runtime exceptions (RUNTIME_OK, closed vocabulary)
When you run with `exec_capability=RUNTIME_OK`, steps may fail due to runtime exceptions (not `fail_if`).
In that case the step output token (after `->`) MUST be one of:
- `EXCEPTION_IO`
- `EXCEPTION_TIMEOUT`
- `EXCEPTION_AUTH`
- `EXCEPTION_INVARIANT`
- `EXCEPTION_DEPENDENCY`

The kernel maps these by fixed rules to existing blockers (no new tags):
- IO / TIMEOUT / AUTH -> `BLOCKED_EXTERNAL` (gate `EXTERNAL_BLOCK`)
- DEPENDENCY -> `DEPENDENCY_BLOCK`
- INVARIANT -> `REDESIGN_TRIGGER`

## Batch debug line (TopGates)
The `[USER_PANEL]` includes a `TopGates:` line showing up to 5 highest-impact gate causes across the batch (derived from `gate:` tokens only).

## Concurrency (optional, rule-ordered)
You can model resource contention across items with two optional fields:
- batch: `resource_pool=R1:1,R2:2`
- item: `uses=R1:1,R2:1`

If multiple `DO NOW` items exceed capacity, the kernel demotes lower-priority items to `DO LATER` by fixed priority rule with `trigger=DEPENDENCY_BLOCK` (no new tags).

## Dependencies (batch, DAG-only)
`depends_on` encodes execution-order constraints across items.

Normal dependency block (expected):
- if an item depends on another item that is not `CLOSED/DISCARDED`, it must wait or explicitly waive the dependency (debt applies).

Circular dependency (structural modeling error):
- the `depends_on` graph among items present in the same batch MUST be acyclic (DAG-only).
- if a cycle is detected (including self-dependency), each item in the cycle is parked by fixed rule:
  - `new_state=STANDBY`, `verdict=DO LATER`
  - `trigger=CIRCULAR_DEPENDENCY` (dominant gate maps to `REDESIGN_TRIGGER`)
  - the earliest non-conflicting step must break the cycle (split/refactor ids, remove/replace an edge, waive one edge with debt, or discard a dominated item).
  - even though the item is `STANDBY` (no execution now), the cycle-breaking step(s) must still be recorded in `[STEPS]` as the planned actions for when it is later revived.

## Step 1 priority (hard, fixed-order)
Multiple rules can require a specific "Step 1 MUST ..." action in the same run (reopen mitigation, cycle redesign, probes, dependency blocks, conservation redesign).
To prevent contradictions, the kernel uses a fixed priority order for what Step 1 must be:

1) Reopen mitigation (previously `CLOSED` with `ct:REAL` and reopened)
2) Circular dependency break (`trigger=CIRCULAR_DEPENDENCY`)
3) Reality probe / probe harness redesign (when entering `PROBING`)
4) Dependency/resource block (unresolved `depends_on` or resource overload demotion)
5) Timebox/complexity redesign (conservation)

Lower-priority requirements are not dropped: if they are still relevant, they must appear as the earliest non-conflicting later step.

Probe executability constraint (hard):
- If an item enters `PROBING` but has unresolved dependencies, the probe/data-acquisition chosen as Step 1 must be dependency-independent:
  - it must not require the dependency's output and must not require executing blocked dependent steps.
- If no such probe can exist, the item must not enter `PROBING` in that run; it is parked under `DEPENDENCY_BLOCK` and follows dependency resolution instead.

Waiver positioning (hard):
- If you waive a dependency, the waiver action is a dependency-resolution step (often Step 1 when no higher priority applies).
- The probe/acquisition step required by the waiver must be placed at the earliest non-conflicting position after the waiver.

Cheap-first probe note:
- "Cheap-First" is a design constraint for probe steps, not a scheduling override over Step 1 priority.

## Conservation (TimeboxBudget) and Complexity Tax
To prevent "free" simulation plans, the kernel enforces:
- a derived per-item timebox budget (based on impact)
- a derived complexity tax (steps + dependencies + contested/missing ratings + hedge + resource usage)

If a plan exceeds these budgets under weak evidence (`SIMULATED/DERIVED`), it triggers `REDESIGN_TRIGGER` and must be simplified or split.

Debt and complexity:
- If `falsification_debt=OPEN(...)`, treat it as extra coordination/risk and count it in the complexity tax (soft disincentive) in addition to the hard `DEBT_CEILING` guardrail.

## Probe cycles and near-miss loops (optional, mechanical)
Inside `[DERIVED]` the kernel can expose counters:
- `derived_version`: derivation ruleset id used to compute derived fields (audit compatibility)
- `pc`: probe cycles completed (distinct probe_id reaching VALIDATED/FALSIFIED/EXPIRED)
- `nm`: near-miss count (distinct probe_id marked `PASS_NARROW`)
- `nmd`: near-miss count in the Deterministic domain (same as `nm`, but only for Deterministic domain)

Repeated near-misses under weak evidence can trigger `REDESIGN_TRIGGER` (probe harness redesign) without introducing new tags.
In Deterministic domains, any `nmd>=1` must be treated as a learning signal: confidence is capped to `MED` and the plan must include a mitigation step (second independent check or threshold redesign).

Counters are optional:
- Treat `counters:` as derived scan-only. If it is omitted, derive the same quantities from `[PROBE_LOG]` (log-first).

Anti-stagnation (policy, mechanical):
- If the same critical uncertainty remains unresolved across iterations under weak evidence (`SIMULATED/DERIVED`) and counters indicate spent cycles (`pc/nm/inc`), treat it as a stall and force a redesign:
  - change proxy/metric/kill-switch/constraint_signature, OR split the item, OR park it.
- Avoid adding hard-coded "max attempts" counters to the kernel; prefer these observable stall signals + mandatory redesign.

## AskUser: always NONE until you need it
Default behavior is simulation-first:
- `AskUser` stays `NONE` and the protocol advances using internal simulation/probes.

If you truly need external evidence, explicitly enable it:
- Batch: `<<<B ... ask_user=ALLOW>>>`
- Per item: `<<<I ... ask_user=ALLOW>>>`

Even when enabled, it can ask at most **one** minimal evidence question, and it should ask only when it is mechanically justified:
- the item is externally locked (`dominant_gate=EXTERNAL_BLOCK`, derived from a closed trigger set)
- and either the run is real-only (`close_policy=REAL_ONLY` / irreversible) or the loop has already "spent" internal cycles (`pc/nm/inc` counters).

EXTERNAL_BLOCK escalation (policy, orchestrator-side):
- If an item remains locked on `EXTERNAL_BLOCK` for multiple iterations, the orchestrator should force a proxy redesign or an evidence acquisition step (and only then consider AskUser, if enabled).

## Human workflow modes (supervision)
These are "how you run the loop", not protocol states:

### Single-host
One reviewer/agent. Best for speed and discipline.

### Multi-host (peer validation)
Two independent reviewers run the same input.
- Convergence (same verdict + similar top steps) increases confidence.
- Divergence indicates unresolved risk or missing evidence -> run probes / acquire data.

<a id="peer-convergence-a-b-consumer-level"></a>
#### Peer convergence (A/B, consumer-level)
In multi-host mode, you can compare two independent PA_PVP outputs by fixed rules **without changing the kernel**.
This is a consumer/renderer layer that reads the two snapshots and produces a convergence decision.

Two parts:
- **B (required): fixed compare rules**
- **A (optional): fixed-rule convergence report** (a derived table printed *outside* the code block)

**B) Fixed compare rules**
For each item id present in either output, extract:
- `verdict`: from `[QUEUE]` (fallback to `[ITEM_PANEL]` if missing)
- `gate`: from `[QUEUE]` `gate:` (dominant gate token)
- `conf`: `critical_driver_confidence` from `[ITEM_PANEL]`
- `tokens`: a set of step output tokens (see below)

Step output token extraction (language-agnostic):
- for each `S#:` line in `[STEPS]`, take the substring after `->` up to the next `|` (or end), trim whitespace
- keep the first **3** step tokens + the `[NEXT]` token (if present)

Fixed metrics (per item):
- `VA` (verdict agreement): `1` if equal else `0`
- `GA` (gate agreement): `1` if equal else `0`
- `SO` (step overlap): Jaccard overlap of `tokens` sets (`intersection_size / union_size`, `0..1`)
- `CD` (confidence distance): map `VERY_LOW/LOW/MED/HIGH` to `0/1/2/3`, then `abs(a-b)` (missing -> `3`)

Fixed classification:
- `CONVERGED` if `VA=1` AND `GA=1` AND `intersection_size>=1`
- `SOFT_DIVERGENCE` if `VA=1` but not `CONVERGED`
- `HARD_DIVERGENCE` if `VA=0`

Fixed next-action selection:
- if `CONVERGED` and `[NEXT]` tokens match -> execute that (only if the item is not `STANDBY`)
- else if `intersection_size>=1` -> execute the shared token with the smallest rank-sum (rank is 1..3 for steps, 0 for `[NEXT]`; ties -> lexical)
- else:
  - `SOFT_DIVERGENCE`: execute the smallest `timebox<=...` step across both outputs (ties -> lexical token)
  - `HARD_DIVERGENCE`: do not merge. acquire new evidence (probe or external data) and rerun

**A) Optional convergence report**
If you want a scan-friendly UI, render a derived block after both code blocks:
- header: `[PEER_CONVERGENCE]`
- one fixed-row markdown table with: `id | VA | GA | SO | CD | class | next_action`

This report is not SSOT and must be ignored by SNAPSHOT-AS-INPUT.

N-host consensus (optional, external):
- You can generalize the same fixed-rule compare to N independent outputs by computing per-item agreement metrics and selecting the next action via a fixed rule (e.g., majority verdict + shared token with lowest rank-sum).
- Keep this logic outside the kernel so the SSOT remains stable.

### Snapshot integrity (optional, external)
If you need tamper-evidence (legal/audit), compute and store a hash/signature of the emitted code block outside the kernel.
Do not add signatures inside `[PANEL]`/`[QUEUE]` unless you also define a strict canonicalization rule.

### Red-team (optional)
One reviewer's explicit goal is to break the plan: find failure modes and abuse cases.

## Rating rubric (LOW / MED / HIGH)
Treat labels as triage, not fake precision:
- **Cost**: time/money/complexity/dependencies to do it
- **Benefit**: value gained or risk removed if it works
- **Risk**: likelihood x impact of failure

Rule of thumb:
- if you cannot estimate -> default `MED` and declare what evidence would reduce uncertainty.

## Common failure modes (anti-patterns)
- Output says "it depends" instead of forcing a verdict.
- Steps are vague ("improve", "optimize", "refactor") without falsifiable action.
- Alternative is invented without showing it is clearly superior.
- Step ordering is arbitrary (not cost/benefit or information ROI).

## Validation (do not regress the kernel)
Protocol changes should be tested with the Examples Pack:
- Examples: [examples.md](examples.md)
- Glossary: [glossary.md](glossary.md)

Run the same examples before/after and verify invariants still hold.

## GitHub Pages (optional)
This repo includes a mini-site authored in `docs/` (see [overview.md](overview.md) for the doc landing and `index.html` for GitHub Pages).
Publish from GitHub -> Settings -> Pages -> Source: Deploy from a branch -> Branch: `gh-pages` -> Folder: `/ (root)`.

Modeling guidance (policy):
- Mutual exclusion: model alternatives inside the same item (options) or use a coordinator item; avoid cross-item auto-discard fields in the kernel.
- Knowledge transfer: when an item is falsified, promote any validated probe outcomes into `<<<NEW_EVIDENCE>>>` for related items (or create a "lessons learned" item) instead of inventing sunk-cost math in the kernel.

## Consistency upgrades (optional fields)
For probes, you can add a canonical `constraint_signature` inside `decision_context` to make `context_match` more mechanical and less subjective across iterations.

In `PROBING`, probes are adversarial-by-construction: they must include an explicit `kill_switch` and canonical `constraint_signature` (otherwise they stay `INCONCLUSIVE` and must be redesigned).
