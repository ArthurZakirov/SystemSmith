# Token Usage and Cost Model

Use this reference to analyze context-window consumption, token usage, prompt-cache behavior, API cost, or ChatGPT/Codex allowance and credit accounting. Do not infer any of them from stored rollout-record counts alone.

## Five diagnostic layers

| Layer | What it establishes |
| --- | --- |
| Stored rollout or transcript records | Append-only persistence and diagnostic evidence. A block can be stored once yet participate in many later model calls. |
| Active conversation/context items | Items retained after pruning, reconstruction, or compaction and available to build the next request. |
| Rendered input for one inference | The complete input supplied to one model call, including retained items and harness-provided surfaces that may not recur as transcript messages. |
| Usage accounting | Per-inference `input_tokens`, `cached_tokens`, output tokens, and reasoning tokens. These fields support conclusions about processed usage and aggregate cache reuse. |
| Product accounting | The applicable API price, ChatGPT/Codex subscription allowance, credit balance, workspace rate card, or other product-specific accounting surface. |

Never infer token cost, context-window occupancy, repeated model exposure, cache treatment, or instruction robustness merely from how many times a block appears in stored logs. Repeated skill-catalog or runtime records can reflect reinjection or updates without proving full-price processing on every occurrence or stronger instruction following.

## What reaches rendered input

Codex CLI enumerates applicable `AGENTS.md` files and injects each discovered chunk near the top of conversation history as a separate user-role message, before the user prompt, in root-to-leaf order. This describes discovery and injection, not physical rereading before every inference. Once retained in active history, one stored instruction block can appear in many later rendered inputs. See OpenAI's [Codex model guidance](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.3-codex#using-agentsmd).

Skills use progressive disclosure. Initial context contains compact metadata: name and description, plus the path for local Responses API skills. The model selects from that metadata, then reads the chosen `SKILL.md`; supporting references, scripts, templates, and assets remain on-demand. See the [Skills API guide](https://developers.openai.com/api/docs/guides/tools-skills) and [plugin Skills concepts](https://developers.openai.com/plugins/concepts/skills).

Tool descriptions can arrive through request tool definitions rather than repeated transcript messages. OpenAI's [function-calling token-usage guidance](https://developers.openai.com/api/docs/guides/function-calling#token-usage) states that callable definitions count against the context limit and as input tokens. The [token-counting guide](https://developers.openai.com/api/docs/guides/token-counting) likewise includes tool definitions and request-structure formatting in the exact input count.

## Repeated input and prompt caching

Without caching, a stable block of `L` tokens retained across `N` model calls contributes approximately:

`L × N` input tokens

Prompt caching reuses computation for an unchanged prefix. Cached tokens remain input tokens, occupy context-window space, and count toward rate limits; a cache hit reduces latency and the applicable input rate rather than removing those tokens from the request.

With one full-price processing followed by cached reads at multiplier `r`, a simplified price-equivalent is:

`L + (N − 1) × r × L`

Every call still occupies `L` context-window tokens. When a model uses a distinct cache-write multiplier `w`, replace the first `L` with `w × L`. For current GPT-5.6+ API mechanics, reads are documented at `0.1×` and writes at `1.25×`. These are current API pricing mechanics, not a universal Codex-subscription billing promise. See [Prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching).

## Compaction

Compaction replaces older active history with a smaller opaque compaction item plus retained items. This can reduce later rendered inputs while changing the prompt prefix enough to reduce cache reuse immediately afterward. Treat the returned compacted window as the canonical next context rather than inferring its contents from older stored records. See [Compaction](https://developers.openai.com/api/docs/guides/compaction).

## Product accounting

API token prices, ChatGPT plan allowances, Codex credits, workspace rate cards, and internal product meters are separate accounting surfaces. Signing into Codex with ChatGPT uses the applicable ChatGPT plan's usage and billing, while using an API key uses API pricing. Check the live account surface before concluding cost or remaining capacity:

- [Using Codex with your ChatGPT plan](https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan)
- [Using credits for flexible usage](https://help.openai.com/en/articles/12642688-using-credits-for-flexible-usage-in-chatgpt-personal-plans)
- [ChatGPT Work and Codex](https://help.openai.com/en/articles/20001275-chatgpt-work-and-codex)

## Practical diagnostic sequence

Use one explicit rollout and avoid recursively dumping full logs:

1. Count stored records by category without returning their full bodies.
2. Identify the active replacement state after any compaction.
3. Inspect per-inference token snapshots instead of treating transcript records as model calls.
4. Use `input_tokens`, `cached_tokens`, output-token, and reasoning-token fields to characterize usage.
5. Consult the applicable ChatGPT/Codex allowance, credit, workspace rate card, or API pricing view before concluding monetary or quota cost.

## Local rollout observation

The following evidence is version- and session-specific, not a universal ratio:

| Observation | Measured value |
| --- | ---: |
| Explicit `AGENTS.md` records | 1 |
| Additional analyzer-classified skill-catalog/runtime blocks | 5 |
| Token-count snapshots | 290 |
| Explicit `AGENTS.md` size | Approximately 19,812 characters / 4,953 estimated tokens |
| Five additional blocks | Approximately 112,454 characters / 28,113 estimated tokens |
| Latest exact inference input | 158,680 tokens |
| Reported cached input in that inference | 158,208 tokens |
| Remaining non-cached input | 472 tokens |

Record count and model-call snapshot count clearly differ. The aggregate cache report does not attribute cached tokens to individual blocks, so it cannot establish per-block cache treatment or cost.

## Optimization principles

- Keep compact, broadly applicable invariants in `AGENTS.md`.
- Put concise trigger conditions in skill metadata.
- Keep detailed procedures in selected skill bodies.
- Load deeper references and assets only when needed.
- Keep tool descriptions concise and defer rarely used tools when the runtime supports tool search.
- Preserve stable prefixes when cache reuse matters, but evaluate compaction by total rendered input and task quality rather than cache-hit rate alone.
