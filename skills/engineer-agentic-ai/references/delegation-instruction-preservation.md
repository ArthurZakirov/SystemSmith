# Delegation and instruction preservation

## Failure mode

A delegator can know an applicable rule and still replace it with an invented generic default in a handoff prompt. For example, a repository may require commits and pushes by default after validation, yet the delegator writes “do not commit/push unless authorized.” The receiving agent then executes a contradictory task. Handoff summaries are not a substitute for authoritative instructions.

## Before dispatch

1. Enumerate applicable sources: current user request and preceding relevant conversation, instruction hierarchy, workspace/repository `AGENTS.md` and nested guidance, selected skills, config, and any repository-specific validation/commit policy.
2. Determine whether the receiver can access those sources in its target workspace and runtime. If yes, give concrete paths and require inspection; if no, transmit essential instructions and identify the authoritative origin. Never assume context survives a tool, session, harness, device, or chat boundary.
3. Compose the delegated task separately from its constraints. Preserve scope, user intent, expected outputs, and workflow obligations; do not add conservative boilerplate that contradicts known rules.
4. Audit the draft: Does it forbid something the applicable workflow requires? Does it authorize something disallowed? Did it omit essential constraints or inaccessible source references? Resolve conflicts against the instruction hierarchy, not personal defaults.
5. After dispatch, verify the receiving agent actually loaded the relevant guidance when this can be observed. When not observable, report the limitation and use outcome checks. Do not claim preservation was proven merely because the prompt contained instructions.

## Validation

Use paired positive/negative cases: (a) a repository requiring commit/push after green checks and (b) a repository or user explicitly prohibiting push. The delegated prompt must preserve the difference. Include a no-source-access case and a higher-priority conflicting-rule case. For model-mediated handoffs, exercise the actual target harness and observe the final Git state, not just the prompt text.

Static tests can validate a controlled prompt template, but cannot guarantee consistency of arbitrary free-form handoffs. Stronger enforcement requires all handoffs to pass an audited builder or runtime gate; do not claim such a gate exists unless verified.
