# 🧪 Codex configuration-freshness experiments

Documentation snapshot: 2026-09-27. These results are bounded observations from the recorded Codex versions, surfaces, workspace scopes, and session lifecycles. Re-run the procedures before applying them to another build or lifecycle.

Use this reference when diagnosing whether an existing Codex CLI thread or continuously open Codex desktop chat has incorporated changed project instructions or skill content. For the conceptual runtime model, start with the [`Agent runtime model`](agent-runtime-model.md#session-freshness).

## 🗂️ Contents

- [🎯 Question and evidence boundaries](#question-and-boundaries)
- [⌨️ Controlled Codex CLI resume experiment](#cli-resume-experiment)
- [🖥️ Controlled Codex desktop experiment](#desktop-experiment)
- [🔀 Cross-surface interpretation](#cross-surface-interpretation)
- [🔁 Reproduction procedure](#reproduction)

<a id="question-and-boundaries"></a>
## 🎯 Question and evidence boundaries

Official OpenAI documentation establishes `AGENTS.md` discovery/injection and skill progressive disclosure, but does not promise universal hot-reload semantics for a continued Codex CLI or desktop session. These controlled local experiments therefore changed one configuration surface at a time and inspected targeted rollout evidence rather than inferring freshness from the final prose response.

The results establish behavior only for the tested **surface + version + configuration + workspace scope + session lifecycle**. They do not establish behavior for a continuously open CLI TUI, another Codex build, another skill source, every configuration surface, or a desktop chat created under different workspace conditions.

Measured token and prompt-cache projections from the same experiments live in the [`Token usage and cost model`](token-usage-and-cost-model.md#rollout-observations). Those counts characterize rendered inputs and aggregate cache reuse; they do not independently prove configuration freshness.

<a id="cli-resume-experiment"></a>
## ⌨️ Controlled Codex CLI resume experiment

### 🧰 CLI setup

The experiment used Codex CLI `0.149.0`, one temporary Git repository, one project-local `AGENTS.md`, one project-local test skill, and one persisted thread resumed by successive `codex exec resume` processes. The prompts, marker strings, and rollout projections were repeated where practical.

Refresh the installed CLI version before comparing a later run:

```bash
codex --version
```

The recorded version is a time-of-experiment snapshot. A later command result supersedes it for a new run.

### 📊 CLI results

| Change before resumed turn | Rollout evidence | Observed behavior |
| --- | --- | --- |
| None; skill description did not match the prompt | Initial skill-catalog developer message contained the unrelated description | The model did not select the test skill. |
| Skill YAML `description` changed to match the same request | A new skill-catalog developer message in the same rollout contained the changed description before the response | The model selected the skill on that turn and again on a repeated turn. |
| Full skill body marker changed from `V1` to `V2` without changing metadata | Tool-call output showed a fresh read of the current `SKILL.md` body | The same continued thread returned `V2` on the changed turn and its repeat. |
| Project `AGENTS.md` marker changed from `V1` to `V2` | A new user-role instruction message carrying `V2` appeared in the same rollout before the prompt; no file-read tool call occurred | The continued thread returned `V2` on the changed turn and its repeat. |

### 🧭 CLI conclusion and limitations

In this CLI version, a **resume boundary** refreshed changed project instructions and changed skill metadata, while selected skill bodies were read from disk at invocation. A durable thread identifier alone therefore did not imply frozen configuration.

This does not establish behavior for a continuously open interactive TUI process, the Codex desktop app, another CLI version, another skill source, or every configuration surface. The observation must not be promoted into a general hot-reload guarantee.

### 🔁 CLI reproduction

To reproduce the test without touching canonical or generated user files:

1. Create a temporary Git repository with a project `AGENTS.md` marker and `.agents/skills/<probe>/SKILL.md` containing a nonmatching description and a body marker.
2. Start `codex exec --json` in that directory and record its thread identifier.
3. Change only the skill description, resume the same thread with an equivalent prompt, and inspect the explicit rollout JSONL for the new catalog message and token snapshot.
4. Change only the body marker, resume again, and inspect the skill-read tool output.
5. Change only the `AGENTS.md` marker, resume again without allowing a file-read step, and inspect the new injected instruction message.
6. Repeat the unchanged variants, record the CLI version and exact lifecycle boundary, then remove the temporary repository and test thread.

<a id="desktop-experiment"></a>
## 🖥️ Controlled Codex desktop experiment

### 🧰 Desktop setup

The experiment kept one desktop chat and one running app session open across all measured variants. The tested app identified itself as bundle `com.openai.codex`, desktop version `26.924.20706` (build `11431`), backed by Codex CLI `0.149.0`.

Refresh these time-bound values before comparing a later run:

```bash
plutil -p /Applications/ChatGPT.app/Contents/Info.plist \
  | rg 'CFBundle(Identifier|ShortVersionString|Version)'
codex --version
```

The recorded values are a time-of-experiment snapshot. Later command results supersede them for a new run.

The test used one projectless workspace, one `.agents/skills/isolated-procedure-zeta/SKILL.md`, distinctive `V1`/`V2` markers, and rollout projections from the same thread identifier. The app remained running; no chat recreation, app restart, compaction, or model-setting change occurred during the measured matrix.

### 📊 Desktop results

| Change in the open desktop chat | Rollout evidence | Observed behavior |
| --- | --- | --- |
| Baseline skill description did not match `comet orchard protocol` | The injected skill-catalog developer message contained the unrelated description; no skill-body read occurred | The model returned `NO_SKILL_SELECTED`. |
| Only the on-disk YAML `description` changed to match; same prompt repeated twice | No new catalog message appeared, and the rollout retained only the original unrelated description | Both turns returned `NO_SKILL_SELECTED`; this running desktop chat did not refresh the catalog entry. |
| Only the skill body changed from `V1` to `V2`; the skill was invoked explicitly | The tool call reread the same `SKILL.md` path and its output contained `BODY_MARKER_SKILL_V2` | The model returned `SKILL_BODY_V2`; the unchanged repeat reread and returned `V2` again. |
| A project `AGENTS.md` was added after chat creation and later changed from `V1` to `V2` | Neither marker appeared as an injected instruction record, and the no-file-read probe returned `NO_PROJECT_AGENTS_MARKER` | This setup did not dynamically discover the post-creation project instruction file. It does **not** prove whether an `AGENTS.md` present at desktop-chat creation would hot-reload from `V1` to `V2`. |

### 🧭 Desktop conclusion and limitations

For this desktop version and projectless-chat setup, skill metadata was snapshotted when first injected, while an explicitly selected local skill body was read from disk at invocation. The project-instruction result is narrower: the setup did not dynamically discover a project `AGENTS.md` added after chat creation, but it did not test hot reload of a file that existed when the chat was created.

This differs from the CLI `exec resume` experiment, where each resume process refreshed metadata and project instructions. Neither result transfers to another lifecycle without testing the exact boundary.

### 🔁 Desktop reproduction

To recheck a future desktop build:

1. Record the app and CLI versions, workspace scope, thread identifier, model setting, and app/session lifecycle.
2. Keep the app process and chat constant across the measured variants.
3. Pre-provision the workspace before chat creation when testing instruction-file refresh rather than post-creation discovery.
4. Change one surface at a time: skill metadata, selected skill body, or project instruction content.
5. Repeat each unchanged variant.
6. Inspect targeted rollout projections for catalog messages, injected instruction records, file-read outputs, and per-inference usage.

<a id="cross-surface-interpretation"></a>
## 🔀 Cross-surface interpretation

| Lifecycle boundary | Skill metadata | Selected skill body | Project instructions |
| --- | --- | --- | --- |
| Successive Codex CLI `exec resume` processes, CLI `0.149.0` | Refreshed in the observed resumed turn | Read from disk at invocation | Refreshed in the observed resumed turn |
| One continuously open desktop chat, app `26.924.20706` / CLI `0.149.0`, projectless setup | Remained at the originally injected value | Read from disk at explicit invocation | A file added after chat creation was not dynamically discovered; pre-existing-file hot reload was not tested |

The table is a dated comparison of the recorded experiments, not a compatibility promise. Re-run the applicable procedure for the exact surface and lifecycle under diagnosis.

<a id="reproduction"></a>
## 🔁 Reproduction procedure

Use this evidence discipline for either surface:

1. State the exact hypothesis and lifecycle boundary.
2. Record surface, version, configuration, workspace scope, thread identifier, and every resume, restart, reload, or new-chat event.
3. Establish a baseline with distinctive markers.
4. Change one surface at a time and repeat unchanged controls.
5. Inspect the smallest safe projection of one explicit rollout file; distinguish injected messages from file-read tool output.
6. Compare a fresh session with the exact continued-session lifecycle when freshness remains uncertain.
7. Report the tested boundary and unresolved cases explicitly.

Rollout files can contain prompts, tool output, and other sensitive context. Keep inspection local, target a known file, and return only the fields needed for the diagnosis. A final answer alone cannot distinguish remembered content, context injection, metadata selection, and a fresh file read.
