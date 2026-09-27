---
name: engineer-agentic-ai
description: "Translate natural-language requests about AI-agent behavior, skills, MCP tools/servers, prompts, routing, context loading, memory, hooks, plugins, permissions, delegation, portability, or other agentic-system behavior into the concrete runtime mechanism and change surface. Use when a request is phrased as what the AI should understand/do rather than how the target harness actually observes, triggers, routes, persists, or enforces that behavior."
---

# 🛠️ Engineer Agentic AI

Turn desired agent behavior into mechanisms the target runtime can actually observe, trigger, execute, persist, and verify.

## 🗂️ Contents

- [🧭 Workflow](#workflow)
- [📚 Reference routing](#reference-routing)
- [🔎 Engineering translation](#engineering-translation)
- [✅ Completion standard](#completion-standard)

<a id="workflow"></a>
## 🧭 Workflow

1. **Preserve the intent.** State the desired outcome without prematurely converting it into prompt text or a specific provider feature.
2. **Model the runtime.** Identify the harness, available context/sensors, lifecycle event, routing surface, actuators, persistence, permissions, and freshness constraints.
3. **Operationalize.** Connect observable signal → trigger → decision rule → actuator → persistence → verification. If the chain is incomplete, keep the item in the wishlist/problem space rather than calling it implemented.
4. **Choose the narrowest reliable mechanism.** Prefer the lightest surface that meets frequency, miss-cost, routability, instruction-size, and enforcement requirements.
5. **Implement only after the mechanism is justified.** Edit the canonical source, validate the real propagation path, and verify behavior rather than relying on stronger wording.
6. **Keep the information architecture lean.** Put routing and invariants in the top-level skill; substantial runtime facts, provider mappings, examples, and procedures belong in references.
<a id="reference-routing"></a>
## 📚 Reference routing

Read only the references needed for the current failure or design question.

| Need | Read |
| --- | --- |
| How context, harnesses, tools, hooks, state, goals, sessions, compaction, or subagents actually behave | `references/agent-runtime-model.md` |
| How Voice/Desktop/Web/product surfaces capture input, stay responsive, render output, gate approvals, run in background, or return delegated results | `references/interaction-runtime-model.md` |
| How to turn a desired behavior into an observable, testable mechanism; mechanism selection; debugging; wishlist promotion | `references/agent-engineering-process.md` |
| Skill/MCP routing, interface-vs-implementation boundaries, trigger metadata, and when prose is being mistaken for control | `references/interface-routing-and-control.md` |
| Portability, identity-neutral wording, and concept-first provider mapping | `references/portability-and-provider-mapping.md` |
| Exact provider-specific instruction/skill/agent/hook/plugin paths | `references/provider-paths.md` |

Do not preload every reference. If current product behavior matters, verify current official documentation or direct runtime evidence instead of relying on stale provider assumptions.

<a id="engineering-translation"></a>
## 🔎 Engineering translation

Before editing, make the mapping explicit:

| Field | Answer |
| --- | --- |
| **User intent** | What outcome should be true in the user's world? |
| **Runtime interpretation** | Which observable/runtime mechanism controls it? |
| **Change surface** | Which canonical file, config, hook, tool, script, plugin, permission, scheduler, or other surface changes it? |
| **Why this surface** | Why are nearby alternatives insufficient or redundant? |
| **Verification** | What evidence will prove the mechanism actually worked? |

If the request contains `always`, `never`, “must”, or another reliability claim, explicitly test whether passive guidance can meet that requirement. Do not equate emphatic wording with enforcement.
<a id="completion-standard"></a>
## ✅ Completion standard

A change is complete only when all relevant claims below are supported:

- The target behavior is operationalized rather than merely restated as intent.
- Trigger/applicability information lives on a surface available when the decision must be made.
- Detailed instructions are progressively disclosed instead of duplicated across metadata, body, and references.
- A deterministic requirement uses the strongest available enforcement surface needed by its miss cost.
- Reusable artifacts do not accidentally depend on one person's identity, filesystem, machine, or provider vocabulary.
- Current provider capabilities were verified when mechanism choice depends on them.
- The canonical source was changed, generated/deployed copies were refreshed as needed, and session/config freshness was considered.
- Validation tests the actual mechanism or propagation path, not just Markdown wording.

Deliver the concrete edit when authorized. If the mechanism is not yet known or validated, preserve the desired outcome in the canonical wishlist with the missing evidence/decision stated explicitly; do not promote speculative instructions into production guidance.
