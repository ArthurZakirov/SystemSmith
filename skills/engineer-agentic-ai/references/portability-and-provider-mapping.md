# 🌐 Portability and provider mapping

Use this reference when reusable agent artifacts risk leaking one person's paths, identity, operating system, or provider terminology into the design. For exact current provider paths, read `provider-paths.md`.

## 🧳 Portability rule

Treat local names, absolute home paths, hostnames, and one-machine assumptions as dependencies that must be removed or made explicit. Prefer environment variables, repository-relative paths, runtime discovery, and configuration surfaces.

| Avoid | Prefer |
| --- | --- |
| `/home/arthur/...` or `C:\\Users\\Arthur\\...` | `${HOME}`, documented root variables, config, or discovery |
| Hard-coded repository root | Repository-relative paths |
| One fixed binary location | Configurable command/path lookup |
| Personal names in reusable instructions | `user`, `I`, `you`, or role names |
| One OS/shell implied silently | Explicit platform scope or portable construction |

## 👤 Identity and product names

Use `you` for the current agent and `user`/`I` for the human unless a specific identity is semantically required. Product names belong only where platform-specific mechanics, files, limitations, or setup make the distinction necessary.

Translate requests like “make Codex do X” into the underlying cross-provider concept first. Isolate genuinely provider-specific implementation in the smallest possible section rather than letting vendor wording leak through the whole artifact.

## 🗺️ Provider equivalents

When a reusable artifact refers to a provider-specific instruction file, skill directory, command, agent/subagent, hook, or plugin, name the shared concept first and then enumerate relevant provider equivalents. Do not imply feature parity; an equivalent may use a different mechanism or may not exist.

Read `provider-paths.md` whenever an exact filename or directory matters. If the target ecosystem is broader than the enumerated providers, end with an explicit fallback such as “or the equivalent surface for the current agentic AI tool.”