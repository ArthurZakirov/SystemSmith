---
name: thoughts-to-artifact
description: Transform a user's raw thoughts, dictation, transcript, notes, brainstorm, or conversational explanation into a structured communication artifact while preserving meaning, examples, nuance, and recoverable detail. Use for articles, documentation, skills, READMEs, messages, briefs, posts, or similar outputs when the source structure is incidental and the output should organize ideas by meaning rather than original speaking order.
---

# Thoughts to Artifact

## Core Principle

Treat the source as an information set, not as an outline that must be copied in order.

Reorganize ideas around the structure that best serves the target artifact while preserving the information needed to reconstruct the user's intended meaning.

## Preserve Information, Not Source Order

- Group statements about the same concept even when they appeared far apart in the source.
- Reorder material when conceptual grouping, causal order, or target-audience flow is clearer than dictation order.
- Remove transcript noise, false starts, and redundant wording only when no distinct information is lost.
- Do not invent missing claims or silently resolve genuine ambiguity.

## Preserve Concrete Evidence

Do not discard examples merely because a generalized rule can be extracted from them.

A generalization is a lossy encoding of concrete cases: the rule often cannot reconstruct the original examples, while the examples still contain evidence for deriving or challenging the rule. Preserve examples that add boundary conditions, causal detail, intent, counterexamples, or diagnostic value.
## Transformation Procedure

1. Identify the target artifact and its audience/purpose.
2. Extract distinct claims, examples, constraints, causal explanations, questions, and decisions from the source.
3. Group semantically related material regardless of where it appeared in the source.
4. Separate general rules from the concrete examples that support or qualify them.
5. Choose the appropriate representation with `information-representation-design`.
6. Apply `relevance-first-information-design` to decide what belongs at the current layer versus a deeper linked layer.
7. Preserve unresolved ambiguity explicitly instead of filling it in.
8. Verify that important source information was not lost during compression or abstraction.

## Loss Check

Before finishing, ask:

- Which concrete examples disappeared from the output, and did they carry unique information?
- Which nuance, exception, uncertainty, motivation, or causal explanation was compressed away?
- Could the user recover the important original intent from the transformed artifact?
- Did source chronology survive only where chronology itself matters?

Use `format-markdown` additionally when the requested artifact is Markdown and Markdown-specific syntax or linking rules matter.
