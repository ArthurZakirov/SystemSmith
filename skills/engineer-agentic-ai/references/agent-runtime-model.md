# Agent Runtime Model

Last checked: 2026-09-27

This reference describes how agentic runtimes work: how context reaches the model, how capabilities are selected, how actions affect the environment, and how state or events persist across turns.

## Core Model

An agentic system is not just an LLM. Model the runtime as a loop with distinct layers:

1. **Environment** — files, repositories, apps, APIs, clocks, calendars, issue trackers, browsers, operating system, and other external state.
2. **Harness / orchestrator** — the product or runtime that assembles context, exposes tools, manages permissions, runs the model loop, compacts history, and may persist sessions.
3. **Context assembly** — system/developer/user messages, conversation history, scoped instruction files, selected skill metadata or bodies, tool schemas, retrieved resources, and tool results.
4. **Model inference** — the LLM receives only the context made available for that turn and predicts a response and/or tool calls.
5. **Actuation** — tools, shell commands, MCP actions, file edits, browser actions, delegated agents, or other executable capabilities affect the environment.
6. **Observation** — results from those actions are returned to the harness and may be added to later model context.
7. **Persistence** — files, session state, memory systems, databases, git, or other stores preserve information beyond one inference call.
8. **Event / automation layer** — hooks, triggers, schedules, workflows, and external processes can run without relying on the model to spontaneously remember to act.

```mermaid
flowchart TB
    U([💬 User message / goal])
    M{{🧠 Model inference<br/>Decide next step}}
    A[[🛠️ Tool / action]]
    O[/👁️ Observation<br/>Tool result/]
    F([💬 Final response / result])

    U -->|request| M
    M -->|act| A
    A -->|result| O
    O -->|update| M
    M -->|complete| F

    classDef message fill:#DBEAFE,stroke:#2563EB,color:#172554,stroke-width:2px
    classDef decision fill:#EDE9FE,stroke:#7C3AED,color:#3B0764,stroke-width:2px
    classDef action fill:#FFEDD5,stroke:#EA580C,color:#431407,stroke-width:2px
    classDef observation fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px

    class U,F message
    class M decision
    class A action
    class O observation

    linkStyle 0 stroke:#64748B,stroke-width:1.5px
    linkStyle 1 stroke:#EA580C,stroke-width:2px
    linkStyle 2 stroke:#16A34A,stroke-width:2px
    linkStyle 3 stroke:#7C3AED,stroke-width:2px
    linkStyle 4 stroke:#2563EB,stroke-width:1.5px
```

This loop aligns with the thought-action-observation trajectories described in Yao et al.'s [ReAct paper](https://arxiv.org/abs/2210.03629): model inference updates the working decision state, actions interact with an environment, and observations ground the next iteration. Here, “reasoning” means model inference and decision state; it does not imply that private chain-of-thought is exposed, human-readable, or persisted.

### Expanded runtime and mechanism map

The overview above isolates the ReAct cycle. The expanded map adds product-surface routing, active-context assembly, durable state, and the mechanisms that inject, gate, execute, delegate, or validate work.

```mermaid
flowchart TB
    USER((👤 User))

    subgraph ROUTING["1 · Interaction routing"]
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

    subgraph CONTEXT["2 · Context and durable state"]
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
                TOOL_SCHEMAS["🛠️ Tool schemas /<br/>capability descriptions"]

                SKILL_CATALOG ~~~ TOOL_SCHEMAS
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
        CACHE_NOTE["Prompt cache<br/>cache hit lowers processing / price<br/>not context-window size"]
        CURRENT[(🗂️ Current model context)]
        SKILLS["📘 Skills / retrieval"]
        PERSIST[(💾 Persistence<br/>thread / session state)]
        GOAL([🎯 Optional Codex Goal<br/>thread-scoped contract])

        SKILLS -->|selected content| INPUTS
        PERSIST -->|load state| INPUTS
        ASSEMBLE -->|assembled context| CURRENT
        CURRENT -->|persist state| PERSIST
        GOAL -.->|optional completion contract| CURRENT
        CURRENT -.-> CACHE_NOTE
    end

    subgraph LOOP["3 · Agent loop and runtime controls"]
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

    subgraph DISCLOSURE["4 · Progressive skill disclosure"]
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
    CURRENT -->|rendered on every inference<br/>input tokens| MODEL
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
    class SYSTEM,DEVELOPER,CURRENT_REQUEST,EARLIER_MESSAGES,SCOPED,SKILL_BODY,SUPPORTING,SKILL_CATALOG,TOOL_SCHEMAS,HISTORY,RESOURCES,PRIOR_RESULTS,ASSEMBLE,CURRENT,UPDATE,D_META,D_CONTEXT,D_BODY,D_UPDATE,D_SUPPORT context
    class MODEL,D_SELECT,D_NEXT decision
    class ACTION action
    class OBS observation
    class GATE,HOOKS,VALIDATE control
    class PERSIST,GOAL persistence
    class SKILLS,DELEGATE delegation
    class ENV,INJECT_NOTE,CACHE_NOTE,COMPACT_NOTE note
```

The diagram uses the general term **agent loop** from OpenAI's [long-horizon Codex explanation](https://developers.openai.com/blog/run-long-horizon-tasks-with-codex). In provider terms, Codex runs a turn within a durable thread, while Anthropic's [Claude Code CLI reference](https://docs.anthropic.com/en/docs/claude-code/cli-usage) uses **agentic turns** within a session. OpenAI's [Goals guide](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex) keeps a Goal separate as an optional thread-scoped completion contract across turns. The realtime-voice branch records one observed Codex macOS architecture, not a universal voice design. Whether a frontend speech model can access backend instructions such as `AGENTS.md` or `CLAUDE.md` must be verified separately for each voice implementation; the diagram does not assume that access.

### Context-window and cost mechanics

Codex CLI enumerates applicable `AGENTS.md` files and injects each discovered chunk near the top of conversation history as a separate user-role message, before the user prompt, in root-to-leaf order. This describes discovery and injection, not a claim that the file is physically reread before every inference. Once retained in active history, the injected tokens can appear in every later rendered model input until context reconstruction, pruning, or compaction replaces them. See OpenAI's [Codex model guidance](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.3-codex#using-agentsmd).

Skills use progressive disclosure. The initial model context contains compact metadata: name and description, plus the path for local Responses API skills. The model selects a matching skill from that metadata and then reads its full `SKILL.md`; supporting references, scripts, templates, and assets remain additional on-demand material. See the official [Skills API guide](https://developers.openai.com/api/docs/guides/tools-skills) and [plugin Skills concepts](https://developers.openai.com/plugins/concepts/skills).

| Mechanic | Context-window effect | Processing and price effect |
| --- | --- | --- |
| Repeated inference | Every model call receives a rendered context. Retained content occupies input space on every call in which it remains active. | Input-token usage measures the rendered input, including repeated content. A turn, an individual tool-loop inference, and a stored transcript record are different units. |
| Prompt caching | Cached prefix tokens still occupy the context window and count as input tokens and toward rate limits. | A cache hit reuses prefix computation, reducing latency and the applicable input-token rate. For GPT-5.6+ API pricing, reads are currently `0.1×` and writes `1.25×`; this is API pricing, not a universal Codex-subscription accounting promise. |
| Compaction | Older active history is replaced by a smaller opaque compaction item plus retained items, reducing later rendered inputs. | The changed prefix can reduce prompt-cache reuse immediately after compaction. |

Without caching, a stable block of `L` tokens retained across `N` model calls contributes approximately `L × N` input tokens. With one full-price processing followed by cached reads at multiplier `r`, its simplified price-equivalent is approximately `L + (N − 1) × r × L`, while every call still occupies `L` context-window tokens. When the model uses a distinct cache-write rate `w`, replace the first `L` with `w × L`; for current GPT-5.6+ API mechanics, `w = 1.25` and `r = 0.1`. Product subscriptions and internal accounting can differ from API token pricing. See [Prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching) and [Compaction](https://developers.openai.com/api/docs/guides/compaction).

Never infer token cost, context-window occupancy, repeated model exposure, cache reuse, or instruction robustness from stored-record counts alone. Diagnose five separate layers:

| Layer | What it establishes |
| --- | --- |
| Stored rollout or transcript records | Append-only persistence and diagnostic evidence; one stored record can participate in many later model calls. |
| Active conversation/context items | The items retained after pruning, reconstruction, or compaction and available to build the next request. |
| Rendered input for one inference | The complete model input, including retained items and harness-provided surfaces such as tool schemas that may not recur as transcript messages. OpenAI's [function-calling token-usage guidance](https://developers.openai.com/api/docs/guides/function-calling#token-usage) states that callable definitions count against context and input tokens. |
| Usage accounting | Per-inference `input_tokens`, `cached_tokens`, output tokens, and reasoning tokens; these fields support conclusions about processed usage and aggregate cache reuse. |
| Product accounting | The applicable API price, ChatGPT/Codex subscription allowance, credit balance, or other product-specific accounting surface. |

A block can be stored once yet remain in many later rendered inputs, as observed for `AGENTS.md`. Conversely, repeated skill-catalog or runtime records can reflect reinjection or updates without proving full-price processing on every occurrence or stronger instruction following. Prompt caching can make repeated rendered tokens cheaper and faster without removing them from context-window occupancy; only per-inference usage/cache fields plus the applicable product accounting surface support a cost conclusion.

> **Local rollout observation — version/session-specific, not universal.** The analyzer classified one explicit `AGENTS.md` record, five additional skill-catalog/runtime blocks, and 290 token-count snapshots in this rollout. Record count and model-call snapshot count therefore clearly differ. The explicit `AGENTS.md` record was approximately 19,812 characters or 4,953 tokens using the existing four-characters-per-token estimator; the five additional blocks totaled approximately 112,454 characters or 28,113 estimated tokens. The latest measured inference reported 158,680 exact input tokens, including 158,208 cached tokens and 472 non-cached input tokens. These are aggregate observations and do not attribute cached tokens to individual blocks.

Design for progressive disclosure: keep compact, broadly applicable invariants in `AGENTS.md`; put concise trigger conditions in skill metadata; keep detailed procedures in the selected `SKILL.md`; and load deeper references or assets only when the task needs them.

The model cannot reason from information that never reaches its context, and it cannot perform an action for which the harness exposes no actuator.

## Mechanism Taxonomy

Different mechanisms control different parts of the runtime.

| Mechanism | What it changes | Typical trigger | Reliability profile |
| --- | --- | --- | --- |
| Always-loaded instructions | Baseline model guidance | Context/session construction | Probabilistic; competes with other context |
| Skills / reusable playbooks | Conditional workflow knowledge | Model or user selects skill | Probabilistic selection plus probabilistic execution |
| Retrieved references/resources | On-demand facts or procedures | Explicit retrieval or skill step | Available only after retrieval |
| Tool/MCP descriptions | Whether a capability is selected | Model tool selection | Probabilistic routing |
| Tool implementation / scripts | What happens after selection | Tool invocation | Deterministic within code/environment limits |
| Hooks / event handlers | Automatic checks or context injection | Runtime event | Deterministic if event is emitted and hook succeeds |
| Permissions / approvals | Allow, deny, or gate actions | Before protected action | Strong enforcement |
| Validators / tests / linters | Detect invalid outputs or state | Explicit/event-driven execution | Deterministic detection for encoded conditions |
| Scheduler / automation | Run work at a time or external event | Clock/cron/event | Does not depend on a fresh user prompt |
| Memory / persistent files | Preserve state across turns/sessions | Read/write policy | Persistence only; not automatic use unless loaded |
| Subagents / delegation | Isolate or parallelize task context | Harness/model delegation | Same sensor/actuator constraints apply per worker |
| UI / human confirmation | Obtain missing judgment or authorization | Interaction point | Human-controlled decision surface |

Stronger wording does not change the underlying mechanism. `Always`, `never`, `critical`, repetition, and all-caps remain prompt-level guidance unless a separate control surface enforces the behavior.

## Natural-Language Rules Are Still Conditional Programs

A Markdown instruction file can contain conditional rules even after the file itself has been loaded. Such a rule has two logical parts:

1. **Condition / trigger** — the observable situation in which the rule applies.
2. **Behavior / action** — what the agent should do once that condition is satisfied.

The condition can only affect runtime behavior when the model has evidence from which it can determine that the condition applies. If that evidence is absent, the rule may be present in context but remain inapplicable or impossible to evaluate.

This is distinct from skill-level progressive disclosure. Skill metadata can determine whether a skill body becomes available at all; once the body is loaded, individual rules inside it can still have their own conditions. Loading the body makes a rule available to the model, but does not mean its condition is satisfied. A rule such as "when you are tired, do X" therefore depends on whether the runtime supplies evidence from which "tired" can be inferred.

Because condition and consequence are separate logical roles, instruction text can represent them explicitly rather than embedding both in one prose sentence. For example, Markdown may use visibly separated **When** and **Then** blocks, a two-column condition/action table, or another structure that preserves the same distinction. This formatting does not create a new runtime mechanism; it makes the rule's applicability and consequence easier to distinguish within the context already loaded.

## Reliability Characteristics

Mechanisms provide different levels of enforcement:

1. Passive prose in a prompt or instruction file.
2. Conditionally retrieved skill/reference.
3. Explicit model classification with the required evidence in context.
4. Tool-assisted check using live external state.
5. Deterministic validator or script invoked by the workflow.
6. Event hook or permission gate that runs by construction.
7. External automation or system policy independent of model initiative.

The sequence generally moves from probabilistic guidance toward stronger enforcement, while implementation cost and platform coupling also tend to increase.

## Provider / Harness Reality

Agent harnesses do not have feature parity. The same conceptual mechanism can be exposed through different concrete surfaces, or may be absent on a given runtime.

### OpenAI / Codex

- Codex-style `AGENTS.md` files are scoped instructions assembled from the filesystem hierarchy and injected into model context.
- Agent Skills expose name/description metadata before selection; the full `SKILL.md` and supporting files are loaded after the skill is selected.
- Plugins can package skills, MCP configuration, and lifecycle hooks; exact availability still depends on the execution surface.
- OpenAI's Agents API uses a managed Codex harness that owns orchestration, context compaction, and durable sessions while the application supplies tools and execution environment.

### Claude Code

- Claude Code distinguishes `CLAUDE.md`/rules, skills, subagents, hooks, output styles, and system-prompt additions by when they enter context and how they persist.
- Hooks are event-driven commands and can run at lifecycle points such as session/prompt/tool events; their execution is tied to emitted runtime events rather than model initiative.
- Skills and subagents are not equivalent to hooks: they guide or delegate model work rather than deterministically firing on every matching runtime event.

### MCP

MCP is a protocol boundary, not an agent by itself. Servers can expose resources, prompts, and tools; clients decide how those capabilities are surfaced to the model and user. Tool selection is generally model-controlled, while the protocol does not mandate a single agent loop or user interface.

The 2026-07-28 MCP specification uses a stateless protocol core. Cross-call server state therefore requires an explicit state mechanism such as handles or application-managed persistence rather than implicit transport session state.

## Codex Runtime Mechanism Map

These are distinct control surfaces with different loading, triggering, and enforcement behavior.

### Hierarchical `AGENTS.md`

Codex loads global guidance from `$CODEX_HOME/AGENTS.md` (normally `~/.codex/AGENTS.md`). It then discovers project guidance from the identified project root down to the current working directory. The default project-root marker is `.git`; when no configured project marker is found, Codex checks only the current working directory for project guidance. An `AGENTS.md` in a non-Git parent is therefore not inherited merely because it is an ancestor.

The resulting instruction chunks are injected before the current user prompt in root-to-leaf order; deeper scopes can override earlier guidance. `AGENTS.override.md` can replace the normal file at a scope. Treat the configured `project_root_markers` list as live configuration rather than assuming `.git` when diagnosing a specific installation.

### Skills and progressive disclosure

Skill discovery is staged:

1. Codex initially sees skill metadata, especially `name` and `description`.
2. When a skill is selected, its `SKILL.md` instructions become available.
3. `references/`, `scripts/`, and `assets/` are consumed only when the workflow needs them.

The metadata catalog is therefore a pre-selection routing surface, while the body and supporting resources are post-selection execution surfaces. A behavior that must shape every response belongs in always-loaded guidance or a stronger lifecycle/enforcement mechanism, not only in a conditional skill.

### Hooks

Codex currently documents command hooks for lifecycle events including `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PreCompact`, `PostCompact`, `SubagentStart`, `SubagentStop`, `Stop`, `Interrupt`, and `SessionEnd`.

Hooks can inspect structured runtime input. Depending on the event, they can inject context, block a prompt or tool call, rewrite tool input, emit warnings, or run post-action checks. `PostToolUse` cannot undo an action that already happened.

Current Codex command hooks are not a generic arbitrary-agent callback system. OpenAI documents `prompt` and `agent` hook handlers as parsed but skipped, so only the supported command-hook events and effects are active runtime mechanisms.

### MCP and tools

An MCP server exposes live information and controlled actions; it does not itself guarantee that the model will choose a tool. Tool descriptions and schemas are model-visible selection surfaces, while server-side code determines what happens after invocation.

### Plugins

A plugin is a packaging and distribution boundary. OpenAI plugins can bundle skills, MCP configuration, assets, and Codex lifecycle hooks. Plugins do not by themselves make a capability deterministic: reliability still depends on the contained mechanism and the surface where it runs.

### Permissions and approvals

Permissions and approval policies are enforcement surfaces that can determine whether an action may proceed. Hooks are a separate event-driven surface that can perform checks before or after supported tool calls.

### Sessions, compaction, goals, and persistent state

Conversation state, global instructions, and memory are separate forms of state. In OpenAI's managed Codex harness, a session is a durable unit of work.

"Context usage" means the currently rendered model input, not the visible chat length or the entire persisted transcript. OpenAI's [compaction guide](https://developers.openai.com/api/docs/guides/compaction) documents that crossing a configured threshold can emit an encrypted, opaque compaction item, prune older active context, and carry forward key prior state and reasoning in fewer tokens. A compacted window can also retain selected earlier items. The full local rollout may remain persisted even though those records are no longer all rendered into the next inference.

**Version-specific local observation (Codex 0.149.0):** one inspected rollout stored a `type: compacted` record with `replacement_history`; the replacement retained explicit messages plus an encrypted compaction item, while reported input fell from roughly 223k to 51k tokens. These field names and counts describe that version and rollout, not a stable public schema.

To inspect another rollout safely, choose one explicit rollout JSONL file and return only the compacted-record projection instead of searching the whole rollout tree:

```bash
ROLLOUT_FILE=/absolute/path/to/one/rollout.jsonl
jq -c 'select(.payload.type == "compacted") | .payload | {type, replacement_history_count: (.replacement_history | length)}' "$ROLLOUT_FILE" | head -n 3

jq -c '
  select(
    .payload.type == "event_msg"
    and .payload.payload.type == "token_count"
  )
  | .payload.payload.info
  | {
      total_token_usage,
      last_token_usage,
      model_context_window
    }
' "$ROLLOUT_FILE"
```

The second projection reproduces the token snapshot without returning prompt or tool content. Rollout files can contain prompts, tool output, and other sensitive context. Keep inspection local, target a known file, and project only the fields needed for the diagnosis. Field names and record shapes may change between Codex versions.

Codex Goals are thread-scoped persisted state rather than global memory or project instructions. Thread/session state belongs to one ongoing trajectory, while files or another explicit store can persist state beyond that scope.

### Session and configuration freshness

A running conversation or agent session has its own assembled context and runtime state. Changes to external configuration surfaces such as instruction files, skills, MCP configuration, hooks, or generated installations do not imply that every already-running session has incorporated those changes. Freshness behavior is harness-specific.

Therefore, the existence of an updated file on disk and the presence of that update in a particular session's effective context are separate facts. A new session commonly rebuilds configuration/context from current sources; an existing session may require an explicit refresh, reload, re-read, session update, or restart depending on the harness. Do not assume automatic propagation without a documented mechanism or direct verification.

### Subagents

Subagents are separate model workers started by the harness for delegated work. They may have isolated task context and their own lifecycle events, but they remain bounded by what context, tools, permissions, and environment the harness gives them.

Delegation changes task decomposition and context isolation; it does not by itself create an enforcement mechanism.

## Context-Efficient Observation

Local computation is not itself model context. Only the command or tool result returned to the model consumes the active context window. Parse large artifacts locally and return a compact projection: bound matches, select relevant fields, cap lines, and summarize counts.

Avoid broad recursive `rg`, web search, resource-listing, or rollout-log queries when a narrower target will answer the question. In particular, self-referential searches over rollout history can reproduce earlier prompts and nested tool outputs, injecting thousands of duplicate tokens into the very context being diagnosed.

For a token or context diagnosis, use one explicit rollout and proceed in order:

1. Count stored records by category without dumping their full bodies.
2. Identify the active replacement state after any compaction.
3. Inspect the per-inference token snapshots rather than treating transcript records as model calls.
4. Use `cached_tokens` and related usage fields to separate rendered occupancy from uncached processing.
5. Consult the applicable ChatGPT/Codex allowance, credit, or API pricing view before concluding monetary or quota cost.
