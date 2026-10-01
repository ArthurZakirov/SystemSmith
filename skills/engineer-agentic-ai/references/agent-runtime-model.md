# ⚙️ Agent runtime model

Documentation snapshot: 2026-10-01. Re-verify provider-specific claims from their linked primary sources before relying on them as current behavior.

Use this reference to see how interaction enters an agent runtime, context and capabilities reach the model, actions affect the environment, and state persists. The map is the backbone; the taxonomy immediately below explains the same numbered regions and named components.

<a id="runtime-map"></a>
## 🗺️ Runtime and mechanism map

```mermaid
flowchart TB
    USER((👤 User))

    subgraph ROUTING["1 · 💬 Interaction routing"]
        direction LR
        TYPED([⌨️ Typed text])
        VOICE([🎙️ Realtime voice input])
        SPEECH([🗣️ Frontend speech model])
        SPOKEN([💬 Spoken response])
        BACKEND[[⚙️ Backend agent turn]]

        TYPED -->|normally enters| BACKEND
        VOICE -->|utterance| SPEECH
        SPEECH -->|direct reply| SPOKEN
        SPEECH -->|selected delegation| BACKEND
    end

    subgraph CONTEXT["2 · 🗂️ Context and durable state"]
        direction TB
        subgraph INPUTS["📚 Context inputs"]
            direction TB
            subgraph BASELINE_INPUTS["🔒 Harness-controlled baseline"]
                direction TB
                SYSTEM["🧾 System instructions"]
                DEVELOPER["🏗️ Developer instructions"]

                SYSTEM ~~~ DEVELOPER
            end
            subgraph GUIDANCE_INPUTS["👤 User- and project-controlled guidance"]
                direction TB
                CURRENT_REQUEST["🎯 Current user request"]
                EARLIER_MESSAGES["💬 Earlier user messages"]
                SCOPED["📜 Scoped instruction files<br/>AGENTS.md / CLAUDE.md"]
                SKILL_BODY["📘 Selected SKILL.md body /<br/>loaded workflow guidance"]
                SUPPORTING["🗃️ Supporting references / assets"]
                INJECT_NOTE["discover / inject at session or context build<br/>retained active content repeats in later inputs"]

                CURRENT_REQUEST ~~~ EARLIER_MESSAGES ~~~ SCOPED ~~~ INJECT_NOTE ~~~ SKILL_BODY ~~~ SUPPORTING
            end
            subgraph CAPABILITY_INPUTS["🧰 Available capabilities"]
                direction TB
                SKILL_CATALOG["🗂️ Skill metadata /<br/>catalog entries"]
                TOOL_SCHEMAS["🛠️ Tool / MCP schemas<br/>and capability descriptions"]
                PLUGINS["🧩 Plugins / packaging"]

                SKILL_CATALOG ~~~ TOOL_SCHEMAS ~~~ PLUGINS
            end
            subgraph EVIDENCE_INPUTS["🔎 Runtime state and evidence"]
                direction TB
                HISTORY["🧠 Conversation history or<br/>compacted replacement state"]
                RESOURCES["📚 Retrieved resources /<br/>references"]
                PRIOR_RESULTS["👁️ Prior tool results /<br/>observations"]
                COMPACT_NOTE["Compaction replaces older active context<br/>with opaque item + retained items"]

                HISTORY ~~~ RESOURCES ~~~ PRIOR_RESULTS ~~~ COMPACT_NOTE
            end
            ASSEMBLE([Assemble context])

            BASELINE_INPUTS --> ASSEMBLE
            GUIDANCE_INPUTS --> ASSEMBLE
            CAPABILITY_INPUTS --> ASSEMBLE
            EVIDENCE_INPUTS --> ASSEMBLE
        end
        CURRENT[(🗂️ Current model context)]
        SKILLS["📘 Skills / retrieval"]
        PERSIST[(💾 Persistence<br/>thread / session state)]
        GOAL([🎯 Optional Codex Goal<br/>thread-scoped contract])

        SKILLS -->|selected content| INPUTS
        PERSIST -->|load state| INPUTS
        ASSEMBLE -->|assembled context| CURRENT
        CURRENT -->|persist state| PERSIST
        GOAL -.->|optional completion contract| CURRENT
    end

    subgraph LOOP["3 · 🔁 Agent loop and runtime controls"]
        direction TB
        MODEL{{🧠 Model inference<br/>Decide next step}}
        GATE{{🔐 Permissions / approvals}}
        ACTION[[🛠️ Tool / MCP action]]
        ENV[(🌐 External environment)]
        OBS[/👁️ Observation / tool result/]
        UPDATE[(🗂️ Context update)]
        RESULT([💬 Response / result])
        DELEGATE[[🤝 Delegation / subagent]]
        HOOKS["⚡ Hooks / events / automations"]
        VALIDATE["✅ Validators / tests"]

        MODEL -->|complete| RESULT
        MODEL -->|act| GATE
        GATE -->|allowed| ACTION
        ACTION -->|interact| ENV
        ENV -->|observe| OBS
        OBS -->|add result| UPDATE
        UPDATE -->|next inference| MODEL
        MODEL -->|delegate| DELEGATE
        DELEGATE -->|returns| OBS
        HOOKS -.->|trigger / gate| GATE
        VALIDATE -->|evidence| OBS
    end

    subgraph DISCLOSURE["4 · 📘 Progressive skill disclosure"]
        direction TB
        D_META["🗂️ Skill metadata / catalog"]
        D_CONTEXT[(🗂️ Rendered model context)]
        D_SELECT{{"🧠 Model selects<br/>matching skill"}}
        D_BODY["📘 Load selected<br/>SKILL.md body"]
        D_UPDATE[(🗂️ Context update)]
        D_NEXT{{"🧠 Next model inference"}}
        D_SUPPORT["🗃️ Supporting references / assets"]

        D_META -->|rendered into| D_CONTEXT
        D_CONTEXT -->|selection decision| D_SELECT
        D_SELECT -->|match / invoke<br/>progressive disclosure| D_BODY
        D_BODY -->|loaded content| D_UPDATE
        D_UPDATE --> D_NEXT
        D_BODY -.->|load only when needed| D_SUPPORT
        D_SUPPORT -.->|retrieved content| D_UPDATE
    end

    USER -->|types| TYPED
    USER -->|speaks| VOICE
    SPOKEN -->|played to| USER

    BACKEND -->|starts with| CURRENT
    CURRENT -->|rendered context| MODEL
    RESULT -->|delivered to| USER

    HOOKS -.->|inject / trigger| UPDATE
    CONTEXT ~~~ DISCLOSURE

    classDef actor fill:#1D4ED8,stroke:#1E3A8A,color:#FFFFFF,stroke-width:3px
    classDef message fill:#DBEAFE,stroke:#2563EB,color:#172554,stroke-width:2px
    classDef surface fill:#CFFAFE,stroke:#0891B2,color:#164E63,stroke-width:2px
    classDef context fill:#F1F5F9,stroke:#64748B,color:#0F172A,stroke-width:2px
    classDef decision fill:#EDE9FE,stroke:#7C3AED,color:#3B0764,stroke-width:2px
    classDef action fill:#FFEDD5,stroke:#EA580C,color:#431407,stroke-width:2px
    classDef observation fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px
    classDef control fill:#FEE2E2,stroke:#DC2626,color:#450A0A,stroke-width:2px
    classDef persistence fill:#FEF3C7,stroke:#D97706,color:#451A03,stroke-width:2px
    classDef delegation fill:#CCFBF1,stroke:#0F766E,color:#134E4A,stroke-width:2px
    classDef note fill:#F8FAFC,stroke:#94A3B8,color:#334155,stroke-width:1px

    style BASELINE_INPUTS fill:#FFF7ED,stroke:#C2410C,stroke-width:2px,color:#7C2D12
    style GUIDANCE_INPUTS fill:#EFF6FF,stroke:#2563EB,stroke-width:2px,color:#1E3A8A
    style CAPABILITY_INPUTS fill:#F0FDFA,stroke:#0F766E,stroke-width:2px,color:#134E4A
    style EVIDENCE_INPUTS fill:#FAF5FF,stroke:#7E22CE,stroke-width:2px,color:#581C87

    class USER actor
    class TYPED,VOICE,SPOKEN,RESULT message
    class SPEECH,BACKEND surface
    class SYSTEM,DEVELOPER,CURRENT_REQUEST,EARLIER_MESSAGES,SCOPED,SKILL_BODY,SUPPORTING,SKILL_CATALOG,TOOL_SCHEMAS,PLUGINS,HISTORY,RESOURCES,PRIOR_RESULTS,ASSEMBLE,CURRENT,UPDATE,D_META,D_CONTEXT,D_BODY,D_UPDATE,D_SUPPORT context
    class MODEL,D_SELECT,D_NEXT decision
    class ACTION action
    class OBS observation
    class GATE,HOOKS,VALIDATE control
    class PERSIST,GOAL persistence
    class SKILLS,DELEGATE delegation
    class ENV,INJECT_NOTE,COMPACT_NOTE note
```

The map expresses the thought-action-observation trajectory described in Yao et al.'s [ReAct paper](https://arxiv.org/abs/2210.03629) without implying that private chain-of-thought is exposed, human-readable, or persisted. It uses **agent loop** in the sense of OpenAI's [long-horizon Codex explanation](https://developers.openai.com/blog/run-long-horizon-tasks-with-codex). The taxonomy below uses the map's exact region numbers, labels, and emoji.

<a id="runtime-taxonomy"></a>
## 🧭 Map-aligned runtime taxonomy

| Map region / component | What it does | When or how it enters or acts | Control / reliability character | Deeper evidence or reference |
| --- | --- | --- | --- | --- |
| **1 · 💬 Interaction routing** — ⌨️ **Typed text**; 🎙️ **Realtime voice input**; 🗣️ **Frontend speech model**; 💬 **Spoken response**; ⚙️ **Backend agent turn** | Routes human input either into the backend agent turn or through a frontend speech path. A voice frontend may answer directly or delegate selected work to the backend. | Before backend context assembly. Typed text normally enters the backend directly; voice behavior depends on the product surface. | Product-controlled routing. A speech model cannot be assumed to see backend instructions such as `AGENTS.md` or `CLAUDE.md`. | [`Interaction runtime model`](interaction-runtime-model.md) for desktop/web, text/voice, approvals, background work, and result delivery. |
| **2 · 🗂️ Context and durable state** — 🔒 **Harness-controlled baseline**: 🧾 **System instructions** and 🏗️ **Developer instructions** | Establishes baseline behavior the harness places into model context. | During context construction, before model inference. | High-priority model guidance, but still interpreted by the model rather than enforced like a permission gate. | Provider-specific surfaces belong in the [`Provider path reference`](provider-paths.md). |
| **2 · 🗂️ Context and durable state** — 👤 **User- and project-controlled guidance**: 🎯 **Current user request**, 💬 **Earlier user messages**, 📜 **Scoped instruction files** | Supplies the current goal, retained conversation, and filesystem-scoped rules. Codex discovers `AGENTS.md`/`AGENTS.override.md` from the identified project root through the working directory and injects applicable chunks root-to-leaf before the prompt; deeper scopes can override earlier guidance. | Discovery and injection occur at session or context-build boundaries; retained active content can reappear in later rendered inputs without a physical reread. `project_root_markers` is live configuration, so do not assume `.git` when diagnosing an installation. | Probabilistic instruction following. A loaded rule still needs observable evidence that its condition applies. | OpenAI's [Codex `AGENTS.md` guidance](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.3-codex#using-agentsmd) and the [`Provider path reference`](provider-paths.md). |
| **2 · 🗂️ Context and durable state** — 📘 **Selected `SKILL.md` body / loaded workflow guidance**; 🗃️ **Supporting references / assets**; 📘 **Skills / retrieval** | Supplies selected workflow instructions and deeper material. The skill body is the post-selection execution surface; references, scripts, templates, and assets remain on demand. | After a skill is selected; supporting material enters only when retrieved. | Selection and instruction following are probabilistic. Retrieval makes content available but does not enforce it. | OpenAI's [Skills API guide](https://developers.openai.com/api/docs/guides/tools-skills), [plugin Skills concepts](https://developers.openai.com/plugins/concepts/skills), and region 4 below. |
| **2 · 🗂️ Context and durable state** — 🧰 **Available capabilities**: 🗂️ **Skill metadata / catalog entries**; 🛠️ **Tool / MCP schemas, discovery metadata, and capability descriptions**; 🧩 **Plugins / packaging** | Advertises conditional knowledge and executable or retrievable capabilities. MCP is a runtime-extension protocol boundary, not a provider, harness, or agent. Its servers can expose resources, prompts, and tools; clients decide whether concrete tool schemas enter model context eagerly or only after a discovery/search step. Plugins package skills, MCP configuration, assets, and hooks but add no enforcement by themselves. | Capability exposure is harness-specific. Some runtimes place callable schemas directly into the initial context; others expose only a discovery surface plus enough metadata to decide whether to search, then inject matching tool definitions later. | Model-visible descriptions influence probabilistic selection. A registered tool can therefore be usable by the runtime without being present in the model's current context. The 2026-07-28 MCP protocol core is stateless, so cross-call server state needs explicit handles or application-managed persistence. | [Capability loading modes](#capability-loading-modes), [`Interface, routing, and control design`](interface-routing-and-control.md), and the [`Provider path reference`](provider-paths.md). |
| **2 · 🗂️ Context and durable state** — 🔎 **Runtime state and evidence**: 🧠 **Conversation history or compacted replacement state**; 📚 **Retrieved resources / references**; 👁️ **Prior tool results / observations** | Provides the evidence available to the next inference. Stored rollout records, retained active items, and the rendered input for one model call are distinct layers. Compaction can replace older active history with an opaque compacted item plus retained items while the full rollout remains persisted. | Loaded during context assembly and updated after observations or compaction. | The model cannot reason from evidence that never reaches this state. Compaction and retention are harness-controlled. | [`Token usage and cost model`](token-usage-and-cost-model.md) for rollout projections, rendered input, caching, compaction, and product accounting. |
| **2 · 🗂️ Context and durable state** — **Assemble context** → 🗂️ **Current model context** | Combines baseline instructions, guidance, capability descriptions, runtime evidence, and loaded state into the input rendered for one inference. | Immediately before a model call and again after context updates. | Defines the model's observable world for that inference; file presence alone does not prove effective context presence. | [`Token usage and cost model`](token-usage-and-cost-model.md#diagnostic-layers) for the distinction between stored, active, and rendered layers. |
| <a id="session-freshness"></a> **2 · 🗂️ Context and durable state** — 💾 **Persistence: thread / session state**; 🎯 **Optional Codex Goal: thread-scoped contract** | Preserves an ongoing trajectory and, optionally, its completion contract. A Goal is separate from global memory or project instructions. | Loaded when a thread/session continues and updated as the trajectory progresses. | Persistence does not guarantee that changed external configuration has been reloaded. In bounded tests, CLI `exec resume` refreshed skill metadata and project instructions, while one continuously open desktop chat retained its original skill catalog but reread an explicitly selected skill body. | Full setup, evidence, limits, and reproduction: [`Codex configuration-freshness experiments`](codex-configuration-freshness-experiments.md). OpenAI's [Goals guide](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex) covers the optional contract. |
| **3 · 🔁 Agent loop and runtime controls** — 🧠 **Model inference: decide next step** | Chooses whether to answer, act, delegate, or request more evidence from the context available for that turn. | Once the current model context is rendered; repeated after each context update. | Probabilistic inference. Stronger wording alone does not turn a decision into enforcement. | [`Agent engineering process`](agent-engineering-process.md) for operationalization and behavioral verification. |
| **3 · 🔁 Agent loop and runtime controls** — 🔐 **Permissions / approvals** | Allows, denies, or pauses protected actions for authorization. It is distinct from a hook that merely observes or advises. | Before a protected action proceeds. | Strong enforcement when the runtime actually gates the action. | [`Interaction runtime model`](interaction-runtime-model.md) when the human approval path or delivery surface matters. |
| **3 · 🔁 Agent loop and runtime controls** — 🛠️ **Tool / MCP action** ↔ 🌐 **External environment** | Acts on files, repositories, apps, APIs, browsers, operating systems, or other external state. Tool/MCP descriptions help the model select a capability; implementation code determines what happens after invocation. | After model selection and any required permission gate. | Selection is probabilistic; execution is deterministic only within code, permission, and environment limits. An MCP server does not guarantee that the model will call it. | [`Interface, routing, and control design`](interface-routing-and-control.md) for pre-selection interfaces versus post-selection implementation. |
| **3 · 🔁 Agent loop and runtime controls** — 👁️ **Observation / tool result** → 🗂️ **Context update** | Returns action results to the harness and adds the relevant evidence to a later model input. | After an action, delegated result, validator, or hook output. | Grounding is only as complete and accurate as the returned observation. Large results should be projected locally before entering context. | [🔎 Context-efficient observation](#context-observation). |
| **3 · 🔁 Agent loop and runtime controls** — 💬 **Response / result** | Completes the current backend turn and returns the result through the active interaction surface. | When the model decides the turn is complete. | Delivery behavior is product-controlled; prose is not proof that the requested external effect occurred. | [`Interaction runtime model`](interaction-runtime-model.md). |
| **3 · 🔁 Agent loop and runtime controls** — 🤝 **Delegation / subagent** | Gives a separate model worker an isolated task context. Delegation changes decomposition and context isolation, not the available sensors, actuators, or enforcement strength. | When the harness or model delegates. The worker's result returns as an observation. | Bounded by the context, tools, permissions, and environment granted to that worker. | [`Agent engineering process`](agent-engineering-process.md#behavior-validation) for end-to-end verification. |
| **3 · 🔁 Agent loop and runtime controls** — ⚡ **Hooks / events / automations** | Runs event-driven checks, injections, gates, or scheduled work without waiting for spontaneous model recall. Codex command hooks can inject context, block prompts or tool calls, rewrite tool input, warn, or run post-action checks; `PostToolUse` cannot undo an action. Documented events include `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PreCompact`, `PostCompact`, `SubagentStart`, `SubagentStop`, `Stop`, `Interrupt`, and `SessionEnd`. Parsed `prompt` and `agent` handlers are currently skipped. | When the runtime emits a supported lifecycle event, or when an external schedule/event fires. | Deterministic if the event is emitted, the supported handler runs, and its dependencies succeed. | [`Agent engineering process`](agent-engineering-process.md#mechanism-selection) for choosing the lightest sufficient control. |
| **3 · 🔁 Agent loop and runtime controls** — ✅ **Validators / tests** | Encodes checks that turn expected properties into repeatable evidence. | Explicitly in a workflow or from an event hook. | Deterministic detection for encoded conditions; cannot detect properties it does not test. | [`Agent engineering process`](agent-engineering-process.md#behavior-validation). |
| **4 · 📘 Progressive skill disclosure** — 🗂️ **Skill metadata / catalog** → 🗂️ **Rendered model context** → 🧠 **Model selects matching skill** | Keeps initial context compact while exposing enough metadata—especially `name` and `description`—for routing. | Metadata is rendered before selection; the model selects a matching skill for the current task. | Probabilistic routing. Poor or stale metadata can prevent the body from loading. | [`Interface, routing, and control design`](interface-routing-and-control.md) and the [`freshness experiments`](codex-configuration-freshness-experiments.md). |
| **4 · 📘 Progressive skill disclosure** — 📘 **Load selected `SKILL.md` body** → 🗂️ **Context update** → 🧠 **Next model inference**; 🗃️ **Supporting references / assets** | Loads the workflow only after selection, then retrieves deeper material only when needed. This is the same selected skill content shown in region 2, expanded here as a lifecycle. | The body enters after match or explicit invocation; references/assets enter through later retrieval steps. | Probabilistic execution after selection. Progressive disclosure reduces baseline context but cannot enforce a behavior that must govern every turn. | OpenAI's [Skills API guide](https://developers.openai.com/api/docs/guides/tools-skills) and [plugin Skills concepts](https://developers.openai.com/plugins/concepts/skills). |

<a id="capability-loading-modes"></a>
## 🔎 Capability loading modes

A harness can expose tools to the model in two materially different ways:

- **Eager loading:** concrete callable tool definitions are placed in the model context during request or session assembly. The model can select them directly from what it sees.
- **Deferred discovery:** the runtime initially exposes a search/discovery capability plus routing metadata for deferred namespaces, MCP servers, or tools. Concrete definitions enter model context only after the model invokes discovery and the harness returns matching tools.

This distinction changes failure diagnosis. A configured MCP server or function can exist in the runtime while its concrete tool definitions are absent from the current model context. In that case, failure to use the capability may be a **pre-invocation discovery/routing miss**, not evidence that the integration is unavailable.

OpenAI's Responses API exposes this explicitly, on models that support tool search, through [`tool_search`](https://developers.openai.com/api/docs/guides/tools-tool-search) and `defer_loading: true`; deferred MCP servers retain server-level routing metadata while individual functions load only after search. OpenAI's Agents API loads ordinary function definitions eagerly by default, supports deferred functions through tool search, and can automatically defer/discover MCP tools when the model/provider support it. These are API semantics, not proof that every ChatGPT, Codex, Claude, Cursor, or other product surface uses the same default. Verify the actual harness, version, and configuration before relying on eager or deferred exposure.

For agent-system design, treat **tool registration**, **tool discoverability**, **tool definition presence in current context**, and **tool invocation** as separate states. Instructions that require use of a structured integration may need an explicit discovery action when the harness supports deferred loading; merely preferring MCP over browser automation does not guarantee the relevant MCP tools are visible at decision time.

<a id="cross-cutting-principles"></a>
## 🧠 Cross-cutting reasoning and enforcement principles

- **Natural-language rules are conditional programs.** A rule has an observable condition and an action. Loading the rule makes it available; it does not prove the condition is satisfied. Separate **When** and **Then** visibly when that improves reviewability.
- **Information absent from context cannot affect inference.** A file, memory, resource, or changed configuration surface matters only after the harness loads or exposes it. Likewise, the model cannot perform an action for which the harness exposes no actuator.
- **Wording does not upgrade the mechanism.** `Always`, `never`, repetition, and all-caps remain prompt-level guidance. Reliability generally rises from passive prose → retrieved guidance → model classification with evidence → live tool check → validator → event hook or permission gate → external automation or system policy. Implementation cost and platform coupling usually rise with it.
- **Freshness is lifecycle-specific.** An updated file on disk and its presence in one session's effective context are separate facts. Test the exact surface, version, workspace scope, and resume/reload/restart boundary rather than assuming universal hot reload.

<a id="provider-comparison"></a>
## 🌐 Provider and harness comparison

Provider and harness products are peers; MCP is not one of them. Keep conceptual mechanisms stable, then map them onto the actual runtime.

| Provider / harness | Distinctive runtime surfaces | Comparison boundary | Deeper route |
| --- | --- | --- | --- |
| OpenAI / Codex | Hierarchical `AGENTS.md`, Agent Skills, plugins, command hooks, durable threads/sessions, optional Goals, tools, permissions, and managed compaction. OpenAI's Agents API uses a managed Codex harness while the application supplies tools and execution environment. | Availability and lifecycle behavior vary across CLI, desktop, web, and API surfaces. | [`Provider path reference`](provider-paths.md), [`Portability and provider mapping`](portability-and-provider-mapping.md), and [`Codex configuration-freshness experiments`](codex-configuration-freshness-experiments.md). |
| Anthropic / Claude Code | `CLAUDE.md`/rules, skills, subagents, hooks, output styles, and system-prompt additions enter context or run at different lifecycle points. Hooks are event-driven commands; skills and subagents guide or delegate model work rather than firing deterministically on every matching event. | Claude Code's **agentic turns** and session surfaces should not be assumed equivalent to Codex turns, threads, or resume boundaries. | Anthropic's [Claude Code CLI reference](https://docs.anthropic.com/en/docs/claude-code/cli-usage), [`Provider path reference`](provider-paths.md), and [`Portability and provider mapping`](portability-and-provider-mapping.md). |
| Cursor, OpenCode, and other harnesses | May expose analogous instructions, skills, tools, hooks, permissions, and persistence through different names or omit them entirely. | Verify the actual discovery, injection, event, enforcement, and persistence semantics instead of transferring another product's behavior. | [`Portability and provider mapping`](portability-and-provider-mapping.md) and [`Provider path reference`](provider-paths.md). |

<a id="context-observation"></a>
## 🔎 Context-efficient observation

Local computation is not itself model context. Only the command or tool result returned to the model consumes the active context window. Parse large artifacts locally and return a compact projection: bound matches, select relevant fields, cap lines, and summarize counts.

Avoid broad recursive `rg`, web search, resource-listing, or rollout-log queries when a narrower target answers the question. Self-referential rollout searches can reproduce earlier prompts and nested tool outputs, injecting thousands of duplicate tokens into the context being diagnosed. Use the [`Token usage and cost model`](token-usage-and-cost-model.md#diagnostic-sequence) for rollout, cache, compaction, and usage projections.
