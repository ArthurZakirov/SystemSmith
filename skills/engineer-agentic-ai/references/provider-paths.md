# 🗺️ Provider path reference

Documentation snapshot: 2026-09-27. Re-verify provider-specific paths from the linked [primary sources](#sources) before changing a live installation.

Use this reference when a skill, prompt, plugin, hook, or instruction file needs to mention provider-specific filenames or directories. Prefer concept-first wording in the main instructions, then enumerate concrete provider equivalents only when the filesystem location matters.

## 🗂️ Contents

- [🧭 Core rule](#core-rule)
- [🧩 Concept map](#concept-map)
- [📜 Always-loaded instruction files](#instruction-files)
- [📘 Skills](#skills)
- [⌨️ Custom commands or prompt files](#commands)
- [🤝 Agents or subagents](#agents)
- [⚡ Hooks, plugins, and deterministic controls](#hooks-plugins)
- [⚙️ Settings and general config](#settings)
- [✍️ Recommended wording pattern](#wording)
- [📚 Sources](#sources)

<a id="core-rule"></a>
## 🧭 Core rule

Do not write reusable instructions that mention only one provider-specific file such as `CLAUDE.md` or only one provider-specific directory such as `.claude/skills/` unless the implementation is truly platform-specific.

When a concrete path matters, enumerate the known equivalents across the current target tools and then end with a fallback such as:

`or the equivalent instruction/skill/plugin location for your agentic AI tool`

<a id="concept-map"></a>
## 🧩 Concept map

These names often refer to the same underlying idea even when the provider uses different terminology:

- always-loaded project instructions: `CLAUDE.md`, `AGENTS.md`, or another rule file
- on-demand reusable playbook: skill
- reusable slash action or prompt file: command, slash command, or prompt
- specialized worker persona: agent or subagent
- event-driven deterministic automation: hook or plugin event handler
- extension package: plugin

<a id="instruction-files"></a>
## 📜 Always-loaded instruction files

### Claude Code

- Project-shared instructions: `./CLAUDE.md`
- User-global instructions: `~/.claude/CLAUDE.md`
- Deprecated local project instructions: `./CLAUDE.local.md`

### OpenCode

- Project-shared instructions: `./AGENTS.md`
- User-global instructions: `~/.config/opencode/AGENTS.md`
- Claude-compatible project fallback: `./CLAUDE.md`
- Claude-compatible global fallback: `~/.claude/CLAUDE.md`

### Codex

- User-global instructions: `$CODEX_HOME/AGENTS.md` (normally `~/.codex/AGENTS.md`).
- Project instructions: `AGENTS.md` or `AGENTS.override.md` from the identified project root through the current working directory.
- The default project-root marker is `.git`; `project_root_markers` can configure other markers.
- If no configured marker is found, Codex checks only the current working directory for project guidance. An `AGENTS.md` in a non-project parent is not inherited.

### AGENTS.md ecosystem

- Common project-level path: `./AGENTS.md`
- Nested `AGENTS.md` files are used for subprojects in large repos.

<a id="skills"></a>
## 📘 Skills

### Claude Code

- Project skill: `.claude/skills/<skill-name>/SKILL.md`
- User skill: `~/.claude/skills/<skill-name>/SKILL.md`
- Plugin skill: `<plugin-root>/skills/<skill-name>/SKILL.md`

### OpenCode

- Native project skill: `.opencode/skills/<skill-name>/SKILL.md`
- Native global skill: `~/.config/opencode/skills/<skill-name>/SKILL.md`
- Claude-compatible project skill: `.claude/skills/<skill-name>/SKILL.md`
- Claude-compatible global skill: `~/.claude/skills/<skill-name>/SKILL.md`
- Agent-compatible project skill: `.agents/skills/<skill-name>/SKILL.md`
- Agent-compatible global skill: `~/.agents/skills/<skill-name>/SKILL.md`

### Codex

- User skill root in the installed local tooling: `${CODEX_HOME:-$HOME/.codex}/skills/<skill-name>/SKILL.md`
- No public OpenAI doc in the sources consulted enumerates a repo-local Codex skill directory analogous to `.claude/skills/` or `.opencode/skills/`.

### Agent Skills standard

- The format requires a directory containing `SKILL.md`.
- The standard itself does not mandate one universal discovery path.
- A common ecosystem path is `.agents/skills/<skill-name>/SKILL.md`.

<a id="commands"></a>
## ⌨️ Custom commands or prompt files

### Claude Code

- Project command: `.claude/commands/<command-name>.md`
- User command: `~/.claude/commands/<command-name>.md`
- Plugin command directory: `<plugin-root>/commands/`

### OpenCode

- Project command file: `.opencode/commands/<command-name>.md`
- Global command file: `~/.config/opencode/commands/<command-name>.md`
- JSON-configured commands can also live under the `command` object in `opencode.json`

### Codex

- No equivalent slash-command file convention was found in the sources consulted.
- Map this concept to a skill, plugin capability, automation, or prompt surface depending on the actual runtime.

### FastMCP

- FastMCP has a `prompt` concept, but that is not a shared cross-client filesystem path.
- Treat it as an implementation-specific prompt surface, not as a direct substitute for `CLAUDE.md`, `AGENTS.md`, or a skill directory.

<a id="agents"></a>
## 🤝 Agents or subagents

### Claude Code

- Project subagent: `.claude/agents/<agent-name>.md`
- User subagent: `~/.claude/agents/<agent-name>.md`
- Plugin agent directory: `<plugin-root>/agents/`

### OpenCode

- Project agent: `.opencode/agents/<agent-name>.md`
- Global agent: `~/.config/opencode/agents/<agent-name>.md`
- JSON-configured agents can also live under the `agent` object in `opencode.json`

### Codex

- No public repo-local `agents/` path for Codex itself was found in the sources consulted.
- In the installed local Codex tooling here, `.agents/plugins/marketplace.json` is plugin marketplace metadata, not an agent-definition directory.

<a id="hooks-plugins"></a>
## ⚡ Hooks, plugins, and deterministic control surfaces

### Claude Code

- Hooks are configured in:
  - `~/.claude/settings.json`
  - `.claude/settings.json`
  - `.claude/settings.local.json`
- Plugin manifest: `.claude-plugin/plugin.json`
- Plugin hooks default location: `<plugin-root>/hooks/hooks.json`
- Plugin MCP config: `<plugin-root>/.mcp.json`
- Plugin LSP config: `<plugin-root>/.lsp.json`

### OpenCode

- Plugins load from:
  - `.opencode/plugins/`
  - `~/.config/opencode/plugins/`
- Plugin files are JavaScript or TypeScript modules.
- OpenCode does not use a dedicated plugin manifest analogous to `.claude-plugin/plugin.json` in the sources consulted.
- Permission and other runtime policy commonly live in `opencode.json` or `~/.config/opencode/opencode.json`.
- Hooks are exposed as plugin event handlers, not a separate standard `hooks.json` file.

### Codex

- Plugin manifest in the installed local tooling: `.codex-plugin/plugin.json`
- Repo-local marketplace metadata in the installed local tooling: `.agents/plugins/marketplace.json`
- Home-local marketplace metadata in the installed local tooling: `~/.agents/plugins/marketplace.json`
- The installed local Codex scaffold supports optional companion content such as `skills/`, `hooks/`, `scripts/`, `assets/`, `.mcp.json`, and `.app.json`
- The same local scaffold also uses a manifest placeholder `hooks: ./hooks.json`, so treat Codex hook wiring as manifest-driven rather than assuming a single universal hook directory convention.

<a id="settings"></a>
## ⚙️ Settings and general config

### Claude Code

- User settings: `~/.claude/settings.json`
- Project settings: `.claude/settings.json`
- Local project settings: `.claude/settings.local.json`

### OpenCode

- Project config: `opencode.json` or `opencode.jsonc`
- Global config: `~/.config/opencode/opencode.json`

### Codex

- The sources consulted here do not provide one public cross-surface settings-file path equivalent to Claude Code `settings.json` or OpenCode `opencode.json`.
- Do not invent one. If you need a Codex-specific settings path, verify it from the actual product surface or local installed tooling first.

<a id="wording"></a>
## ✍️ Recommended wording pattern

Prefer concept-first wording:

`Update the always-loaded project instruction file, such as AGENTS.md, CLAUDE.md, or the equivalent instruction file for your agentic AI tool.`

If the skill directory matters:

`Place the skill in the provider’s skill directory, for example .opencode/skills/<name>/SKILL.md, .claude/skills/<name>/SKILL.md, ${CODEX_HOME:-$HOME/.codex}/skills/<name>/SKILL.md, .agents/skills/<name>/SKILL.md, or the equivalent skill location for your agentic AI tool.`

If the plugin manifest matters:

`Update the provider’s plugin manifest, for example .claude-plugin/plugin.json, .codex-plugin/plugin.json, or the equivalent plugin manifest/config surface for your agentic AI tool.`

<a id="sources"></a>
## 📚 Sources

- Claude Code memory, commands, subagents, plugins, and `.claude` directory:
  - [Memory](https://docs.anthropic.com/en/docs/claude-code/memory)
  - [Slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
  - [Subagents](https://code.claude.com/docs/en/sub-agents)
  - [`.claude` directory](https://code.claude.com/docs/en/claude-directory)
  - [Plugin reference](https://code.claude.com/docs/en/plugins-reference)
  - [Plugins](https://code.claude.com/docs/en/plugins)
- OpenCode rules, skills, commands, agents, plugins, permissions, and config:
  - [Rules](https://opencode.ai/docs/rules/)
  - [Skills](https://opencode.ai/docs/skills/)
  - [Commands](https://opencode.ai/docs/commands/)
  - [Agents](https://opencode.ai/docs/agents/)
  - [Plugins](https://opencode.ai/docs/plugins/)
  - [Permissions](https://opencode.ai/docs/permissions)
  - [Configuration](https://opencode.ai/docs/config/)
- `AGENTS.md` and Agent Skills:
  - [`AGENTS.md`](https://agents.md/index)
  - [Agent Skills specification](https://agentskills.io/specification)
  - [Agent Skills quickstart](https://agentskills.io/skill-creation/quickstart)
- OpenAI Codex public documentation:
  - [Introducing Codex](https://openai.com/index/introducing-codex/)
  - [Codex plugins and skills](https://openai.com/academy/codex-plugins-and-skills)
  - [Latest-model guide](https://developers.openai.com/api/docs/guides/latest-model)
  - [Codex configuration reference](https://developers.openai.com/codex/config-reference)
  - [Compaction](https://developers.openai.com/api/docs/guides/compaction)
- Installed local Codex skill/plugin tooling in this environment:
  - `${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-creator/SKILL.md`
  - `${CODEX_HOME:-$HOME/.codex}/skills/.system/plugin-creator/SKILL.md`
  - `${CODEX_HOME:-$HOME/.codex}/skills/.system/plugin-creator/references/plugin-json-spec.md`
