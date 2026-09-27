# 🧭 Interface, routing, and control design

Use this reference when the problem is about whether a skill/tool/server is selected, where trigger criteria belong, or whether prompt wording is being mistaken for enforcement.

## 🔌 Interface vs implementation

| Surface | Pre-selection role | Put here | Do not put here |
| --- | --- | --- | --- |
| Skill frontmatter `description` | Skill retrieval/routing | Observable request patterns, target artifacts, outcome contract | Long procedures or internals |
| MCP tool description | Tool selection | When to call, distinguishing inputs, observable result | Libraries, caching, internal algorithms |
| MCP server description | Capability-area routing | When the server is relevant | Tool-level implementation detail |
| `SKILL.md` body | Post-selection execution | Procedure, constraints, decision rules | Trigger logic that must work before load |
| Tool/script code | Post-invocation behavior | Deterministic implementation | Routing prose |
| UI metadata | Human chooser | Display name and concise human-facing description | Runtime routing logic |

A missed skill invocation is fixed on a pre-selection surface; adding more trigger prose to a body that never loaded cannot fix retrieval.

Skill disclosure is staged: the runtime exposes a metadata catalog before selection, loads the complete `SKILL.md` after selection, and reads supporting references, scripts, or assets only as needed. Keep routing criteria in metadata and conditional detail behind explicit links from the body.

Always-on response behavior is not a conditional workflow. Put it in an always-loaded instruction surface, or use a lifecycle hook, validator, permission, or other stronger control when the miss cost requires enforcement.

**Surface/version observation:** in one Codex desktop realtime session, catalog-bearing developer messages were repeated at `task_started`. This is evidence about that client/version's context assembly, not a portable lifecycle guarantee.

## 🎯 Write routing metadata for retrieval

Name concrete requests, artifacts, mechanisms, and failure modes. Prefer contract-oriented language: what kind of request should route here and what outcome becomes possible. When capabilities overlap, distinguish them by inputs, outputs, and intended use rather than implementation details.

Treat skill descriptions and MCP tool descriptions as the same class of interface: both help the model decide whether to bring a capability into play, even though one expands to instructions and the other to executable code.

## 🛡️ Emphasis is not control

Repeated `always`, `never`, `critical`, duplicated warnings, and all-caps remain prompt-level guidance. If reliability is the problem, diagnose the missing mechanism instead of amplifying prose.

Escalate only as far as required:

1. Fix routing metadata when the capability is not selected.
2. Simplify/split post-load guidance when it is selected but crowded or ambiguous.
3. Use scripts/validators when a deterministic check can encode the requirement.
4. Use hooks, permissions, approval gates, plugins, schedulers, or external automation when the action must fire or be blocked by construction.

Examples: a mandatory post-edit check suggests an event hook or validator; a forbidden command suggests a pre-tool gate/permission; recurring work suggests a scheduler rather than a reminder in context.

## 🔎 Failure-to-surface mapping

| Failure | First surface to inspect |
| --- | --- |
| Skill not loaded | Skill name/description and discovery |
| Wrong tool chosen | Tool/server description and schema |
| Correct skill, wrong execution | Skill body/reference procedure |
| Correct tool, wrong effect | Tool implementation/schema/environment |
| `always`/`never` missed | Stronger deterministic control surface |
| Context overload | Reduce, split, route, and progressively disclose |
| Picker/display problem only | UI metadata |

Do not assume that one provider exposes another provider's hook, permission, plugin, or automation surface. Verify current platform capabilities before choosing the mechanism.
