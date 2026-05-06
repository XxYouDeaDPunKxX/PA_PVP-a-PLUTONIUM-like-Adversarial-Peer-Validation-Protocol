# ⚖️ PA_PVP

**🧩 a PLUTONIUM-like Adversarial Peer Validation Protocol**

PA_PVP is a copy/paste decision loop.

You give it a plan, artifact, or uncertain situation.  
It gives back one verdict, one next action, and a state you can paste back to continue.

It can stay light for small reversible choices and expand for higher-impact decisions without changing the workflow.

![PA_PVP Banner](docs/banner.png)

🧠 Doubt is not a blocker.  
🔁 It is a state transition.

---

## 🚀 Start here

Use PA_PVP when you have something that keeps staying vague:

- 🪶 a small reversible choice that still needs a clean next step
- ⚖️ a decision that will not close
- 🧪 a plan that needs pressure-testing
- 🧱 an artifact that needs validation
- 🧭 a project with too many possible next steps
- 🔁 a previous PA_PVP output you want to continue

PA_PVP does not answer with a discussion by default.

It forces the work into an operational shape:

- ⚡ `DO NOW`
- 🕓 `DO LATER`
- 🗑️ `DISCARD`
- ▶️ one `[NEXT]` action
- 📦 a snapshot you can paste back into the next run

---

## ⚡ Try it now

1. 📄 Load the canonical kernel:

   [`PA_PVP_full_v9.8.0_canonical.txt`](PA_PVP_full_v9.8.0_canonical.txt)

2. 📋 Paste this template into the chat host running PA_PVP:

```text
<<<B expire_days=30 ask_user=NONE exec_capability=NO_RUNTIME close_policy=SIM_OK ssot_scale=MIN resource_pool=>>>

<<<I id=E1 title="Short title" expire_days=60 ask_user=INHERIT uses= reversibility=>>>
<<<PLAN>>>
- <your plan in bullets or numbered steps>
<<<END>>>
```

3. ✏️ Replace the placeholder title and bullets with your real situation.

That is enough for a first run.

---

## 🧾 What comes back

A valid PA_PVP run gives you two surfaces. 🔍

First: 🧠 one machine snapshot in a single code block.

Inside it, look for:

```text
[QUEUE]
```

and:

```text
[NEXT]
```

Typical first shape:

```text
[QUEUE]
E1 | verdict: DO NOW / DO LATER / DISCARD | gate: ...

[NEXT]
S1: <one executable or planned next action>
```

Second: 📋 a human table after the code block.

👀 That table is for scanning.  
📦 The code block is the state.

---

## 🔁 Run the loop

PA_PVP is meant to be run more than once.

Basic loop:

1. 📥 paste a plan, artifact, or current state
2. ⚖️ get the verdict and `[NEXT]`
3. 🧪 execute, simulate, or inspect the next step
4. 🔁 paste the previous output back
5. 🧭 PA_PVP continues from the state instead of restarting from zero

You do not need to manually cut the previous output into pieces for normal continuation.  
A full previous PA_PVP snapshot can be pasted back as input.

---

## 🧭 Choose your entrypoint

| If you want to... | Open |
|---|---|
| ⚡ try PA_PVP quickly | [`QUICKSTART.md`](QUICKSTART.md) |
| 🧠 run the canonical protocol | [`PA_PVP_full_v9.8.0_canonical.txt`](PA_PVP_full_v9.8.0_canonical.txt) |
| 🗺️ understand the operating model | [`docs/guide.md`](docs/guide.md) |
| 📚 check field meanings and tokens | [`docs/glossary.md`](docs/glossary.md) |
| 🧪 test behavior against examples | [`docs/examples.md`](docs/examples.md) |
| 📜 reuse or adapt the project | [`LICENSE`](LICENSE) and [`docs/license.md`](docs/license.md) |

---

## 📦 Repo contents

| File | Purpose |
|---|---|
| 🧠 [`PA_PVP_full_v9.8.0_canonical.txt`](PA_PVP_full_v9.8.0_canonical.txt) | Canonical kernel. This is the authority surface. |
| ⚡ [`QUICKSTART.md`](QUICKSTART.md) | Short copy/paste path for first use. |
| 🗺️ [`docs/guide.md`](docs/guide.md) | Operating model, continuation rules, reports, evidence, and execution behavior. |
| 📚 [`docs/glossary.md`](docs/glossary.md) | Closed vocabulary for states, verdicts, gates, evidence, panels, and tags. |
| 🧪 [`docs/examples.md`](docs/examples.md) | Validation examples and invariants. |
| 📜 [`docs/license.md`](docs/license.md) | Practical license and attribution guidance. |

---

## 🧯 Practical limits

PA_PVP is not a free-form brainstorming prompt. 🚫

Scalable does not mean heavy by default. Small runs should stay short; larger or riskier runs can expand only when the structure justifies it.

It works best when there is something to push through a decision loop: a plan, artifact, trade-off, blocker, or previous state.

If the available evidence is weak, PA_PVP should not pretend certainty.  
Depending on the settings, it may 🔬 simulate a closure, 🅿️ park the item, 📡 request one minimal evidence probe, or 🗑️ discard the item.

If the output is marked invalid, do not reuse it as machine state. ⛔

---

<details>
<summary>🧩 Technical contract</summary>

## 📌 Authority surface

The canonical kernel is:

[`PA_PVP_full_v9.8.0_canonical.txt`](PA_PVP_full_v9.8.0_canonical.txt)

The README is an entrypoint.  
The quickstart, guide, glossary, reports, and examples are support surfaces.

The kernel is the source of truth for execution semantics.

---

## 🧱 Kernel output invariants

For decision runs, PA_PVP must produce:

- 📦 exactly one machine code block
- 🏷️ `STATE: DECISION.TEST.FINAL`
- 👤 `[USER_PANEL]`
- 🧭 `[PANEL]`
- ⚖️ `[QUEUE]`
- 🧩 item blocks
- ▶️ one `[NEXT]` action when the item is executable or planned
- 👀 a derived `[HUMAN_TABLE]` after the code block

The human table is derived-only.  
It must not be treated as the state source.

---

## 🪜 Scalability model

PA_PVP is designed to scale without changing the operator workflow.

A small low-impact item should not trigger the full machinery by default.  
A larger, contested, blocked, or higher-risk item can expand into the deeper contract when the state requires it.

The mechanism is `MIN_SURFACE`: low-impact or DIY items stay compact unless they actually enter probing or hit a gate that needs more structure.

---

## 🎛️ Input wrappers

Batch header:

```text
<<<B keep_open=NO expire_days=30 ask_user=NONE exec_capability=NO_RUNTIME close_policy=SIM_OK ssot_scale=MIN resource_pool=>>>
```

Item wrapper:

```text
<<<I id=E1 title="short title" keep_open=NO expire_days=30 depends_on= ask_user=INHERIT uses= reversibility=>>>
...
<<<END>>>
```

Each item uses exactly one input mode:

```text
<<<PLAN>>>
```

```text
<<<ARTIFACT>>>
```

```text
<<<TOOL>>>
<<<TARGET>>>
```

---

## ⚖️ Verdicts

Closed verdict vocabulary:

- ⚡ `DO NOW`
- 🕓 `DO LATER`
- 🗑️ `DISCARD`

No `"it depends"` verdict.

If an item cannot produce a valid next action, it must be parked, redesigned, discarded, or marked invalid by rule.

---

## 🔁 Continuation state

PA_PVP supports continuation through previous state.

You can provide item-level previous state with:

```text
<<<PREV>>>
```

You can also paste a full previous PA_PVP snapshot back as input.

The runtime should recover the previous item state and continue from it instead of treating the pasted snapshot as a new plan.

---

## 🧾 Reports

Human report commands:

```text
REPORT LITE
```

```text
REPORT
```

Reports are derived-only render passes.

They must not:

- ⚖️ change verdicts
- 🚧 change gates
- 🏷️ change triggers
- ▶️ change steps
- 🧪 introduce new facts
- 📦 output a new machine code block

A report is for reading.  
It is not reusable state.

---

## 📡 External evidence mode

Default behavior:

```text
ask_user=NONE
```

If enabled:

```text
ask_user=ALLOW
```

PA_PVP may ask for one minimal real-world input when mechanically justified.

The user is not treated as a passive copy/paste operator.  
The user can become the evidence source that connects the run to external reality.

---

## 🧪 Evidence and closure

Default batch profile:

```text
exec_capability=NO_RUNTIME
close_policy=SIM_OK
```

This allows simulated closure when real execution is unavailable, but simulated closure must remain labeled.

Important distinction:

- 🧪 `ct:SIM` = simulated closure
- 🧾 `ct:REAL` = closure backed by real, historical, or experimental evidence

Irreversible decisions must not close as simulated.

---

## 🧭 Gates and blockers

PA_PVP uses closed gate vocabulary to keep outputs stable across iterations.

Common gates include:

- 🪓 `FAIL_FAST_CUT`
- 🧪 `FALSIFICATION`
- 🧾 `DEBT_CEILING`
- 🧨 `FRAGILITY`
- 🔗 `DEPENDENCY_BLOCK`
- 📉 `CONFIDENCE_CAP`
- 📡 `EXTERNAL_BLOCK`
- 🧱 `REDESIGN_TRIGGER`
- ⚖️ `SCORE_TRIAGE`
- ✅ `NONE`

Use [`docs/glossary.md`](docs/glossary.md) for field meanings.

---

## 🧷 Dependencies and resources

PA_PVP can model:

- 🔗 `depends_on`
- 🧰 `resource_pool`
- 📌 `uses`

Circular dependency is a modeling error.  
It should force redesign instead of waiting forever.

Resource overload can demote lower-priority items by fixed rule.

---

## 🧪 Validation examples

Use [`docs/examples.md`](docs/examples.md) as a behavior check when editing the kernel or documentation.

The examples are not just demos.  
They are equivalence probes for whether the protocol still behaves correctly after changes.

</details>

---

## 📜 License

Unless otherwise noted, this repository is licensed under:

**Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0).**

See [`LICENSE`](LICENSE) and [`docs/license.md`](docs/license.md).

Attribution recipe:

- 🧩 Project: PA_PVP - a PLUTONIUM-like Adversarial Peer Validation Protocol
- 🔗 Source: <link to this repo or the specific file/version>
- 📜 License: CC BY-SA 4.0
- ✏️ Changes: <state what you changed, if anything>
