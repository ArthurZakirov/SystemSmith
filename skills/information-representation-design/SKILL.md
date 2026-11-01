---
name: information-representation-design
description: "Organize or review human-readable text (chat replies, code explanations, documentation, Markdown, PRs, tickets, emails, websites, articles, agent skills) for clear lists, readable paragraphs, navigation, and exact technical wording. Use for dense comma-separated inventories, walls of prose, vague descriptions of code, or headings and bullets that mix unrelated facts and rules."
---

# 🧩 Information Representation Design

Use these **four stages in order** when composing or restructuring information for a reader. Each numbered section below is a stage of the workflow, not a standalone principle:

1. **[Structure the route](#stage-1).** Define what questions the sections answer and how their children are organized.
2. **[Choose representations](#stage-2).** Select prose, lists, tables, code, or visuals for the content inside each section.
3. **[Apply format rules](#stage-3).** Use the applicable rules for each representation actually selected.
4. **[Anchor references](#stage-4).** Resolve source links and repeated references before handoff.

Return to an earlier stage if a later choice reveals a grouping or labeling problem.

This skill organizes information already selected for the reader. Use [relevance-first-information-design](../relevance-first-information-design/SKILL.md) to decide which information belongs on the current surface and which should remain behind deeper navigation.

<a id="stage-1"></a>
## 🧭 Stage 1 — Structure the reader's route

Follow these **four steps in order** to establish the reader's navigation and the meaning of every section.

**Steps in this stage:**

1. [Step 1.1 — Define the answer containers](#step-1-1)
2. [Step 1.2 — Choose the internal pattern](#step-1-2)
3. [Step 1.3 — Label each answer and attach its detail](#step-1-3)
4. [Step 1.4 — Adapt navigation to the medium](#step-1-4)

<a id="step-1-1"></a>
### Step 1.1 — Define the answer containers

Identify the questions the reader needs answered and group the corresponding material. For each parent heading or list lead-in, decide what type of children it promises:

- **Steps:** actions or stages to carry out in a specified order.
- **Principles:** independent guidelines, each with supporting explanation.
- **Reasons:** separate answers to why something matters or was chosen.
- **Alternatives:** comparable options for a choice.
- **Examples:** concrete illustrations of a rule already introduced.
- **Information categories:** different content types collected under a clearly named purpose.

Before introducing sibling headings, **state their shared type and whether order matters** in the parent section's lead-in. Give steps numbers when sequence is required. Give principles labels such as P1, P2, and P3 when those labels help track independent principles; the numbers do not imply execution order.

If existing siblings have different roles, regroup them beneath suitable parents. An example belongs within the rule it illustrates, not beside that rule as if both were rules.

**Example — identify the meaning of the children**

**Ambiguous:** A heading called "Release guidance" is followed by "Validate a release", "Preserve rollback paths", and "Why staged rollout matters", without explaining whether these are steps, principles, or reasons.

**Clear:** Separate their semantic roles:

```markdown
## Release procedure

Perform these steps in order:

1. Validate the candidate release.
2. Roll out gradually.
3. Verify the result.

## Release principles

Apply these principles during the procedure:

### P1 — Make failures visible
Explain the principle and its implications.

### P2 — Preserve rollback paths
Explain the principle and its implications.

## Reasons for staged rollout

- Limit the impact of regressions.
- Detect failures before wider deployment.
```

Here, the numbered items are procedure steps, P1/P2 are principles, and the final bullets are reasons. Each parent explicitly defines its children.

<a id="step-1-2"></a>
### Step 1.2 — Choose the internal pattern

Select a relationship that matches the questions answered inside each container. These are **alternative patterns**, not stages to perform in sequence:

- **Peer items** — distinct principles or reasons, each explained under its own label.
- **State → elaborate → exemplify** — identify a claim, explain its meaning, then provide evidence or an example.
- **Before → after → how** — explain the initial situation, resulting state, and what produced the change.
- **What → why → how** — identify a deliverable, its purpose, and the means of producing it.
- **Completed → remaining** — separate finished work from outstanding work.
- **Claim → meaning → evidence → impact** — introduce a statement, interpret it, substantiate it, and explain its consequences.

Use a pattern only when the information actually follows that relationship; do not turn illustrative patterns into mandatory heading templates.

<a id="step-1-3"></a>
### Step 1.3 — Label each answer and attach its detail

For each meaningful block of text, identify the reader question it answers. Assign a **short, skimmable semantic label** that tells the reader what follows, then keep its explanation, example, evidence, and qualifications directly beneath it.

- Prefer a meaningful one-to-three-word label when it remains accurate. Use a longer label when needed to preserve meaning.
- Use a heading or bold lead-in depending on the surrounding hierarchy.
- Keep the label and its elaboration in the same container.
- When the reader's question changes, introduce the next labeled block rather than continuing unframed prose.

**Example — one principle in a testing guide**

- **Assert outcomes.** Check the database state after an operation rather than asserting that an internal helper was called. For example, `assert row["status"] == "ready"` checks an observable result; `mock_writer.assert_called_once()` ties the test to an implementation detail.

A longer testing guide may give this principle a heading; a concise list may use the bold lead-in shown above.

<a id="step-1-4"></a>
### Step 1.4 — Adapt navigation to the medium

In documents and articles, headings usually make the route visible. In shorter chat replies, emails, or ticket comments, a brief lead-in or bold label may give sufficient navigation without formal headings.

Compare each container with its immediate children before moving to Stage 2:

- **Parent contract:** can you say "every immediate child of this section is a [step/principle/reason/etc.]"?
- **Sibling role:** are children comparable, rather than an example or exception promoted to the rank of a rule?
- **Ordering:** is any sequence clear from the lead-in and numbering?
- **Reader questions:** can the reader identify the answer from its label without first reading unframed paragraphs?

Correct mismatches while structuring the draft, before choosing formats for its content.

<a id="stage-2"></a>
## 🗂️ Stage 2 — Choose a representation inside each section

After a section has a clear purpose, select a format for each part of its content. A section can contain multiple formats: a short explanation followed by a table, a diagram, or a code excerpt.

| Information within the section | Representation |
| --- | --- |
| Explanation or causal reasoning | Prose |
| Independent items of the same kind | Bulleted list |
| Actions in a required order | Numbered list |
| Repeated records with shared fields | Table |
| A command, expression, or source implementation | Inline code or code fence |
| Relationships, dependencies, or spatial structure | Diagram |
| Quantitative comparison or trend | Chart or table |
| Parent/child navigation | Nested headings or hierarchy |

Choose the format that makes the relevant relationship visible. Use labeled `When` / `Then` prose for one simple condition and action; use a table when several conditions share the same fields.

**Example — environment variables**

In a section titled **Configure environment variables**, first tell the reader what to configure and where the settings belong. Then show the individual variables as a table.

| Variable | Effect | Example value |
| --- | --- | --- |
| `REPOS_DIR` | Root directory for cloned repositories | `~/Repos` |
| `LOG_LEVEL` | Application log verbosity | `debug` |

These are illustrative settings, not values inspected from a project. The section establishes the purpose; the table lets readers compare fields without rereading repeated prose.

<a id="stage-3"></a>
## 🎛️ Stage 3 — Apply rules to the chosen format

The four subsections below are **representation categories**, not ordered steps. Apply only the rules for formats used in the current document.

### 📝 Prose and lists

Apply the following three **text rules** to prose and lists:

#### T1 — Frame each list

Use a short label or lead-in to say what its children represent and why the reader needs them.

- **Bullets:** unordered items of the same kind.
- **Numbers:** steps whose order matters.
- **Short prose:** connected explanation where causal flow matters more than scanning separate entries.

Give list items meaningful names when useful. For a short message, one sentence can serve as both the label and lead-in.

**Example — a priority list**

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

Review these in priority order; read the description before each diff:

1. **PR A** — Fix the data-loss scenario.
2. **PR B** — Remove duplicate processing.
3. **PR C** — Simplify the deployment path.
```

#### T2 — Separate scannable items and distinct rules

When one sentence contains three or more independently scannable items, replace the enumeration with a lead-in and one bullet per item. Keep inseparable pairs, such as "staged versus unstaged changes", together.

When the text changes from describing contents to stating rules, exceptions, or consequences, start a second explicitly named group.

**Hard to scan:**

> The updater preserves staged versus unstaged changes, untracked and ignored files, symlinks, executable modes, conflicted indexes, and previous commits. Submodule copies receive independent Git metadata. Interrupted-operation metadata is saved as evidence; archived operations are not automatically resumed in recovery copies.

**Easier to scan:**

The updater preserves:

- staged versus unstaged changes
- untracked and ignored files
- symlinks
- executable modes
- conflicted indexes
- previous commits

The following rules apply:

- Submodule copies receive independent Git metadata.
- Interrupted-operation metadata is saved as evidence.
- Archived operations are not automatically resumed in recovery copies.

#### T3 — Make paragraphs purposeful

Keep explanatory prose inside the statement, rule, or question it develops.

- Begin with the claim or an identifying lead-in.
- Elaborate when a short label is too abstract.
- Show an example or evidence when the reader needs to see what the claim means.
- Start a new paragraph or list item when the subject, action, or reason changes.

Keep genuinely connected causal reasoning in prose. A new sentence does not automatically require a separate paragraph.

### 📋 Tables

Use a table when the reader compares records with the same columns or conditions with the same fields.

- State the table's purpose in a heading or preceding sentence.
- Use each row for one comparable record.
- Put repeated labels in column headers rather than repeating them inside every cell.
- Split unrelated rules or multi-paragraph explanations out of cells and place them under their owning section.
- Prefer a numbered list for a simple sequence instead of a table with one row per step.

### 📊 Diagrams and charts

Use a diagram when the relationship, flow, topology, or spatial arrangement is easier to perceive visually. Use an appropriate chart or table when quantitative patterns or comparisons matter.

Introduce what the visualization shows and explain its relevant implication nearby. Keep the surrounding section responsible for the question the visualization answers.

### 💻 Code and commands

Apply the following four **code rules** when presenting inspected source or executable commands.

When an answer locates, explains, or proposes changing repository code, inspect the relevant source before drafting its description. Compare the final explanation against the actual symbols, expressions, and observed behavior.

#### C1 — Use exact identifiers and concrete operations

Copy the original spelling of any inspected identifier: every character, including punctuation and underscores. Use inline backticks for literal code; use verified links when referring to a located symbol.

**Identifier example**

- **Bad:** "the underscore keep intervals before rebuild function"
- **Good:** `_keep_intervals_before_rebuild`

Explain concrete source operations rather than replacing them with metaphors or abstract jargon. "Left edge", "tail", "owns", "partition", and "idempotent" describe different words but the same failure when they hide the relevant expression. Use technical terminology when it adds meaning after the executable operation is identified.

**Rows processed — illustrative**

- **Bad:** "The left-edge partition owns the earlier days."
- **Good:** "Rows where `state_date < rebuild_start` are passed to `_keep_intervals_before_rebuild`."

**Condition's effect — illustrative**

- **Bad:** "The older dates are kept."
- **Good:** "Rows where `state_date < rebuild_start` are kept; rows where `state_date >= rebuild_start` are rebuilt."

**Incorrect output — hypothetical**

- **Bad:** "The old left edge is wrong."
- **Good:** "With `valid_from='2026-10-01'` and `rebuild_start='2026-10-10'`, the output starts at `2026-10-01`; the expected start is `2026-10-10`."

Use representative inputs and resulting values when they make an operation easier to understand. Distinguish observed behavior from a hypothetical demonstration.

#### C2 — Quote inspected source with its scope

When describing existing code, quote its decisive lines. The excerpt establishes **what** is implemented; accompanying prose explains the consequence.

Give every source excerpt two location cues:

- **Immediately above:** a verified clickable file-and-line link.
- **Inside the fence:** the exact source-relative path on the first line as a comment.

Keep enough enclosing syntax to show where the quoted code runs. Mark omitted spans with the language's comment syntax.

- **Python:** include the actual `def` and, for a method, its enclosing `class`. Mark omissions `# ...`.
- **Markdown:** include the surrounding heading. Mark omissions `<!-- ... -->` when quoting literal Markdown.
- **YAML:** include the parent keys. Mark omissions `# ...`.
- **JSON:** include the enclosing keys. Use a `jsonc` fence for annotated excerpts containing `//` comments.
- **SQL:** include the `SELECT` or CTE name. Mark omissions `-- ...`.
- **Other languages:** include the nearest meaningful declaration and use the language's comment syntax.

Use a language-tagged fence. Verify copied lines against the correct repository revision before linking them.

#### C3 — Show both sides of changes

For proposed or completed code edits, show the same enclosing scope and relevant lines in two separately labeled excerpts:

- **BEFORE:** the original code and its verified source location.
- **AFTER:** the modified code, labeled as *proposed* when not yet applied.

In a larger annotated AFTER excerpt:

- Mark added lines `# NEW`.
- Mark replaced lines `# CHANGED`.
- Put each commented-out old line directly above its replacement.
- Leave unchanged lines unmarked.

Use the target language's comment syntax. These review annotations explain a change; they are not automatically part of production code.

#### C4 — Distinguish quotations from commands

- **Markdown wording:** when discussing what quoted text says, use a blockquote or concise paraphrase with a link. Use a `markdown` fence with the enclosing heading when the syntax itself matters.
- **Runnable commands:** show the actual command in a `sh` or `bash` fence with representative arguments. A command being introduced for execution is not a source quotation and needs no invented file path or enclosing function.

<a id="stage-4"></a>
## 🔗 Stage 4 — Anchor sources and recurring references

Apply the following three **reference rules** to mentions of inspectable sources and to recurring labels. These are complementary requirements, not ordered actions.

### L1 — Make source mentions navigable

When the draft names a file, symbol, or other inspectable artifact, locate its real target before adding a link. Verify that the path and line exist in the relevant revision or worktree.

- Use the symbol or filename as the visible link label.
- Put the location and line anchor in the href, not in an unnecessarily long visible label.
- Keep link labels as plain text; use inline backticks separately for exact code spelling.
- Preserve the real identifier even when its name is long.
- If a leading `_` or `*` breaks a renderer's link label, omit that character **only in the displayed label**, keeping the complete identifier at the destination or in adjacent inline code.
- For unresolved targets, show the exact known locator in backticks and state what could not be verified.

**Link-label example**

- **Bad:** `[here](src/loader.py#L90)` or `[line 90](src/loader.py#L90)`
- **Good:** `[loader.py:90](src/loader.py#L90)`

### L2 — Resolve links for the destination

| Where readers open the text | Link target |
| --- | --- |
| Chat using a local repository workspace | Relative to the session working directory, with a verified line anchor |
| Markdown inside the same repository | Relative to the containing Markdown file's directory; verify the target path |
| Markdown referring to another repository | Git-host URL to that repository; prefer a stable repository-level destination when file and line links may drift |
| PR, external ticket, documentation app, or messaging app | Absolute URL usable without a local checkout |

A remote Git-host link must refer to a revision that exists there. For an uncommitted change, use a local source locator rather than pretending it is already available remotely.

### L3 — Keep repeated references stable

When a fact, rule, question, or example is cited again later in the same explanation, give it a short bold identifier at first mention, such as **F1**, **R1**, or **Q1**. Use the same bold identifier at every subsequent reference.

Use labels only where later references benefit from them; one-off statements need no numbered tag.
