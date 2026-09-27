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
        INPUTS["📚 Context inputs<br/>System / developer / user messages<br/>History or compacted replacement state<br/>Scoped instructions · selected skill metadata/body<br/>Tool schemas · retrieved resources · prior tool results"]
        CURRENT[(🗂️ Current model context)]
        SKILLS["📘 Skills / retrieval"]
        PERSIST[(💾 Persistence<br/>thread / session state)]
        GOAL([🎯 Optional Codex Goal<br/>thread-scoped contract])

        SKILLS -->|selected content| INPUTS
        PERSIST -->|load state| INPUTS
        INPUTS -->|assemble| CURRENT
        CURRENT -->|persist state| PERSIST
        GOAL -.->|optional completion contract| CURRENT
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

    USER -->|types| TYPED
    USER -->|speaks| VOICE
    SPOKEN -->|played to| USER

    BACKEND -->|starts with| CURRENT
    CURRENT -->|rendered input| MODEL
    RESULT -->|delivered to| USER

    HOOKS -.->|inject / trigger| UPDATE

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

    class USER actor
    class TYPED,VOICE,SPOKEN,RESULT message
    class SPEECH,BACKEND surface
    class INPUTS,CURRENT,UPDATE context
    class MODEL decision
    class ACTION action
    class OBS observation
    class GATE,HOOKS,VALIDATE control
    class PERSIST,GOAL persistence
    class SKILLS,DELEGATE delegation
    class ENV note
```

The diagram uses the general term **agent loop** from OpenAI's [long-horizon Codex explanation](https://developers.openai.com/blog/run-long-horizon-tasks-with-codex). In provider terms, Codex runs a turn within a durable thread, while Anthropic's [Claude Code CLI reference](https://docs.anthropic.com/en/docs/claude-code/cli-usage) uses **agentic turns** within a session. OpenAI's [Goals guide](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex) keeps a Goal separate as an optional thread-scoped completion contract across turns. The realtime-voice branch records one observed Codex macOS architecture, not a universal voice design.

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
