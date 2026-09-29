---
name: engineer-agentic-ai
description: "Translate natural-language requests about AI-agent behavior, skills, MCP tools/servers, prompts, routing, context loading, memory, hooks, plugins, permissions, delegation, portability, or other agentic-system behavior into the concrete runtime mechanism and change surface. Use when a request is phrased as what the AI should understand/do rather than how the target harness actually observes, triggers, routes, persists, or enforces that behavior."
---

# 🛠️ Engineer Agentic AI

Translate desired agent behavior into mechanisms the target runtime can observe, trigger, execute, persist, and verify.

## 🗂️ Contents

- [🎯 Purpose and applicability](#purpose)
- [🧭 Engineering framework](#framework)
- [📚 Reference routing](#reference-routing)
- [✅ Quality gates](#quality-gates)

<a id="purpose"></a>
## 🎯 Purpose and applicability

Use this skill when a request describes what an agent should understand, decide, remember, trigger, prevent, or accomplish, but the concrete runtime mechanism is missing or uncertain. It applies to instructions, skills, routing, tools, MCP servers, hooks, permissions, plugins, automations, persistence, delegation, and related agent-system behavior.

The outcome is an implemented and behaviorally verified mechanism—or an explicitly retained problem-space requirement when the runtime chain cannot yet be closed.

<a id="framework"></a>
## 🧭 Engineering framework

Use this as the single end-to-end process:

1. **Preserve the problem-space intent.** State the observable outcome without prematurely turning it into prompt wording or a provider feature.
2. **Route the investigation.** Load only the narrowest references required by the current uncertainty using [Reference routing](#reference-routing).
3. **Operationalize the behavior.** Connect observable signal and evidence → trigger/lifecycle event → decision rule → actuator → persistence → verification. Include reliability and miss-cost requirements. If a required link is unavailable, keep the outcome in the problem space rather than calling it implemented.
4. **Choose the mechanism and surface.** Select the lightest available mechanism that can meet the required reliability; verify current provider capabilities when the choice depends on them.
5. **Implement canonically.** Change the authoritative instruction, skill, hook, tool, permission, plugin, script, automation, or configuration surface and validate its propagation path. Translate problem-space role language into the runtime's actual perspective: production guidance should address the executing model as `you` (and the human as `the user`) or describe directly observable state. Do not write conditions such as `When the orchestrator...` or `When this chat...` unless the runtime can actually observe that identity or state.
6. **Validate real behavior.** Exercise the mechanism through a representative agent interaction or runtime event. Test the observable outcome, not only syntax, file presence, or plausible prose.
7. **Diagnose and iterate.** On failure, trace guidance freshness, trigger match, visible evidence, rule specification, routing/selection, actuation, permissions, persistence, and propagation; adjust the mechanism or test and run it again. Use positive, negative, boundary, and repeated cases when behavior is model-mediated or routing-dependent.
8. **Promote and hand off.** Promote only after the required behavioral evidence exists. Refresh deployed/generated copies, record remaining unknowns and tested scope, and retire or mark any superseded wishlist item.

The detailed operationalization, validation, debugging, and promotion methods live in the [`Agent engineering process`](references/agent-engineering-process.md); do not recreate parallel procedures in this file.

<a id="reference-routing"></a>
## 📚 Reference routing

Load a reference only when its condition matches the current decision or failure.

| When | Read | Use it for |
| --- | --- | --- |
| Translating, validating, debugging, or promoting an agent behavior | [`Agent engineering process`](references/agent-engineering-process.md) | Operationalization, mechanism selection, behavioral testing, failure diagnosis, and wishlist promotion. |
| Establishing how context, tools, hooks, state, sessions, compaction, or delegation behave | [`Agent runtime model`](references/agent-runtime-model.md) | Runtime mechanics and Codex-specific lifecycle evidence. |
| Diagnosing selection, metadata, interface boundaries, or missing enforcement | [`Interface, routing, and control design`](references/interface-routing-and-control.md) | Pre-selection routing versus post-selection execution and control strength. |
| Reasoning about voice/text, desktop/web, approvals, background work, or result delivery | [`Interaction runtime model`](references/interaction-runtime-model.md) | Human-to-runtime input, lifecycle, and return-path behavior. |
| Designing a reusable artifact across identities, machines, operating systems, or providers | [`Portability and provider mapping`](references/portability-and-provider-mapping.md) | Concept-first portability and provider-neutral wording. |
| Needing an exact provider filename, directory, manifest, or settings location | [`Provider path reference`](references/provider-paths.md) | Current provider-specific path mappings and sources. |
| Analyzing context size, tokens, caching, compaction cost, allowances, credits, or billing | [`Token usage and cost model`](references/token-usage-and-cost-model.md) | Usage evidence, cache mechanics, product accounting, and optimization. |

<a id="quality-gates"></a>
## ✅ Quality gates

A change is ready only when:

- the [`operationalization chain`](references/agent-engineering-process.md#operationalization-test) is complete, or the missing link is explicitly retained as an unresolved requirement;
- the chosen mechanism and routed references support the required reliability without competing canonical rules;
- [`behavior validation`](references/agent-engineering-process.md#behavior-validation) proves the intended outcome on the stated surface and scope;
- the canonical source, propagation path, and deployed/generated state are verified; and
- the handoff names remaining unknowns and completes the applicable [`wishlist-to-production lifecycle`](references/agent-engineering-process.md#wishlist-lifecycle).
