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

## 🤝 Designing cooperation across agents and surfaces

Apply this design when a requirement actually spans multiple agents, interfaces, or execution boundaries. **Do not assume that every voice interface has a separate backend, that all agents share context, or that every interaction needs user checkpoints.** Establish the target runtime topology first.

1. **Identify participants and entry points.** For each relevant model, controller, tool, and human participant, determine the input/trigger, instructions it receives, evidence and state it can observe, permissions and actuators it controls, and output channels it can use.
2. **Define the requested outcome and interaction constraints.** Identify the user-visible success condition and whether this particular behavior requires delegation, confirmation, pacing, interruption, background continuation, or approval. Treat those as conditional requirements, not universal steps.
3. **Assign decisions and actions to capable participants.** Place a rule where its recipient can observe its trigger and perform its consequence. When it cannot, define a request to a participant with the required capability; do not assume shared memory, tool access, or authority.
4. **Design the handoff contract.** Specify the initiating condition, request payload and necessary context, response format/channel, progress and error states, approval owner, and how changed user instructions or interruptions propagate. Define who retains state and who resumes after a handoff.
5. **Choose the authoritative implementation surfaces.** Place conversational/interaction guidance on the participant that handles that interaction; execution and verification guidance on the participant that performs it. Put the cross-boundary contract where the relevant participants can use it without duplicating task-specific routines.
6. **Specify result attribution and disclosure.** Distinguish an action that was **requested**, **attempted**, **reported successful**, or **independently verified**. Attribute each claim to its actual actor and observation. Require staged disclosure or a visual checkpoint only if the particular workflow needs it.
7. **Validate boundaries and generality.** Exercise successful cooperation, missing access or failed delegation, and a change or interruption when applicable. Test an unrelated task as a negative control for workflow-specific assumptions. Verify real behavior on the target runtime rather than merely checking prompt wording.

The reusable contract is about **who can observe, decide, act, hand off, and verify**. It does not mandate any particular application, editor, repository structure, or conversation sequence.

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

### Apply the capability boundaries to Codex

When the **verified Codex runtime** has a separate realtime conversational model and execution backend, their instructions and capabilities must be treated separately:

- Use the verified frontend prompt/configuration surface for interaction behavior that must occur before delegation, such as deciding when to request backend work or honoring a task's conversation constraints.
- Use backend `AGENTS.md`, skills, tools, or project guidance for actions and checks the backend can perform. The conversational model should request those actions through the supported handoff instead of claiming that it executed them.
- Define the request, return channel, error handling, and attribution appropriate to the requested behavior. If the user-facing layer speaks backend output, ensure that output itself makes the actor and evidence status clear.
- Verify the actual frontend/backend split on the target product and version. Neither all-voice delegation nor any specific step-by-step presentation policy follows automatically from this architecture.

### Example only: paced code-evidence walkthrough

**Scenario-specific requirement:** The user wants a voice-guided review of code evidence, with one visible location at a time. The backend can inspect a repository or worktree and may have access to VS Code; the conversational frontend can coordinate the discussion but cannot itself inspect files or control the editor.

- **Frontend's role in this example:** Request only the next evidence location, wait for the user's confirmation when the requested walkthrough requires a checkpoint, and avoid disclosing future locations early.
- **Backend's role in this example:** Verify the repository/worktree, file, and line; ask VS Code to open the location if available; check the command and UI outcome as far as its tools permit; return grounded evidence and an accurately attributed status.
- **Example handoff:** Instead of instructing the frontend to "open the next file," instruct it to request that operation from the backend. The returned text can say "The backend asked VS Code to open the verified file and line; can you see it?" Only claim the location is visibly open when that was actually verified.
- **Failure case:** If local editor access fails, have the backend return that limitation and a manual target, rather than implying a successful editor action.

**Scope:** VS Code, worktrees, code review, withholding future evidence, and one-at-a-time confirmation are properties of *this example's request*. They are **not** required steps for engineering other agent workflows. For a calendar lookup, for instance, a conversational participant could request calendar data from a permitted backend and present the result with correct attribution—without an editor, a worktree, or visual checkpoints.

### ChatGPT Live boundary

OpenAI currently documents ChatGPT Live as a GPT-Live voice experience that can use web search and memory and may coexist with text/images in the same chat. OpenAI also documents that preset ChatGPT personalities do not currently apply to Live. Custom Instructions are documented as applying across ChatGPT chats, but the public documentation inspected on 2026-10-01 does not establish the exact prompt assembly or whether those instructions are injected directly into the Live model versus applied elsewhere in the product pipeline.

Therefore, do not assume Codex's `experimental_realtime_ws_backend_prompt` architecture or its frontend/backend split maps directly onto ordinary ChatGPT Live. Verify the exact ChatGPT surface before choosing its control mechanism.

Evidence: [ChatGPT Voice](https://help.openai.com/en/articles/20001274-chatgpt-voice) and [ChatGPT Custom Instructions](https://help.openai.com/en/articles/8096356-chatgpt-custom-instructions).
