---
name: information-representation-design
description: "Choose and review the representation and reader navigation of any human-readable text: chat replies, messages, emails, social posts, PR and ticket descriptions, documentation, agent skills, websites, articles, and resumes. Use when organizing prose, lists, tables, headings, and nested sections so readers can find answers and sibling elements have coherent semantic roles."
---

# Information Representation Design

## Core Principle

Choose the representation that makes important information and relationships easiest for the reader to locate and understand, with the least repeated scaffolding.

A reader may want only one answer, not a linear tour of everything the author knows. Organize text so the reader can identify relevant content from its labels and follow the appropriate path without reconstructing the author's intent.

This skill decides **how information already in scope is organized and represented**. Deciding which information belongs on the current surface is the responsibility of [`relevance-first-information-design`](../relevance-first-information-design/SKILL.md).

## Representation Mapping

| Information shape | Preferred representation |
| --- | --- |
| Repeated items with the same fields or dimensions | Table |
| Ordered or procedural sequence | Numbered list |
| Unordered independent points of one kind | Bulleted list |
| Parent/child concepts at different abstraction levels | Hierarchy, headings, or nested navigation |
| Narrative reasoning where sequence and causal flow matter | Prose |
| Relationships, flow, topology, or spatial structure | Diagram when it materially improves comprehension |
| Quantitative comparison or trend | Appropriate chart or table |
| One compact condition and one action | Explicit `When` / `Then` labels may be clearer than a table |

## Reader Navigation

When drafting or restructuring text for a reader:

1. **Identify the reader's questions.** What might they come here to find, decide, or do?
2. **Group information by answer.** Put each answer where its reader would expect it, rather than following the order in which the author happened to think of it.
3. **Name the groups.** Use titles, headings, labels, or lead-ins that tell the reader what information follows before they read the details.
4. **Guide the drill-down.** Keep explanations and evidence under the relevant group so the reader can skip unrelated material without losing context.

A label can name a category (`Reasons to migrate`), describe a task (`How to migrate safely`), or state the question answered (`Why the migration matters`). Question-word headings do not need a question mark. Prefer specific labels over generic labels such as `Details` or `Miscellaneous`.

Do not require formal headings in every medium: a short chat message, email, or social post can use a clear opening sentence and a few grouped lines. The **navigation function** matters, not a particular Markdown syntax.

## Semantic Containers

A **container** tells the reader what kind of information its immediate children contain. This applies recursively:

| Container | Immediate children should represent |
| --- | --- |
| Document title or descriptive filename | Sections that collectively fulfill the document's stated purpose |
| Section such as `Principles of X` | Individual principles of X, each with its own explanation |
| Section such as `Reasons to choose X` | Individual reasons to choose X |
| Section such as `How to perform X` | Actions or stages of that procedure |
| Lead-in such as `Follow these rules when X occurs` | Rules applicable when X occurs |

Apply these rules at **every level**, including document titles, section headings, nested headings, and list items:

- **Define the children's type.** The parent communicates the category, question, or purpose that unites its immediate children. Each child must satisfy that contract. For example, each subsection under `Principles` must actually be a principle, not a workflow or summary checklist.
- **Keep siblings at the same semantic level.** Do not mix a rule, a reason, a procedure, and an exception as if they were instances of one type. Split distinct types into separate named groups, or place supporting detail under the item it qualifies.
- **Keep explanation inside its owner.** A principle can contain prose, examples, evidence, and exceptions that explain or qualify that principle; those details need not become siblings of the principle itself.
- **Name instances by meaning.** Under `Principles`, use names such as `Preserve rollback paths`, not `Principle 1`. Under `Tips`, name the actual tip, not merely `Tip 2`.
- **Align the outermost label with the contents.** When the filename or title promises a particular subject or class of information, its sections must collectively deliver that promise. Rename or regroup when they do not.

A quick test for any group: **"Every item immediately below this heading is a ___."** If no single truthful completion exists, the group needs a clearer parent or different boundaries.

### Example: Separate principles from procedures

**Incoherent hierarchy:** A section called `Release principles` contains `Make failures visible`, `Deployment checklist`, and `Why staged rollout matters` as peer subsections. Only the first is a principle.

**Coherent hierarchy:** Give different semantic roles different containers:

```markdown
## Release principles

### Make failures visible
Explain the principle and its implications.

### Preserve rollback paths
Explain the principle and its implications.

## Deployment procedure

1. Validate the candidate release.
2. Roll out gradually.
3. Verify the results.

## Reasons for staged rollout

- Limit the impact of regressions.
- Create opportunities to detect failures early.
```

The principle explanations can contain local examples or qualifications; the deployment procedure and reasons are not additional principles.

## Framing Lists

When introducing a multi-item list, make its **shared meaning and purpose** clear *before* the items:

1. **Name the container.** A heading, label, or compact opening clause tells the reader what kind of items follow.
2. **Provide a lead-in.** A short sentence explains what the items are for, when they apply, or how to use them, including order when order matters. Do not repeat the heading without adding context.
3. **Make items self-identifying.** Use meaningful short labels when helpful, then expand them with supporting detail. Use numbered items for a required sequence and bullets for unordered peers.

**Unframed:**

```markdown
## Review

- PR A
- PR B
- PR C
```

**Framed:**

```markdown
## Pull requests to review

Review these three PRs in the listed priority order. Read each description
before inspecting its diff:

1. **PR A** — Fix the data-loss scenario.
2. **PR B** — Remove duplicate processing.
3. **PR C** — Simplify the deployment path.
```

In a short message, a single sentence can serve as both the container label and lead-in; avoid adding headings merely to satisfy a template.

## Structural Rules

- When several records repeat the same labels, encode those labels once as table columns instead of repeating headings or `When`/`Then` blocks.
- Do not use a table for a simple linear flow solely because multiple steps exist.
- Group related material by concept rather than preserving an arbitrary source order.
- Use headings to expose real hierarchy, not merely to add visual decoration.
- Prefer a representation that supports scanning when the reader is likely to compare or navigate rather than read linearly.
- Avoid duplicating content just to fit a representation; use references or links when one canonical source should serve several views.
- Do not force a fixed document template on unrelated artifacts. The relevant reader questions determine the sections, and medium constraints determine whether sections are expressed as headings, labels, paragraphs, or another structure.

## Cross-Medium Interpretation

The representation primitive depends on the medium. A table in Markdown may correspond to columns or cards in a UI, grouped fields in a form, a matrix in a dashboard, or typed records in code.

The same container and navigation rules apply when drafting a PR description, ticket, documentation page, agent instruction, website, article, resume, chat reply, Slack message, email, or social post. For example, a PR reader may look for motivation, changes, and blockers; a ticket reader may look for user requirements, constraints, and acceptance criteria. **Those are examples of reader questions, not mandatory heading templates.**

## Self-Check

1. What answer or relationship is the reader looking for, and can they locate it from the visible structure?
2. For each heading, title, or list lead-in, what kind of children does it promise? Does **every immediate child** fulfill that promise?
3. Are reasons, rules, steps, examples, and exceptions correctly grouped or nested rather than mixed as peers?
4. Does each list have enough framing for the reader to understand what the items are and how to use them?
5. Do names of sections and individual items communicate their actual meaning, rather than generic numbers or labels?
6. Does the representation expose order, comparison, hierarchy, causality, or grouping without unnecessary repeated scaffolding?
7. Does the structure fit the medium and expected interaction, without inventing headings or hiding necessary detail?
