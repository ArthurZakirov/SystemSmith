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
