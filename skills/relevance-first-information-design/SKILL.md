---
name: relevance-first-information-design
description: Design or review any human- or agent-facing information surface so the consumer receives the minimum sufficient information at the right abstraction level for the current task, with clear routes to deeper detail only when relevant. Use for chat responses, code structure, APIs, READMEs, docs, AGENTS.md, skills, Confluence or Notion pages, dashboards, charts, websites, resumes, profiles, articles, social posts, messages, UIs, or other communication and interaction surfaces.
---

# Relevance-First Information Design

## Core Principle

Present information in layers of relevance and abstraction.

The consumer should encounter the minimum sufficient information needed for the current action, decision, understanding, or navigation step. Additional detail should remain accessible through an intentional drill-down path rather than competing for attention by default.

This applies whether the consumer is a human or an agent, and whether the surface is prose, code, UI, visualization, navigation, or structured data.

## The Two Requirements

A good information surface must satisfy both:

- **Low unnecessary cognitive load:** do not force the consumer to process detail irrelevant to the current goal.
- **No hidden necessary information:** provide an obvious route to any deeper information needed to continue correctly.

Compression without a route to detail is omission. Detail without routing is overload.

## Separate Routing From Detail

Treat each layer as answering a different question:

- **Routing layer:** Is this relevant to me now? Where should I go next?
- **Working layer:** What do I need to act or understand at this level?
- **Deep layer:** What implementation detail, evidence, history, or explanation do I need only if I drill down?
## Conditional Information

When information or behavior is conditional, keep applicability visibly separate from what follows.

- **When / condition:** what observable situation, audience, goal, or state makes the branch relevant.
- **Then / content or behavior:** what to show, read, do, execute, or inspect once that condition applies.

Use bold labels, tables, links, expandable sections, navigation, functions, modules, hover details, threads, or another mechanism suited to the medium. The mechanism may differ; the principle does not.

## Cross-Medium Patterns

Examples of the same principle in different media:

- **Chat:** concise answer first; offer or link to deeper analysis instead of dumping it inline.
- **Messaging:** short top-level message; move optional detail into a thread, call, attachment, or linked document.
- **README/docs:** route users, maintainers, and contributors to different deeper pages instead of mixing all procedures together.
- **Resume/profile:** concise evidence at the top level; link to fuller project stories or proof when useful.
- **Website/app:** overview first; reveal advanced controls, explanations, or details on demand.
- **Dashboard/chart:** show the primary signal by default; expose secondary dimensions via hover, drill-down, filters, or detail views.
- **Code:** keep functions and modules at a coherent abstraction level; hide lower-level implementation behind well-named calls and interfaces.
- **Agent instructions:** make trigger/applicability clear before detailed behavior; load or retrieve deeper instructions only when relevant.

These are examples, not an exhaustive trigger list. Apply the principle to any information-bearing interface.

## Review Existing Work

When reviewing an artifact, actively look for mismatched information density or abstraction levels even if the user did not name this principle.

Typical failure modes:

- walls of text with no routing structure;
- several audiences forced through the same path;
- low-level implementation detail mixed into high-level explanation;
- required detail hidden without a visible path;
- duplicate detail copied into multiple surfaces instead of linked from a canonical source;
- dashboards, charts, or UIs that expose every dimension simultaneously;
- code whose parent function mixes orchestration with low-level mechanics;
- chat answers whose useful conclusion is buried under optional explanation.
## Design Procedure

For any information-bearing surface:

1. Identify the consumer and current goal.
2. Identify the minimum sufficient information needed at the current layer.
3. Separate information that belongs to deeper or different branches.
4. Provide explicit routes to those branches when they may become relevant.
5. Keep each layer internally coherent in abstraction level.
6. Review whether the default view contains unnecessary detail or hides necessary next steps.

## Self-Check

Before finishing, ask:

1. What is the consumer trying to do right now?
2. What is the minimum sufficient information for that task?
3. What detail is currently competing for attention too early?
4. What deeper information could become necessary later?
5. Is there an obvious route to that deeper information?
6. Are abstraction levels mixed in the same surface?
7. Could the same underlying content be represented more clearly through hierarchy, links, drill-down, encapsulation, or on-demand disclosure?
