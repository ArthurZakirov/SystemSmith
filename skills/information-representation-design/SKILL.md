---
name: information-representation-design
description: "Choose or review the representation form for information that will already be shown: table, list, hierarchy, prose, diagram, code abstraction, chart, or another structure. Use when the same content could be organized in multiple ways and readability, comparison, scanning, or structural clarity depends on the representation."
---

# Information Representation Design

## Core Principle

Choose the representation that makes the important relationships easiest to perceive with the least repeated scaffolding.

This skill does **not** decide whether information belongs on the current surface. That belongs to `relevance-first-information-design`. It decides **how information that is already in scope should be represented**.

## Representation Mapping

| Information shape | Preferred representation |
| --- | --- |
| Repeated items with the same fields or dimensions | Table |
| Ordered or procedural sequence | Numbered list |
| Unordered independent points | Bulleted list |
| Parent/child concepts at different abstraction levels | Hierarchy, headings, or nested navigation |
| Dependent conditions where a child rule applies only inside a parent condition | Nested decision hierarchy; use indented rules by default, pseudocode when code-like logic materially improves precision, and diagrams as a supplementary view when visual branching helps |
| Narrative reasoning where sequence and causal flow matter | Prose |
| Relationships, flow, topology, or spatial structure | Diagram when it materially improves comprehension |
| Quantitative comparison or trend | Appropriate chart or table |
| One compact condition and one action | Explicit `When` / `Then` labels may be clearer than a table |
## Structural Rules

- When several records repeat the same labels, encode those labels once as table columns instead of repeating headings or `When`/`Then` blocks.
- Do not flatten a decision tree into peer table rows when child conditions are meaningful only under a parent condition. Preserve the dependency explicitly with nesting unless each row intentionally restates the complete condition.
- Do not use a table for a simple linear flow solely because multiple steps exist.
- Group related material by concept rather than preserving an arbitrary source order.
- Keep sibling sections at comparable abstraction levels.
- Use headings to expose real hierarchy, not merely to add visual decoration.
- Prefer a representation that supports scanning when the reader is likely to compare or route rather than read linearly.
- Avoid duplicating content just to fit a representation; use references or links when one canonical source should serve several views.

## Cross-Medium Interpretation

The representation primitive depends on the medium. A table in Markdown may correspond to columns/cards in a UI, grouped fields in a form, a matrix in a dashboard, a typed record collection in code, or another structure that exposes the same repeated dimensions.

## Self-Check

1. What relationship should become obvious: order, comparison, hierarchy, causality, topology, or grouping?
2. Does the current representation repeat labels or structure unnecessarily?
3. Would another representation let the consumer perceive the relationship faster without losing information?
4. Are sibling elements represented consistently?
5. Is the representation still appropriate for the medium and expected interaction?
