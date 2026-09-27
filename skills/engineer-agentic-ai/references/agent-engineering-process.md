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

## Behavior Validation and Iteration

Static validation proves that an implementation is well-formed; it does not prove that the agent behaves as intended. Every implemented change to skills, instructions, hooks, tools, routing, automations, prompt rules, persistence, or another agent-system mechanism must pass a behavioral loop before it is considered validated or ready for promotion.

1. **State the outcome.** Begin with the problem-space need and express the desired agent behavior as an observable result.
2. **Operationalize it.** Map the outcome to observable signals, trigger, decision rule, actuator, persistence, reliability requirement, and implementation surface. Record what evidence would prove success or failure.
3. **Implement and validate statically.** Change the canonical surface and run its syntax, schema, link, test, or deployment checks. Confirm that the intended artifact reached the target runtime.
4. **Exercise the real behavior.** Send a representative prompt or event through an actual agent on the target surface so the intended situation, routing decision, and action must occur.
5. **Observe the outcome.** Check whether the requested real-world behavior happened. File existence, valid Markdown, a successful installation, or a plausible model response is insufficient when stronger runtime evidence is available.
6. **Diagnose the complete path on failure.** Determine whether the guidance was present and current, the trigger matched, required evidence was visible, the rule was specified correctly, the relevant skill/tool/hook was available and selected, an actuator and permission existed, and persistence, propagation, or session freshness behaved as assumed.
7. **Adjust and repeat.** Correct the mechanism or the test, then prompt or trigger the agent again. Iterate until the observable outcome is reached at the required reliability level.
8. **Test variation.** Include positive, negative, and boundary cases. Repeat routing-dependent or model-mediated cases enough to distinguish a stable mechanism from stochastic selection.
9. **Promote only behavioral evidence.** Treat the change as validated or promotion-ready only after the real interaction succeeds. Record remaining unknowns and do not silently broaden the claim beyond the tested cases.

The test should exercise the strongest relevant part of the mechanism. A deterministic hook requires evidence that the hook fired and enforced its result; a skill requires evidence that routing and body execution occurred; an automation requires evidence that its event and action path ran; a persistence change requires evidence across the intended boundary.

### When the runtime surface is the subject

Use a narrower controlled runtime experiment when the question itself concerns surface differences, session lifecycle, hot reload, caching, or propagation. Isolate the test in temporary project-local instructions or skills, establish a baseline, change one variable, repeat the treatment, and use a negative control. Inspect targeted instruction/catalog records, file reads, lifecycle boundaries, and usage/cache fields rather than relying on answer text or loading an entire rollout log.

For Codex CLI, distinguish one persistent thread resumed through separate `codex exec resume` processes from a continuously open TUI. For Codex desktop, keep the same chat and app session open when that lifecycle is under test; a newly created chat is a separate control. Record surface, product/app version, model/configuration, workspace scope, and every process, resume, restart, reload, or new-chat boundary.

Runtime findings apply only to the tested **surface + version + configuration + workspace scope + session lifecycle**. Do not equate CLI resume, an open TUI, an open desktop chat, a new chat, or a voice frontend. See the [`Agent Runtime Model`](agent-runtime-model.md) for loading and lifecycle mechanics and the [`Token Usage and Cost Model`](token-usage-and-cost-model.md) for rollout-log, usage, and prompt-cache interpretation.

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
