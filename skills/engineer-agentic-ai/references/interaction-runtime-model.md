# 🎙️ Interaction runtime model

Use this reference when behavior depends on the product surface through which the human interacts with the agent: voice vs text, desktop vs web/mobile, Chat vs Work/Codex, task/goal UI, background execution, approvals, rendering, notifications, or interruption behavior.

This is distinct from the **agent runtime model**. The agent runtime explains how context, models, tools, hooks, permissions, subagents, and persistence execute work. The interaction runtime explains how human input reaches that runtime and how work/results return to the human.

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

## 🗣️ Codex text and realtime voice routing

In ordinary Codex text interaction, a submitted user message starts a Codex model turn using the runtime context assembled for that turn.

In the Codex desktop realtime voice architecture observed on 2026-09-27, the interaction path had two model layers:

1. A frontend realtime speech model handled the live conversation.
2. It delegated selected work to a backend Codex agent.
3. The backend result returned through the coordinating voice session.

Simple spoken exchanges could remain at the frontend without starting a backend Codex turn. Backend-only `AGENTS.md` guidance and skills therefore could not reliably govern every spoken response on that surface. This boundary came from current runtime metadata and direct observation; it is not a universal claim about every OpenAI voice product or future desktop version.

When voice behavior must be reliable, first identify which layer produces the response. Put conversational behavior needed on every spoken turn on a frontend-visible surface. Put repository work and tool procedures in the backend Codex runtime. If the product exposes no frontend control surface, record that as a mechanism gap rather than strengthening backend-only prose.
