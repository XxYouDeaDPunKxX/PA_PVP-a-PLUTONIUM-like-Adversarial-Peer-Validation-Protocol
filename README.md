# PA_PVP - a PLUTONIUM-like Adversarial Peer Validation Protocol

![PA_PVP Banner](docs/banner.png)

You don't need to feel ready.

Doubt is enough to move forward.

In this system, uncertainty does not slow you down.
It generates the next action.

**Doubt is not a blocker.  
It is a state transition.**

Paste your current state.  
Get one verdict. One next action.  
Run the loop.

---

## Decisions that converge

You do not make a perfect choice once.
You run a system that:
- keeps what survives reality
- discards what doesn't
- unlocks what's next

Every iteration must reduce uncertainty or change the state of the world.
If it doesn't, it's invalid.

---

## Reality is inside the loop (autonomy + human sensor)

Autonomous by default (`ask_user=NONE`).
If you enable `ask_user=ALLOW`, it may ask **one** minimal real-world input when mechanically justified.

When internal evidence is not enough, it does not simulate certainty:
- it **parks cleanly**
- and (if enabled) asks for **one** minimal real-world probe
- you return with evidence
- the loop continues stronger

You are not "just copy/paste".
You are the sensor that connects the runtime to the world.

---

## Auto-learning (mechanical, not narrative)

PA_PVP is designed to learn through rule-ordered iterations:
- probes are adversarial-by-construction (try to falsify the critical driver)
- near-miss loops trigger redesign (no infinite "almost passed")
- spent cycles are tracked (`pc/nm/inc`) so the system knows when simulation is exhausted
- debt forces resolution instead of postponement
- circular dependencies (`depends_on` cycles) are treated as structural modeling errors and force redesign

A failed attempt is not a note.
It becomes a constraint.

---

## Quickstart (copy/paste)

1) Load the canonical kernel: [PA_PVP_full_v9.7.1_canonical.txt](PA_PVP_full_v9.7.1_canonical.txt)
2) Paste a batch with wrappers, run it, then paste the full output back (snapshot-as-input is supported).

```text
<<<B expire_days=30 ask_user=NONE exec_capability=NO_RUNTIME close_policy=SIM_OK ssot_scale=MIN resource_pool=>>>

<<<I id=E1 title="Short title" expire_days=60 ask_user=INHERIT uses= reversibility=>>>
<<<PLAN>>>
- <your plan in bullets or numbered steps>
<<<END>>>
```

A successful first run gives you:
- one kernel decision code block
- a forced verdict in `[QUEUE]`
- one `[NEXT]` action (or none if terminal)
- a derived `[HUMAN_TABLE]` after the code block

Typical first output shape:

```text
[QUEUE]
DO NOW | gate: ...

[NEXT]
<one executable next step>
```

Use the canonical file if you want the full machine surface.
Use [QUICKSTART.md](QUICKSTART.md) if you want the shortest operator path.
Use the docs below if you want semantics, vocabulary, and worked examples.

Docs and templates:
- Guide: [docs/guide.md](docs/guide.md)
- Glossary (closed vocabulary): [docs/glossary.md](docs/glossary.md)
- Examples pack (validation): [docs/examples.md](docs/examples.md)
- Quickstart: [QUICKSTART.md](QUICKSTART.md)

---

<details>
<summary><strong>Technical contract (SSOT-grade, rule-ordered)</strong></summary>

### What it returns (invariants)
Every valid run produces:
- a forced verdict: `DO NOW` / `DO LATER` / `DISCARD`
- a rule-ordered machine state (batch-safe)
- one executable `[NEXT]` action (or none if terminal)
- max five atomic steps

No discussions. No "it depends" as output.

The canonical kernel is the authority surface. The README is the public entrypoint. We call it the "kernel" because of its internal structure: a rule-ordered SSOT core enforces invariants and drives state transitions, while everything else (Quickstart, REPORT, orchestration policy) remains a consumer layer and must not alter SSOT semantics.

### Output structure (stable)
- Kernel decision runs: exactly ONE code block.
- Inside the code block: `[USER_PANEL]`, `[PANEL]`, `[QUEUE]`, then `[ITEM id=...]` blocks.
- After the code block:
  - mandatory `[HUMAN_TABLE]` (derived-only from `[QUEUE]`)
  - optional `[HUMAN_TABLE_OPS]` (derived-only ops console)
  
Consumer commands:
- `REPORT` / `REPORT LITE` are derived-only human render passes and MUST NOT output any code blocks (see [`docs/guide.md`](docs/guide.md)).

### Knobs (batch-level)
- `exec_capability=NO_RUNTIME|RUNTIME_OK` (default `NO_RUNTIME`)
- `close_policy=SIM_OK|REAL_ONLY` (default `SIM_OK`)
- `ssot_scale=MIN|DEBUG` (default `MIN`, auto-escalates per item when contested/gated)
- `ask_user=NONE|ALLOW` (default `NONE`)

### DIY/LOW scalability (MIN_SURFACE)
If `profile=DIY` or `impact=LOW` and the item is not entering `PROBING`, the item output must stay minimal:
`[ITEM_PANEL]` + `[DERIVED]` + `[STEPS]` + `[NEXT]` (no probe bureaucracy unless actually `PROBING`).

### AskUser (external evidence mode)
See: [docs/guide.md#askuser-always-none-until-you-need-it](docs/guide.md#askuser-always-none-until-you-need-it)

### Operational UI
- `[QUEUE]` / `[HUMAN_TABLE]` include `gate:` (dominant cause token).
- `[USER_PANEL]` includes `TopGates:` (up to 5 systemic blockers).
- Optional: `[RISK_DASHBOARD]` aggregates fragile drivers across the batch (derived-only).
- Peer convergence (A/B, consumer-level): [docs/guide.md#peer-convergence-a-b-consumer-level](docs/guide.md#peer-convergence-a-b-consumer-level)

</details>

---

## License
Unless otherwise noted, the contents of this repository are licensed under:

- Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)

See [`LICENSE`](LICENSE) and [`docs/license.md`](docs/license.md).

Attribution recipe (copy/paste):
- Project: PA_PVP - a PLUTONIUM-like Adversarial Peer Validation Protocol
- Source: <link to this repo or the specific file/version>
- License: CC BY-SA 4.0
- Changes: <state what you changed, if anything>
