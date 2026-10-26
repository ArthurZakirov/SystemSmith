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

### Do not address a frontend model through backend guidance

When a product uses a separate realtime/frontend model, write each instruction for the agent that can actually read and act on it. Do not put sentences in backend `AGENTS.md` that tell an unseen frontend model how to speak, pause, click, or pace an explanation. Instead:

- put frontend conversational behavior on a verified frontend prompt, personalization, or equivalent surface;
- put backend responsibilities in backend guidance—for example, resolve exact repository/worktree/file/line evidence, inspect UI state, or return one grounded next action;
- define the handoff contract between the two layers when both contribute to one user experience.

Use agent-relative wording such as `you`, `the user`, or `the agent` only when the reader is unambiguous from the artifact's runtime. Avoid architectural prose such as “the orchestrator should...” when the model reading the instruction cannot observe that role.

### Capability and attribution audit for cooperating agents

When developing separate prompts or instructions for a frontend conversational agent and an execution backend, audit the complete handoff—not just the text of each file:

1. **Name the receiving agent.** Identify which model reads each instruction, what it knows from its own context, what tools it can invoke, and which requests it can pass to another model. The two agents are not a single actor merely because the user experiences one conversation.
2. **Match verbs to actuators.** Instructions to the voice frontend may govern listening, speaking, turn-taking, disclosure, confirmation checkpoints, and backend requests. Instructions to open VS Code, run a CLI, inspect repository contents, or verify an editor window belong to an agent with those capabilities. If the frontend must influence such an action, tell it to **ask the backend** to perform or defer it.
3. **Split one requirement across the correct surfaces.** For paced code walkthroughs, the frontend should withhold future evidence from speech and ask the backend not to open future evidence locations. Backend guidance should handle accurate repository/worktree/file/line verification and editor navigation. Keep the interactive checkpoint protocol on the frontend, not duplicated as backend execution guidance.
4. **Attribute the executor in visible language.** A frontend that reads backend text must not imply it performed backend actions. Have the backend's user-visible result identify the actor when needed, and distinguish a request to open a file, an attempted editor operation, and verification that the intended location is visible. Do not rewrite the visible result into an unsupported first-person claim.
5. **Validate the handoff.** Review each imperative, capability assumption, and example in both instructions. Exercise a case requiring delegation, a case where editor control fails, and a multi-step walkthrough. Verify the frontend neither exposes future checkpoints nor claims it performed backend tools, and that the backend returns grounded, accurately attributed results.

For example, rather than telling a voice-only frontend “Do not open the next evidence file,” tell it “Ask the backend not to open the next evidence file until the user confirms readiness; do not tell the user about that file yet.” Rather than suggesting it say “I opened VS Code,” request backend-visible wording such as “The backend asked VS Code to open the verified file and line. Can you see it?” Use the stronger completion claim only when the backend actually verified the result.

### ChatGPT Live boundary

OpenAI currently documents ChatGPT Live as a GPT-Live voice experience that can use web search and memory and may coexist with text/images in the same chat. OpenAI also documents that preset ChatGPT personalities do not currently apply to Live. Custom Instructions are documented as applying across ChatGPT chats, but the public documentation inspected on 2026-10-01 does not establish the exact prompt assembly or whether those instructions are injected directly into the Live model versus applied elsewhere in the product pipeline.

Therefore, do not assume Codex's `experimental_realtime_ws_backend_prompt` architecture or its frontend/backend split maps directly onto ordinary ChatGPT Live. Verify the exact ChatGPT surface before choosing its control mechanism.

Evidence: [ChatGPT Voice](https://help.openai.com/en/articles/20001274-chatgpt-voice) and [ChatGPT Custom Instructions](https://help.openai.com/en/articles/8096356-chatgpt-custom-instructions).
