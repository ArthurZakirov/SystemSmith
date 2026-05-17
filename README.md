# SystemSmith

Engineering methods for code, agents, and architecture.

SystemSmith turns recurring engineering judgment into reusable agent skills: how to scope work, review your own output, write durable docs, protect secrets, structure technical messages, reason about agentic systems, and build cleaner software and information architectures.

The repo exists so engineering methods can compound across tools instead of being re-explained in every session.

## What It Helps With

- Turn tickets and vague requests into a rigorous engineering workflow.
- Review work before handing it back.
- Keep credentials and private material out of agent outputs.
- Write Markdown and technical messages clearly.
- Produce living artifacts that can be refreshed instead of going stale.
- Translate agentic-AI ideas into concrete runtime mechanics.

## Install With skills.sh

List the skills in this repo:

```bash
npx skills add https://github.com/ArthurZakirov/SystemSmith --list
```

Install all skills for Codex on a machine:

```bash
npx skills add https://github.com/ArthurZakirov/SystemSmith --skill '*' -a codex -g -y
```

Install one specific skill:

```bash
npx skills add https://github.com/ArthurZakirov/SystemSmith --skill swe-method -a codex -g -y
```

## Install As A Claude Code Plugin

```text
/plugin marketplace add ArthurZakirov/SystemSmith
/plugin install systemsmith@arthur-zakirov
```

Claude Code plugin skills are namespaced by plugin name, for example:

```text
/systemsmith:swe-method
```

## Install As A Codex Plugin

This repo includes Codex plugin packaging:

- repo marketplace metadata at [`.agents/plugins/marketplace.json`](./.agents/plugins/marketplace.json)
- plugin manifest at [`.codex-plugin/plugin.json`](./.codex-plugin/plugin.json)

Add the marketplace:

```bash
codex plugin marketplace add ArthurZakirov/SystemSmith
```

Then install from Codex with `/plugins`.

For local development from a cloned repo:

```bash
./scripts/setup-local-links.sh
```

Existing non-symlink paths are left untouched unless `--force` is used.

## Included Skills

<!-- BEGIN GENERATED SECTION: skills -->
> Generated from tracked `skills/*/SKILL.md` metadata.

| Skill | Description |
| --- | --- |
| `engineer-agentic-ai` | Translate natural-language requests about modifying an AI coding agent, its skills, MCP tools, MCP servers, prompts, routing, tool descriptions, context loading, memory, portability, or other agentic behavior into the concrete mechanics of the target system. Use when a request is underspecified, phrased in human terms instead of runtime terms, or risks making the wrong change because the agent must reason about how Codex, Claude Code, OpenCode, MCP-based systems, or similar runtimes select skills, choose tools, load instructions, apply configuration, and stay transferable across users and machines. |
| `format-markdown` | Generate properly formatted Markdown from unstructured notes, scratch text, transcripts, or rough drafts. Use when the user wants content rewritten into cleaner Markdown without inventing new information. |
| `living-artifacts` | Apply when producing any markdown artifact for someone else (or future-you) to read — READMEs, analysis documents, Confluence pages, status reports, troubleshooting write-ups, skill content, conversation summaries, ticket descriptions, post-mortems, snapshots of system state, or any document that contains concrete current-state values. Defines the "snapshot + reproducer" rule (every static value must be paired with the command/query/script that regenerates it), the single-source-of-truth rule (reference canonical homes instead of duplicating values), and fact-vs-assumption marking. Trigger keywords include writing or producing or creating or drafting a README, analysis, Confluence page, status update, summary, snapshot, current state, documenting data, posting findings, file tree, port number, version number, table count, or any concrete value that may drift over time. |
| `messaging-framework` | Core principles for technical communication such as commits, PR descriptions, branch names, and technical documentation. Use when writing or refining engineering-facing messaging and when other skills need a shared communication baseline. |
| `security` | Mandatory credential protection skill for all agents. Prevents reading, exposing, or leaking credentials, secrets, API keys, tokens, passwords, and auth-capable config files. |
| `self-review` | Analyze the current conversation or mine historical chat sessions for correction patterns, wrong conclusions, or unnecessary user intervention. Use when the user wants to improve prompts, skills, workflows, or guardrails based on repeated mistakes. |
| `swe-method` | Guide for working through a Jira story efficiently by separating context gathering, immutable evidence collection, codebase exploration, human-in-the-loop specification refinement, execution planning, and implementation. Use when turning a ticket into a reliable engineering workflow instead of jumping straight to code. |
<!-- END GENERATED SECTION: skills -->

## Available Commands

<!-- BEGIN GENERATED SECTION: commands -->
> Generated from tracked `commands/*.md` files.

| Command | Summary |
| --- | --- |
| `/list-skills` | Please list all your available skills with a 1 sentence description for each one. Do not return any additional fluff text before or after. |
<!-- END GENERATED SECTION: commands -->

## Repo Inventory

<!-- BEGIN GENERATED SECTION: repo_inventory -->
> Generated from tracked manifests, scripts, commands, and skills.

```text
.
├── .agents/
│   ├── plugins/marketplace.json
│   └── skills -> ../skills
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
├── .claude/
│   ├── commands -> ../commands
│   └── skills -> ../skills
├── .codex-plugin/
│   └── plugin.json
├── .githooks/
│   └── pre-commit
├── .github/
│   └── workflows/
│       └── readme-generated.yml
├── commands/
│   └── list-skills.md
├── scripts/
│   ├── create-claude-command.sh
│   ├── create-shared-skill.sh
│   ├── generate-readme.py
│   ├── install-git-hooks.sh
│   ├── setup-local-links.sh
│   └── update-readme.sh
├── skills/
│   ├── engineer-agentic-ai/
│   ├── format-markdown/
│   ├── living-artifacts/
│   ├── messaging-framework/
│   ├── security/
│   ├── self-review/
│   └── swe-method/
├── pyproject.toml
└── uv.lock
```
<!-- END GENERATED SECTION: repo_inventory -->

## Development

Use `./scripts/update-readme.sh` after adding or removing tracked skills, commands, scripts, or plugin metadata.
