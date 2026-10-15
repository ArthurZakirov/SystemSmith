---
name: software-design
description: "Design and review code, tests, scripts, configuration, and reusable agent artifacts around clear responsibility and change boundaries, explicit dependencies, truthful contracts, and justified complexity. Use when implementing or refactoring functions, defaults, fallbacks, validators, domain values, entry points, compatibility paths, or skills and documentation that combine reusable rules with changing data."
---

# Software Design

## Core Principle

Build the simplest design that faithfully represents the problem's real contracts, responsibilities, and sources of variation. Optimize for a coherent, maintainable system rather than the fewest lines changed or the smallest immediate edit. An abstraction is valuable when it captures a genuine boundary; introducing layers by habit is not an improvement.

## Responsibility and Change Boundaries

- **One reason to change per responsibility.** Group behavior by what it does and why it changes. Separate concerns that evolve independently, even if combining them is initially convenient.
- **Reusable definitions versus instance data.** Keep stable algorithms, policies, schemas, and instructions separate from invocation-specific values, current state, temporary observations, and environment bindings. Reusable artifacts should not silently capture a snapshot of one use.
- **Entry points versus core behavior.** CLI commands, notebooks, scheduled jobs, web handlers, and event handlers adapt their environment to reusable operations. Parsing, startup, and I/O orchestration should not be entangled with the processing logic they invoke.
- **Information lives at its owning boundary.** A value, decision, or rule belongs where its meaning is defined and its lifecycle is managed. Avoid duplicate definitions and hidden assumptions distributed across consumers.
- **Dependencies are legible.** Callers and maintainers should be able to identify what a component needs, which inputs vary its behavior, and who decides their values. Prefer explicit inputs at meaningful boundaries over implicit state or unrelated hardcoded choices.

## Honest Contracts and Failure Behavior

- Required inputs are required. Missing configuration, invalid domain values, and violated invariants produce clear, actionable failures rather than silently becoming convenient alternatives.
- A default is a **declared part of the contract**, not a means of concealing an undecided value. Optionality, coercion, retry, recovery, and fallback behavior require a real semantic or operational reason.
- Model the actual allowed states. Do not make invalid input seem valid by inventing substitutions, accepting absent fields as meaningful values, or treating impossible states as routine success paths.
- Compatibility supports **real consumers and deployed contracts**, not every intermediate design a feature branch once contained. Before release, replace superseded names, paths, schemas, and behaviors consistently rather than preserving speculative legacy routes. Preserve compatibility when an actual published interface, persisted data, or external consumer requires it.
- Errors remain observable at the layer responsible for addressing them. Recovery is appropriate when the system has a well-defined recovery policy, not merely because execution might fail.

## Explicit Choices Without Mechanical Patterns

- **Parameters and injection:** Caller-controlled inputs or dependencies belong at an explicit interface when they vary between invocations. Prefer required parameters when callers must decide; use defaults only for meaningful, stable choices. Do not pass every invariant through a chain of parameters or create dependency-injection machinery without a real seam.
- **Named values and domain modeling:** Give meaningful, recurring, or policy-bearing values a discoverable home. Constants name fixed values; `Literal` types constrain an allowed set; enums model related named domain choices. Choose the representation that matches the concept rather than replacing every literal indiscriminately.
- **Abstractions:** Extract a module, function, type, configuration layer, or shared rule when it makes ownership, reuse, change, or testing clearer. Avoid one-off indirection, unnecessary configurability, and abstraction for imagined future extensions.
- **Change scope:** Update affected callers, contracts, and tests when a real interface changes. Do not preserve a weak interface merely to avoid touching consumers, but do not refactor unrelated working code just for architectural tidiness.

## Same Principle Across Different Surfaces

| Context | Architectural distinction |
| --- | --- |
| A test helper constructs a simulated event payload | The helper owns construction rules; the caller owns case-specific field values. A fixed value inside the helper should not silently prevent tests from expressing another valid case. |
| An application reads a required environment variable | Configuration is supplied by the deployment context; the application should not invent an environment-specific value when the variable is absent. |
| A validator accepts only two domain states | The type or schema expresses the permitted states; an unrecognized or missing required state is not silently reinterpreted as one of them. |
| A feature changes a key, path, or schema before publication | The new contract replaces the obsolete branch-local one. Compatibility is added only when there is something real to remain compatible with. |
| A reusable shopping skill is given today's pantry inventory | The skill holds durable shopping rules; dated inventory is runtime data, not part of its reusable definition. |
| The same operation is launched by a CLI, notebook, or cloud event | Each entry point translates its inputs and invokes reusable behavior without embedding that behavior into its startup mechanics. |

These examples illustrate change and ownership boundaries, not mandatory recipes for specific technologies.

## Design Standard

A well-structured implementation makes it straightforward to answer: **What is invariant? What changes, who chooses it, and where does that choice enter? Which values are valid? Who owns errors and recovery? Which consumers actually depend on the old contract?**

Prefer a small number of clear, justified boundaries over both hidden coupling and speculative architecture. Where tradeoffs exist, make the chosen semantics explicit rather than hiding them in fallback logic, defaults, or incidental implementation details.
