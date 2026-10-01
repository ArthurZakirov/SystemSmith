# 🎙️ Interaction runtime model

Use this reference when behavior depends on the product surface through which the human interacts with the agent: voice vs text, desktop vs web/mobile, Chat vs Work/Codex, task/goal UI, background execution, approvals, rendering, notifications, or interruption behavior.

This is distinct from the [`Agent runtime model`](agent-runtime-model.md). The agent runtime explains how context, models, tools, hooks, permissions, subagents, and persistence execute work. The interaction runtime explains how human input reaches that runtime and how work/results return to the human.

## 🧩 Interaction layers

1. **Input capture** — microphone/keyboard, voice activity detection, dictation/transcription, attachments, and whether capture continues while work is running.
2. **Turn/session semantics** — what creates a turn, task, thread, Goal, Work/Codex run, interruption, or continuation.
3. **Coordinator responsiveness** — whether the speaking/main session can accept and process new input while delegated/background work is active.
4. **Presentation channels** — spoken audio, live transcript, structured Markdown, widgets, files, notifications, and whether each surface renders the same output.
5. **Approval channel** — spoken approval, on-screen confirmation, permissions, or other human-gating mechanism.
6. **Background lifecycle** — what continues after a call ends, app focus changes, a task is delegated, context compacts, or the product restarts.
7. **Return path** — how worker completion, blockers, and unanswered decisions are surfaced to the coordinating conversation.

## 🔬 Evidence hierarchy

Product-surface behavior changes quickly. Prefer, in order:

1. current official product/runtime documentation;
2. direct reproducible observation on the exact client/version/mode;
3. source code for open-source harnesses;
4. community reports as hypotheses to reproduce.

Do not promote a user-observed quirk into a universal rule without reproduction or documentation. Conversely, do not overwrite a reproducible observed limitation merely because public documentation is silent.

## ⌨️ Codex text and realtime voice

**Observed runtime behavior, not a universal voice contract:** typed input in Codex starts an ordinary Codex turn. In the inspected macOS desktop realtime voice architecture, a frontend realtime speech model can answer directly and delegates selected work to a backend Codex runtime. Backend `AGENTS.md` and skills govern work that reaches that backend; they cannot govern every spoken response produced by the frontend model.

When diagnosing voice behavior, identify which model produced the response and whether delegation occurred before changing backend guidance. Do not infer this split for every client or voice implementation; verify it from current runtime metadata or a reproducible trace on the target surface.

### Realtime prompt control

In current Codex, realtime startup context deliberately excludes repo-memory instructions, `AGENTS.md`, project-doc prompt blends, and memory summaries. Those inputs belong to the normal backend prompt path after delegation. Therefore, adding a rule only to `AGENTS.md` cannot control a spoken response that the realtime conversational model produces before delegation.

Codex exposes the experimental top-level `experimental_realtime_ws_backend_prompt` configuration key as the prompt-level actuator for the realtime websocket conversation. It overrides the realtime conversational-layer instructions without changing normal backend prompts. Because it replaces the bundled prompt rather than appending to it, preserve the required upstream delegation/transcript protocol when supplying a custom value.

Use this distinction when engineering behavior:

| Desired behavior | Change surface |
| --- | --- |
| Must affect the realtime conversational model before it decides whether to delegate. | Realtime prompt/configuration surface such as `experimental_realtime_ws_backend_prompt`, then behaviorally test the exact product surface. |
| Applies only after the backend agent receives work. | Backend guidance such as `AGENTS.md`, skills, tools, hooks, or project context. |
| Must hold across both layers. | Encode each part on the layer that can observe and act on it; do not describe one layer as if it can read the other's private guidance. |

Primary evidence: [Codex config source](https://github.com/openai/codex/blob/main/codex-rs/config/src/config_toml.rs), [bundled realtime backend prompt](https://github.com/openai/codex/blob/main/codex-rs/prompts/templates/realtime/backend_prompt.md), and the source-backed reproduction in [openai/codex#37950](https://github.com/openai/codex/issues/37950). Treat the key as experimental and re-verify it before relying on it in a durable setup.
