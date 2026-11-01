---
name: information-representation-design
description: "Organize or review human-readable text (chat replies, code explanations, documentation, Markdown, PRs, tickets, emails, websites, articles, agent skills) for clear lists, readable paragraphs, navigation, and exact technical wording. Use for dense comma-separated inventories, walls of prose, vague descriptions of code, or headings and bullets that mix unrelated facts and rules."
---

# Information Representation Design

When composing or restructuring human-readable information, establish **what each section communicates**, then choose **how its contents are represented**. Work from the outer document structure toward the individual sentence, table, snippet, or link.

This skill organizes information already selected for the reader. Use [relevance-first-information-design](../relevance-first-information-design/SKILL.md) to decide which information belongs on the current surface and which should be available through deeper navigation.

## 1. Structure the reader's route

### Establish semantic sections

Before choosing tables, lists, or prose, organize the material around the questions readers need answered.

1. **Identify questions.** Determine what a reader comes to find, understand, decide, or do.
2. **Group answers.** Give related material a shared section, heading, or short lead-in.
3. **Name containers.** Make each label identify what follows, rather than using placeholders such as "Details" or "Miscellaneous".
4. **Place details.** Put examples, evidence, qualifications, and exceptions inside the item they explain.
5. **Keep peers comparable.** Children under the same parent should fulfill the same semantic role and sit at a comparable level.

Apply this from the document title through its headings and nested items. For example:

- **Principles:** each child states a principle, followed by its explanation and example.
- **Reasons:** each child gives a separate reason.
- **Procedure:** each child describes an action or stage in the required order.

A section called "Release principles" should contain actual principles, not a deployment checklist presented as another principle.

**Example — separate semantic roles**

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
- Detect failures before wider deployment.
```

The principles, procedure, and reasons are separate sections because their immediate children perform different jobs.

### Choose the section's internal pattern

A semantic container can organize more than a list of similar items. Select the pattern that answers the reader's current question:

- **Peer items** — a collection such as five test-writing principles, with each principle explained under its own label.
- **State → elaborate → exemplify** — name an idea, explain what it means, then provide a concrete example or evidence.
- **Before → after → how** — show the initial problem, the resulting state, and the work that produced the change.
- **What → why → how** — establish the deliverable, its purpose, and the implementation approach.
- **Planned steps → current status** — distinguish the ordered work ahead from observations already confirmed.
- **Interpretation → next action** — explain how feedback changes the intended work, then separate the next concrete step.
- **Completed → remaining** — distinguish finished work from outstanding work in a status report.
- **Claim → meaning → evidence → impact** — establish an assertion, interpret it, substantiate it, then explain why it matters.

These are possible structures, not mandatory document templates. Select them based on the reader's question and the relationships between the information.

### Elaborate beneath short labels

When listing principles or similar concepts, use a short meaningful name as the heading or bold item label. Aim for **one to three words** when that still names the concept accurately; preserve longer names when needed for meaning or exact identification.

Follow a short abstract label with an explanation and, where it helps, a concrete example.

**Example — one principle in a testing guide**

- **Assert outcomes.** Check the database state after an operation, rather than asserting that an internal helper was called. For example, `assert row["status"] == "ready"` checks an observable result; `mock_writer.assert_called_once()` couples the test to an implementation detail.

A longer document can give the same principle its own heading. A short message may use a bold lead-in without formal headings. In either case, the reader sees what the following explanation belongs to.

### Match navigation to the medium

Headings provide the main structure in documents and articles. In a short chat reply, email, or ticket comment, a lead-in and grouped paragraphs can provide the same navigation without unnecessary headings.

For multi-step work updates, separate **what will happen** from **what has already been verified**. Use numbered steps for the plan and a separate status section for observations. Keep simple one-step updates concise rather than adding unnecessary headings.

**Example — a plan followed by current status**

```markdown
### What I'll do

I'll refactor the skill in these steps:

1. Locate the current repository and read its skill.
2. Restructure the content from section navigation through representation choice.
3. Validate the resulting Markdown and commit the changes.

### Mac status

- **Connection:** Connected.
- **Skills:** Relevant guidance loaded.
- **Changes:** Not yet committed.
```

The two sections answer different reader questions: the first describes the planned sequence; the second reports only verified current state.

**Example — interpretation followed by the next action**

```markdown
### How I'll act on your example

I'll treat your proposed formatting as an example for the skill, not
as a request to reformat this conversation. The skill will teach
agents to separate different kinds of information when needed.

### What I'll do next

I'll finish checking the revised skill and commit the validated change.
```

The first section explains what the feedback means for the work; the second names the next operation. These labels are examples, not required headings for every message.

Name instances by meaning rather than arbitrary labels such as "Principle 1". Keep the outer document title aligned with the questions its sections actually answer.

## 2. Choose a representation inside each section

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

### Example — environment variables

In a section titled **Configure environment variables**, first tell the reader what to configure and where the settings belong. Then show the individual variables as a table.

| Variable | Effect | Example value |
| --- | --- | --- |
| `REPOS_DIR` | Root directory for cloned repositories | `~/Repos` |
| `LOG_LEVEL` | Application log verbosity | `debug` |

These are illustrative settings, not values inspected from a project. The section establishes the purpose; the table lets readers compare fields without rereading repeated prose.

## 3. Apply rules to the chosen format

### Prose and lists

#### Frame each list

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

#### Separate scannable items and distinct rules

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

#### Make paragraphs purposeful

Keep explanatory prose inside the statement, rule, or question it develops.

- Begin with the claim or an identifying lead-in.
- Elaborate when a short label is too abstract.
- Show an example or evidence when the reader needs to see what the claim means.
- Start a new paragraph or list item when the subject, action, or reason changes.

Keep genuinely connected causal reasoning in prose. A new sentence does not automatically require a separate paragraph.

### Tables

Use a table when the reader compares records with the same columns or conditions with the same fields.

- State the table's purpose in a heading or preceding sentence.
- Use each row for one comparable record.
- Put repeated labels in column headers rather than repeating them inside every cell.
- Split unrelated rules or multi-paragraph explanations out of cells and place them under their owning section.
- Prefer a numbered list for a simple sequence instead of a table with one row per step.

### Diagrams and charts

Use a diagram when the relationship, flow, topology, or spatial arrangement is easier to perceive visually. Use an appropriate chart or table when quantitative patterns or comparisons matter.

Introduce what the visualization shows and explain its relevant implication nearby. Keep the surrounding section responsible for the question the visualization answers.

### Code and commands

When an answer locates, explains, or proposes changing repository code, inspect the relevant source before drafting its description. Compare the final explanation against the actual symbols, expressions, and observed behavior.

#### Use exact identifiers and concrete operations

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

#### Quote inspected source with its scope

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

#### Show both sides of changes

For proposed or completed code edits, show the same enclosing scope and relevant lines in two separately labeled excerpts:

- **BEFORE:** the original code and its verified source location.
- **AFTER:** the modified code, labeled as *proposed* when not yet applied.

In a larger annotated AFTER excerpt:

- Mark added lines `# NEW`.
- Mark replaced lines `# CHANGED`.
- Put each commented-out old line directly above its replacement.
- Leave unchanged lines unmarked.

Use the target language's comment syntax. These review annotations explain a change; they are not automatically part of production code.

#### Distinguish quotations from commands

- **Markdown wording:** when discussing what quoted text says, use a blockquote or concise paraphrase with a link. Use a `markdown` fence with the enclosing heading when the syntax itself matters.
- **Runnable commands:** show the actual command in a `sh` or `bash` fence with representative arguments. A command being introduced for execution is not a source quotation and needs no invented file path or enclosing function.

## 4. Link sources and recurring references

### Make source mentions navigable

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

### Resolve links for the destination

| Where readers open the text | Link target |
| --- | --- |
| Chat using a local repository workspace | Relative to the session working directory, with a verified line anchor |
| Markdown inside the same repository | Relative to the containing Markdown file's directory; verify the target path |
| Markdown referring to another repository | Git-host URL to that repository; prefer a stable repository-level destination when file and line links may drift |
| PR, external ticket, documentation app, or messaging app | Absolute URL usable without a local checkout |

A remote Git-host link must refer to a revision that exists there. For an uncommitted change, use a local source locator rather than pretending it is already available remotely.

### Keep repeated references stable

When a fact, rule, question, or example is cited again later in the same explanation, give it a short bold identifier at first mention, such as **F1**, **R1**, or **Q1**. Use the same bold identifier at every subsequent reference.

Use labels only where later references benefit from them; one-off statements need no numbered tag.
