---
name: format-markdown
description: Create, edit, review, refactor, or rewrite Markdown artifacts while preserving meaning, valid syntax, navigability, and human reviewability. Use whenever work directly changes or reviews a Markdown file, including README.md, AGENTS.md, SKILL.md, reference docs, design docs, or Markdown produced from notes, transcripts, scratch text, or rough drafts.
argument-hint: "[[--RAW] Raw message here..] [[--PART] Which part to apply guidelines to (default all)] [[--PRINCIPLE] Which specific guideline to apply (default all)] [[--SKIP] Which guidelines to skip (default none)]"
---

# Markdown Formatter

User request: `$ARGUMENTS`

Apply [`information-representation-design`](../information-representation-design/SKILL.md) for tables vs lists, hierarchy, prose, diagrams, and other representation choices. Apply [`relevance-first-information-design`](../relevance-first-information-design/SKILL.md) for information selection, abstraction layering, routing, and progressive disclosure. Apply [`thoughts-to-artifact`](../thoughts-to-artifact/SKILL.md) when the source is raw notes, dictation, a transcript, or a brainstorm whose conceptual structure should be recovered rather than copied in source order.

Also apply [`living-artifacts`](../living-artifacts/SKILL.md) when the output contains current-state values, snapshots, file trees, versions, paths, or other data likely to drift.

## Core Task

Create or edit the Markdown artifact so it remains valid, readable, navigable, and faithful to the intended meaning. Treat machine-consumed Markdown such as `AGENTS.md` and `SKILL.md` as dual-audience artifacts: optimize for correct agent use without sacrificing human review and navigation.

## Preservation Rules

- Preserve all key information, nuance, and examples already present.
- Fix obvious spelling, grammar, transcript noise, malformed sentences, and broken Markdown syntax when doing so does not change meaning.
- Do not invent missing content.
- Reorder content only when it improves clarity without changing meaning.
- If extra guidance must be retained without becoming document content, use an HTML comment when appropriate.
## Markdown-Specific Rules

- Preserve literal `$` syntax when it is semantically part of the source.
- Keep YAML/frontmatter valid when the target artifact expects it.
- Use valid heading levels and do not skip levels merely for visual size.
- Use inline code for repositories, folders, files, commands, functions, variables, configuration keys, and other literal technical identifiers when appropriate.
- In maintained Markdown artifacts, make concrete file references clickable with correct relative links when a real target exists.
- Keep code fences syntactically valid and preserve the intended language identifier when known.
- Escape or restructure Markdown syntax when literal characters would otherwise be interpreted incorrectly.

## Self-Check

- Meaning preserved.
- No invented facts.
- Markdown parses as intended.
- Frontmatter and code fences remain valid.
- Every referenced file or path with a resolvable target is a meaningful clickable Markdown link; unresolved targets are identified rather than presented as if linked.
- Machine-consumed Markdown remains easy for a human reviewer to navigate and inspect.
- Representation and relevance/disclosure rules come from their dedicated skills rather than being redefined here.
- When transforming raw thoughts, examples and nuance are preserved according to `thoughts-to-artifact`.
