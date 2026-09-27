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
