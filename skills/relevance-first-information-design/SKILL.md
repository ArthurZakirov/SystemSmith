---
name: relevance-first-information-design
description: Design or review any human- or agent-facing information surface so the consumer sees the minimum sufficient information at the right abstraction level for the current task, with clear routes to deeper detail only when relevant. Use for chat, docs, code, APIs, dashboards, charts, websites, profiles, messages, UIs, agent instructions, or other information-bearing interfaces.
---

# Relevance-First Information Design

## Core Principle

Decide **which information belongs at the current layer** and which information should remain available behind a clear drill-down path.

The consumer should encounter the minimum sufficient information needed for the current action, decision, understanding, or navigation step. Additional detail should remain accessible without competing for attention by default.

This skill is about **information selection and disclosure**, not about choosing whether visible information should be represented as a table, list, hierarchy, diagram, or prose. Use `information-representation-design` for that.

## Requirements

- Reduce unnecessary cognitive load.
- Do not hide information that may be necessary to continue correctly.
- Separate routing from detail.
- Keep each layer internally coherent in abstraction level.
## Layers

- **Routing layer:** Is this relevant now? Where should the consumer go next?
- **Working layer:** What is needed to act or understand at this level?
- **Deep layer:** What evidence, implementation detail, history, or explanation is needed only after drill-down?

## Cross-Medium Patterns

- Chat: concise answer first; optional depth behind an artifact, link, or explicit drill-down.
- Messaging: short top-level message; optional detail in a thread, call, attachment, or linked document.
- README/docs: route distinct audiences to the relevant deeper pages instead of mixing all procedures together.
- Website/app: overview first; reveal advanced controls or explanations on demand.
- Dashboard/chart: show the primary signal by default; expose secondary dimensions through hover, drill-down, filters, or detail views.
- Code: keep parent functions/modules at a coherent abstraction level and hide lower-level mechanics behind named calls/interfaces.
- Agent instructions: expose applicability before detailed behavior and retrieve deeper instructions only when relevant.

These examples are illustrative, not exhaustive.

## Review Existing Work

Look for overload, hidden necessary detail, mixed abstraction levels, duplicated deep content, several audiences forced through one path, or conclusions buried beneath optional explanation.

## Self-Check

1. What is the consumer trying to do now?
2. What is the minimum sufficient information for that task?
3. What detail is competing for attention too early?
4. What deeper information may become necessary later?
5. Is the route to that information obvious?
