---
family: grok
scope: prompt
versions:
  - grok-4.3
  - grok-build-0.1
retrieved: 2026-06-01
primary_sources:
  - https://docs.x.ai/developers/models
  - https://docs.x.ai/developers/models/grok-4.3
  - https://docs.x.ai/developers/model-capabilities/text/reasoning
  - https://docs.x.ai/developers/migration/may-15-retirement
  - https://docs.x.ai/developers/tools/web-search
maturity_note: |
  grok-4.3 is xAI's current flagship (1,000,000-token context, strong
  agentic tool calling, supports a non-reasoning mode). Reasoning is
  controlled by the `reasoning_effort` parameter ({none, low, medium, high},
  default "low"), a normal reasoning control rather than a per-variant
  semantic. grok-build-0.1 is a fast coding model for agentic coding
  workflows. Effective 2026-05-15, the prior fast / 4.x / 3 slugs
  (grok-4-1-fast-*, grok-4-fast-*, grok-4-0709, grok-code-fast-1, grok-3,
  grok-imagine-image-pro) were retired and auto-redirect to grok-4.3 (or
  grok-build-0.1 for grok-code-fast-1). Knowledge cutoff: November 2024 for
  Grok 3 and Grok 4; not explicitly documented for grok-4.3.
---

# Grok — Prompt-Layer Reference

Portable prompting guidance for the current Grok generation. API-layer detail (parameter shapes, headers, endpoint specifics) lives in `grok-prompt-api.md`.

## 1. Model Selection

The current text lineup is the flagship `grok-4.3` plus the coding-focused `grok-build-0.1`. The prior reasoning / non-reasoning pairs and fast slugs were retired 2026-05-15 (see §7).

| Model            | Context | Reasoning?              | Cached input per MTok | Notes                                                              |
|------------------|---------|-------------------------|-----------------------|--------------------------------------------------------------------|
| `grok-4.3`       | 1M      | yes (controllable)      | $0.20                 | Flagship. Strong agentic tool calling, minimal hallucinations; supports non-reasoning mode |
| `grok-build-0.1` | 256k    | n/a                     | (not quoted)          | Fast coding model for agentic coding workflows. Playground coming soon |

[source: docs.x.ai/developers/models, retrieved 2026-06-01]
[source: docs.x.ai/developers/models/grok-4.3, retrieved 2026-06-01]

### Selection rules

- **Flagship text / agentic tool use**: `grok-4.3`. Reasoning depth is controlled by `reasoning_effort` ({none, low, medium, high}, default `low`); set `none` for a non-reasoning, lower-latency response. See `grok-prompt-api.md` §4.
- **Agentic coding workflows**: `grok-build-0.1`, trained for fast coding, 256k context.
- **Knowledge cutoff**: November 2024 [applies-to: grok-3, grok-4] — do not rely on the model for more recent world state without external grounding (use the server-side `web_search` / X Search tools). The cutoff is not explicitly documented for grok-4.3.
[source: docs.x.ai/developers/models, retrieved 2026-06-01]

## 2. Prompt Structure Conventions

Grok's API is **OpenAI-compatible** at the wire level — messages, tools, and tool_choice follow the OpenAI Chat Completions conventions. Prompts portable from OpenAI Chat Completions generally work on Grok, with two caveats:

- **`reasoning_effort` maps cleanly.** `grok-4.3` accepts `reasoning_effort` with values {none, low, medium, high} (default `low`); `none` disables reasoning. This is a normal reasoning control. Note some sampling fields are incompatible with reasoning (see `grok-prompt-api.md` §3).
- **Some parameter defaults and features differ** (cache-hit routing, built-in tools, custom headers). Covered in `grok-prompt-api.md`.

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]
[source: docs.x.ai/developers/tools/web-search, retrieved 2026-06-01]

## 3. Instruction Patterns

### Reasoning is controllable via `reasoning_effort`

`grok-4.3` accepts `reasoning_effort` with values {none, low, medium, high}; it defaults to `"low"`. `"none"` means no reasoning occurs (a lower-latency, non-reasoning response). This is a normal reasoning control, not a per-variant semantic.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

### Tune effort to workload shape

Raise `reasoning_effort` for multi-step problems; set `none` for quick classification, extraction, or rewriting where latency matters more than deliberation.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

### Real-time data access via server-side search tools

A differentiating feature of the Grok API is the server-side Web Search / X Search tools (`web_search`) that execute on xAI servers and return `response.citations`. For tasks requiring current events or live X state, declare these tools rather than passing scraped content in the prompt — the tools handle retrieval and attribution. (The consumer "DeepSearch" agent on grok.com / X is a separate product surface, not a developer-API feature.)
[source: docs.x.ai/developers/tools/web-search, retrieved 2026-06-01]
[source: https://x.ai/news/grok-3, retrieved 2026-06-01]

## 4. Context Window Practical Guidance

- **1,000,000 tokens** on `grok-4.3`; `grok-build-0.1` is 256k.
- **Automatic prompt caching** is always on; `grok-4.3` cached input is priced at $0.20/MTok.
- **`x-grok-conv-id` HTTP header** is the optional lever to maximize cache hit rate by pinning a conversation to the same cache node; use it for multi-turn sessions to preserve the cached prefix across requests.

[source: docs.x.ai/developers/models, retrieved 2026-06-01]
[source: docs.x.ai/developers/models/grok-4.3, retrieved 2026-06-01]

No explicit per-model "long-input surcharge" (as with OpenAI's 272K threshold on GPT-5.4) is documented in the retrieved sources.

## 5. Multimodal Conventions

- **Image generation** is a separate model family (`grok-imagine-*`), not an inline capability. Note `grok-imagine-image-pro` was retired 2026-05-15 (see §7).

[source: docs.x.ai/developers/models, retrieved 2026-06-01]

Exact image-placement conventions within the OpenAI-compatible content-part array were not captured in this retrieval pass; see `grok-prompt-api.md` §10 (Gaps) for flagged items.

## 6. Behavioral Quirks

- **`reasoning_effort` defaults to `low`, not off.** On `grok-4.3`, omitting the field still incurs low-effort reasoning. Set `reasoning_effort: "none"` explicitly for a non-reasoning response.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

- **Some sampling fields are incompatible with reasoning.** `presence_penalty`, `frequency_penalty`, and `stop` return an error on reasoning models; `logprobs` / `top_logprobs` are silently ignored on grok-4.20 and newer. See `grok-prompt-api.md` §3.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

- **Streaming function calls behavior:** see `grok-prompt-api.md` §5 for the streaming reasoning and tool-call event shapes.

- **Knowledge cutoff is November 2024** [applies-to: grok-3, grok-4] — not explicitly documented for grok-4.3. For anything after November 2024, depend on the server-side search tools rather than the model's parametric memory.
[source: docs.x.ai/developers/models, retrieved 2026-06-01]

- **Caching is best-effort.** Use the optional `x-grok-conv-id` header to increase the probability of hitting the same cache, but do not assume caching is free-and-perpetual. See `grok-prompt-api.md` §7.
[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-06-01]

## 7. Anti-Patterns

- **Do not assume omitting `reasoning_effort` disables reasoning.** It defaults to `low` on `grok-4.3`. Send `reasoning_effort: "none"` for a non-reasoning response.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

- **Do not use values beyond `none`/`low`/`medium`/`high` for `reasoning_effort`.** Only these four are documented for grok-4.3.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

- **Do not send `presence_penalty`, `frequency_penalty`, or `stop` to a reasoning model.** They return an error. See `grok-prompt-api.md` §3.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

- **Do not target retired slugs in new work.** `grok-4-1-fast-*`, `grok-4-fast-*`, `grok-4-0709`, `grok-code-fast-1`, `grok-3`, and `grok-imagine-image-pro` were retired 2026-05-15 and auto-redirect to `grok-4.3` (or `grok-build-0.1` for `grok-code-fast-1`). Pin `grok-4.3` or `grok-build-0.1` directly. See §7 and `grok-prompt-api.md` §9.
[source: https://docs.x.ai/developers/migration/may-15-retirement, retrieved 2026-06-01]

- **Do not rely on parametric knowledge for post-November-2024 facts.** Use the server-side Web Search / X Search tools for current-events grounding.
[source: docs.x.ai/developers/models, retrieved 2026-06-01]

- **Do not ignore the `x-grok-conv-id` header for multi-turn sessions.** Without it, cache-node routing is best-effort — you may pay full input pricing on a cache that exists but was routed elsewhere.
[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-06-01]

## 8. Gaps

- **Max output tokens per model** is not quoted in the retrieved model-lineup excerpt.
- **`grok-4.3`-specific knowledge cutoff** is not documented; only the Grok 3 / Grok 4 November 2024 statement is published.
- **`grok-build-0.1` cached/input/output pricing** is not quoted; the Playground is "coming soon".
- **Built-in tool parameter shapes** (`web_search`, X Search, code execution, collections) beyond type names were not pulled in depth.
- **Structured Outputs parameter shape** was not targeted in this retrieval pass beyond confirmation of support.
- **Deep / agentic research:** Grok's developer surface is the server-side web-search tool-use shape (returning `response.citations`), not a managed async research agent. For the cross-family async-agent comparison see `resources/deep-research-agents.md`. The consumer "DeepSearch" agent (grok.com / X) is not a developer-API feature. "DeeperSearch" is not asserted here (no Tier-1 source).
