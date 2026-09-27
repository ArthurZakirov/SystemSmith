# Agent Engineering Process

Last checked: 2026-09-26

Use this reference when translating a desired behavior into a concrete modification of an agentic system. For runtime facts and available mechanism types, read `agent-runtime-model.md`.

## Operationalization Test

For every desired behavior, establish an explicit chain:

1. **Desired outcome** — what should be true in the user's world?
2. **Observable signal** — what evidence indicates the relevant situation exists?
3. **Sensor / data source** — how does that evidence enter the system?
4. **Trigger** — what event causes evaluation?
5. **Decision rule** — deterministic condition, model classification, heuristic, or human judgment?
6. **Actuator** — what can actually change the outcome?
7. **Persistence** — what state must survive across turns or sessions?
8. **Verification** — how will we know the mechanism fired and changed behavior as intended?

If any required link is missing, classify the behavior as **not operationalized**. Preserve it as problem-space intent rather than copying it into a production instruction and calling it implemented.

## Translation Procedure

When the user states a high-level wish:

1. Preserve the wish as **problem-space intent**.
2. Identify what must be observable for the system to know that the rule applies.
3. Verify that the target harness can receive those observations.
4. Identify the lifecycle event at which the decision must occur.
5. Choose the decision mechanism: deterministic code for crisp conditions, model reasoning for semantic judgment, and human input where value or authority is irreducibly human.
6. Identify the required action and verify that an actuator exists.
7. Select the narrowest runtime surface that connects trigger, evidence, decision, and action.
8. Define how the behavior will be tested.
9. Only then edit the production instruction, skill, hook, tool, permission, script, or automation.

## Runtime Behavior Experiments

Use a controlled experiment when documentation does not establish whether an instruction, skill, configuration, or other runtime surface is refreshed during a particular session lifecycle. Read the [`Agent Runtime Model`](agent-runtime-model.md) for loading and lifecycle mechanics and the [`Token Usage and Cost Model`](token-usage-and-cost-model.md) for rollout-log, usage, and prompt-cache interpretation.

### Experimental record

Before testing, record the dimensions that define the claim:

| Dimension | Record |
| --- | --- |
| Hypothesis | One observable claim, such as “a changed project skill description is visible on the next turn of this same open chat.” |
| Surface | CLI, continuously open TUI, desktop chat, new desktop chat, web, voice frontend, or another exact surface. |
| Runtime | Product mode, app/CLI version, model, relevant feature/config state, and working directory. |
| Lifecycle | Chat/thread identifier where safe, whether the process or app stayed open, and every resume, restart, reload, or new-chat boundary. |
| Mutation | The one file or field changed between baseline and treatment. |
| Evidence | Model response, injected instruction/catalog content, file-read output, targeted session/rollout projection, and usage/cache fields that can confirm or falsify the claim. |

The hypothesis and conclusion must use the same lifecycle vocabulary. “Same thread resumed by a new CLI process” is not equivalent to “same continuously open process.”

### Controlled procedure

1. Create an isolated temporary workspace. Use a project-local `AGENTS.md` and project-local test skill with unmistakable, non-sensitive `V1` markers. Do not alter real global instructions, generated installations, or production skills.
2. Capture the baseline before changing anything. Confirm both the expected positive behavior and a negative condition that should not trigger the test skill.
3. Change exactly one independent variable, such as the `AGENTS.md` marker, skill-description trigger, or skill-body marker from `V1` to `V2`.
4. Run the equivalent prompt in the exact lifecycle under test. Keep model, working directory, permissions, and other relevant settings stable.
5. Repeat the critical treatment unchanged. Skill selection is stochastic; one selection or miss does not by itself demonstrate hot reload or frozen state.
6. Run a negative control where practical, such as a deliberately nonmatching skill description or a prompt that must not require a file read.
7. Inspect runtime evidence instead of accepting the answer text alone. Project only the relevant records from one known rollout/session log: current instruction or catalog markers, targeted file-read calls and outputs, lifecycle boundaries, and per-inference usage/cache fields. Never load an entire large JSONL transcript into model context merely to find a marker.
8. Report four distinct layers: documented provider facts, direct experimental observations, the narrow conclusion supported by those observations, and remaining unknowns.
9. Remove the temporary workspace and test chats/threads with exact targets. Prefer recoverable cleanup where practical; verify that production and global files remained unchanged.

### Surface-specific variants

**Codex CLI resume:** start with `codex exec --json` in the temporary workspace, retain the returned thread identifier, and issue later treatments through `codex exec resume --json <thread-id>`. Record that each resume may be a new operating-system process even though the persisted thread is the same. This variant tests the resume boundary; it does not test a continuously open TUI.

**Continuously open Codex desktop chat:** create the chat from the temporary project and keep both that chat and the desktop app session open across baseline, mutation, treatment, and repeat. Send every probe back into that exact chat without navigating to a new chat or restarting the app. When computer-use tooling is required to preserve that surface, record it as part of the experimental setup and retain evidence of the selected chat. A newly created chat is a separate variant and useful control, not a substitute for the open-chat treatment.

### Generalization rule

A result applies only to the tested **surface + version + model/configuration + workspace scope + session lifecycle**. Do not silently generalize among CLI resume, a continuously open CLI TUI, an open desktop chat, a new chat, or a voice frontend. Differences across those variants are valid findings rather than contradictions. Mark any untested transition—such as editing an `AGENTS.md` that already existed when a desktop chat was created—as unknown until that exact transition is tested.

## Natural-Language Rule Design

Even after a skill or instruction file has loaded, each situational rule inside it should make two parts explicit:

- **Condition / trigger:** the observable situation in which the rule applies.
- **Behavior / action:** what the agent should do when that condition is satisfied.

A loaded Markdown body can contain many rules that are irrelevant to the current situation. Loading the file only makes those rules available to the model; it does not make every rule applicable. Therefore, do not bury applicability and behavior together in vague prose.

If the condition depends on an abstraction such as "when the user is tired" or "when this work is low value," operationalize how that state is inferred from observable evidence before treating the rule as reliable.

## Mechanism Selection

Use the lightest mechanism that can satisfy the required reliability. Escalate from passive prose toward retrieved guidance, tool-assisted checks, deterministic validators, event hooks, permission gates, or external automation as needed. Stronger wording is not stronger enforcement.

For platform-specific choices, verify the actual capabilities of the target harness rather than assuming feature parity.

## Example: Protect Personal Time

Desired behavior: "Do not let work expand into personal time for an unvalidated deadline."

Before implementation, determine:

- how current local time is obtained;
- where work-hour boundaries come from;
- how the current task and deadline are identified;
- how stakeholder validation is observed;
- which lifecycle event should trigger the check; and
- whether the runtime can warn, request confirmation, or block an action.

The resulting implementation may require multiple mechanisms rather than one prompt.

## Codex Translation Checklist

When Codex is the target harness, answer these questions before labeling a behavior implemented:

1. What observable evidence proves or suggests that the situation exists?
2. Which Codex-visible source can provide that evidence?
3. At what Codex lifecycle point must the check occur?
4. Is the decision deterministic, semantic, or human-valued?
5. Which actuator or gate can produce the required consequence?
6. What state must persist, and at what scope?
7. What is the smallest Codex mechanism that closes the complete loop?

## Debugging Unexpected Agent Behavior

When the user asks why the agent behaved a certain way, or reports that intended behavior did not occur, treat the incident as a runtime-debugging problem rather than a politeness problem.

1. Reconstruct the action and the evidence/context that was available when it happened.
2. Check whether the relevant instruction or mechanism existed at that time and was present in the effective session context.
3. Check whether the rule's condition actually matched the observable evidence.
4. Check whether the intended behavior/action was specified correctly.
5. Check routing: was the relevant skill/reference/tool discovered, selected, and loaded?
6. Check actuation: did the runtime have the capability and permission to perform the consequence?
7. Check propagation/freshness: was the source updated but the current or another active session still using earlier context/configuration?
8. Distinguish an implementation defect from a justified prior decision. If the prior decision had a sound reason, explain it rather than retroactively agreeing that it was wrong.
9. Only after the cause is identified should the implementation be changed.

For configuration changes, verify the actual propagation path. Refresh or notify already-running sessions only when the target harness exposes a documented mechanism for doing so. Otherwise, explicitly re-read in the current session where possible and use a fresh session as the robust fallback.

## Wishlist-to-Production Lifecycle

Do not treat every newly stated desired behavior as production guidance.

1. **Capture** the desired outcome in the canonical wishlist/backlog when the mechanism is not yet known or validated.
2. **Investigate** observability, available data sources, triggers, actuators, persistence, existing solutions, reliability needs, and platform constraints.
3. **Design** the concrete mechanism and verification plan.
4. **Implement and validate** the mechanism in the narrowest appropriate runtime surface.
5. **Obtain human review** when the system's production behavior or global guidance changes.
6. **Promote** only the validated mechanism/rule into production guidance or controls; do not copy the original problem-space prose into production as if it were executable.
7. **Retire or mark** the wishlist entry after promotion so it does not become a second active source of truth.

A wishlist is therefore a queue of desired system outcomes, not an instruction file. Its contents may inform future engineering work, but they must not be loaded as active runtime guidance merely because they are important.
