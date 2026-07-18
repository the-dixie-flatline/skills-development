---
family: grok
scope: prompt
versions:
  - grok-4.5
  - grok-4.3
  - grok-build-0.1
retrieved: 2026-07-19
primary_sources:
  - https://docs.x.ai/developers/models
  - https://docs.x.ai/developers/models/grok-4.5
  - https://docs.x.ai/developers/pricing
  - https://docs.x.ai/developers/model-capabilities/text/reasoning
  - https://docs.x.ai/developers/migration/may-15-retirement
  - https://docs.x.ai/developers/tools/web-search
  - https://media.x.ai/v1/website/card-7f81d41b.pdf
maturity_note: |
  grok-4.5 is xAI's current flagship (500,000-token context), superseding
  grok-4.3. Its reasoning defaults are inverted relative to the prior
  generation: `reasoning_effort` accepts {low, medium, high}, defaults to
  `high`, and `none` has been removed — reasoning cannot be disabled.
  grok-4.3 remains live as prior-gen and is the counterintuitive holder of
  the LARGER context window (1,000,000 tokens, twice grok-4.5); it keeps the
  {none, low, medium, high} set with default `low` and a working non-reasoning
  mode. grok-build-0.1 is a fast coding model (256k). Effective 2026-05-15,
  the prior fast / 4.x / 3 slugs were retired; the migration page still routes
  the six retired text/reasoning slugs to grok-4.3 (not grok-4.5), a vendor
  self-inconsistency. grok-4.5 knowledge cutoff is documented inconsistently
  (models page: Feb 1 2026; system card: Jan 2026).
---

# Grok — Prompt-Layer Reference

Portable prompting guidance for the current Grok generation. API-layer detail (parameter shapes, headers, endpoint specifics) lives in `grok-prompt-api.md`.

Reasoning semantics differ by model generation. grok-4.5 (flagship) and grok-4.3 (prior gen) are NOT interchangeable on the `reasoning_effort` axis; every reasoning statement below is scoped to its model.

## 1. Model Selection

The current text lineup is the flagship `grok-4.5`, the prior-gen `grok-4.3` (still live), and the coding-focused `grok-build-0.1`. The prior reasoning / non-reasoning pairs and fast slugs were retired 2026-05-15 (see §7).

| Model            | Context | Reasoning?                       | Default effort | Standard tier (prompt ≤200K) | Long-context tier (prompt >200K) | Notes                                                                 |
|------------------|---------|----------------------------------|----------------|------------------------------|----------------------------------|----------------------------------------------------------------------|
| `grok-4.5`       | 500K    | always on ({low, medium, high})  | `high`         | $2.00 in / $6.00 out per 1M  | $4.00 in / $12.00 out per 1M     | Flagship. `none` removed; reasoning cannot be disabled. Cached-input price: gap (§8) |
| `grok-4.3`       | 1M      | controllable ({none…high})       | `low`          | $1.25 in / $2.50 out per 1M  | $2.50 in / $5.00 out per 1M      | Prior gen, still live. LARGER context than the flagship; retains non-reasoning `none` mode; cached input $0.20/1M |
| `grok-build-0.1` | 256k    | n/a                              | n/a            | (not documented, §8)         | (not documented, §8)             | Fast coding model for agentic coding workflows                        |

[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models/grok-4.5, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/pricing, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models/grok-4.3, retrieved 2026-06-01]

Additional current lineup models (1M context) exist beyond this trio: `grok-4.20-0309-reasoning`, `grok-4.20-0309-non-reasoning`, `grok-4.20-multi-agent-0309`. Aliases: `grok-4.5-latest` and `grok-build-latest` resolve to grok-4.5; the convention is `<name>` = latest stable, `<name>-latest` = latest (new features), `<name>-<date>` = pinned release.
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models/grok-4.5, retrieved 2026-07-19]

### Counterintuitive context regression: grok-4.5 has HALF the context of grok-4.3

Do not assume the newer flagship dominates on every axis. grok-4.5 is 500K context; grok-4.3, the model it supersedes, is 1M. If a task needs more than 500K tokens of context, grok-4.5 cannot hold it and grok-4.3 is the correct choice despite being prior-gen.
[source: https://docs.x.ai/developers/models/grok-4.5, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]

### Selection rules

- **Default flagship text / agentic tool use**: `grok-4.5`. Reasoning is always on; `reasoning_effort` accepts {low, medium, high} and defaults to `high`. For low-latency work drop to `low` — `none` is unavailable on grok-4.5 (reasoning cannot be disabled). See `grok-prompt-api.md` §4.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
- **When you need >500K context, or an actual non-reasoning response**: `grok-4.3`. Its `reasoning_effort` set is {none, low, medium, high}, default `low`; set `none` for a non-reasoning, lower-latency response. This non-reasoning path exists ONLY on grok-4.3 (and other prior-gen slugs), not on the flagship. [applies-to: grok-4.3]
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
- **Agentic coding workflows**: `grok-build-0.1`, trained for fast coding, 256k context.
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]
- **Knowledge cutoff (grok-4.5)** is documented inconsistently across two live authoritative surfaces; report both, do not average. [disputed: models page states February 1, 2026; system card states a pretraining cutoff of January 2026] For strict temporal grounding, default to the earlier January 2026 boundary and rely on the server-side `web_search` / X Search tools for anything more recent. The cutoff is not documented for grok-4.3 (§8).
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]
[source: https://media.x.ai/v1/website/card-7f81d41b.pdf, retrieved 2026-07-19]

## 2. Prompt Structure Conventions

Grok's API is **OpenAI-compatible** at the wire level — messages, tools, and tool_choice follow the OpenAI Chat Completions conventions. Prompts portable from OpenAI Chat Completions generally work on Grok, with two caveats:

- **`reasoning_effort` maps, but the value set and default are model-specific.** grok-4.5: {low, medium, high}, default `high`, no `none`. grok-4.3: {none, low, medium, high}, default `low`, `none` disables reasoning. A wrapper that hard-codes a portable `none` for Grok is stale for the flagship. Note some sampling fields are incompatible with reasoning (see `grok-prompt-api.md` §3).
- **Some parameter defaults and features differ** (cache-hit routing, built-in tools, custom headers). Covered in `grok-prompt-api.md`.

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/tools/web-search, retrieved 2026-06-01]

## 3. Instruction Patterns

### Reasoning control is generation-specific

- **grok-4.5**: `reasoning_effort` accepts {low, medium, high} and defaults to `high` when omitted. There is no `none`; reasoning cannot be disabled. To reduce latency, set `low`. [applies-to: grok-4.5]
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
- **grok-4.3**: `reasoning_effort` accepts {none, low, medium, high} and defaults to `low`. `none` means no reasoning occurs (a lower-latency, non-reasoning response). [applies-to: grok-4.3]
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

### Tune effort to workload shape

Raise `reasoning_effort` for multi-step problems. To cut latency: on grok-4.5 step down to `low` (its floor); on grok-4.3 set `none` for quick classification, extraction, or rewriting where latency matters more than deliberation.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

### Real-time data access via server-side search tools

A differentiating feature of the Grok API is the server-side Web Search / X Search tools (`web_search`) that execute on xAI servers and return `response.citations`. For tasks requiring current events or live X state, declare these tools rather than passing scraped content in the prompt — the tools handle retrieval and attribution. (The consumer "DeepSearch" agent on grok.com / X is a separate product surface, not a developer-API feature.)
[source: https://docs.x.ai/developers/tools/web-search, retrieved 2026-06-01]

## 4. Context Window Practical Guidance

- **grok-4.5: 500,000 tokens. grok-4.3: 1,000,000 tokens. grok-build-0.1: 256k.** The flagship has the smaller window (see §1 regression callout).
[source: https://docs.x.ai/developers/models/grok-4.5, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]

- **200K whole-request repricing threshold (cost cliff).** For both grok-4.5 and grok-4.3, a request whose prompt reaches 200K tokens is billed at the higher long-context rate for ALL tokens in the request, not just the tokens above 200K. Crossing 200K roughly doubles the entire request cost (grok-4.5: $2/$6 → $4/$12; grok-4.3: $1.25/$2.50 → $2.50/$5.00). Batch or trim prompts to stay under 200K where feasible.
[source: https://docs.x.ai/developers/pricing, retrieved 2026-07-19]

- **Automatic prompt caching** is always on. grok-4.3 cached input is priced at $0.20/1M; the grok-4.5 cached-input rate is not confirmed on the pricing page (§8).
[source: https://docs.x.ai/developers/models/grok-4.3, retrieved 2026-06-01]

- **`x-grok-conv-id` HTTP header (Chat Completions) / `prompt_cache_key` (Responses API)** are the optional levers to maximize cache-hit rate by pinning a conversation to the same cache node; use them for multi-turn sessions to preserve the cached prefix across requests. See `grok-prompt-api.md` §7.
[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-07-19]

## 5. Multimodal Conventions

- **Image generation** is a separate model family (`grok-imagine-*`), not an inline capability. Note `grok-imagine-image-pro` was retired 2026-05-15 and redirects to `grok-imagine-image-quality` (see §7).

[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/migration/may-15-retirement, retrieved 2026-07-19]

Exact image-placement conventions within the OpenAI-compatible content-part array were not captured in this retrieval pass; see `grok-prompt-api.md` §10 (Gaps) for flagged items.

## 6. Behavioral Quirks

- **grok-4.5 reasoning cannot be turned off.** Omitting `reasoning_effort` yields `high` effort, not a fast path; there is no `none`. A caller expecting a non-reasoning response from the flagship will not get one. [applies-to: grok-4.5]
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

- **grok-4.3 `reasoning_effort` defaults to `low`, not off.** Omitting the field still incurs low-effort reasoning. Set `reasoning_effort: "none"` explicitly for a non-reasoning response. [applies-to: grok-4.3]
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

- **Some sampling fields are incompatible with reasoning.** `presence_penalty`, `frequency_penalty`, and `stop` return an error on reasoning models. Because grok-4.5 is a reasoning model on every request (reasoning cannot be disabled), these fields are rejected on every grok-4.5 call; on grok-4.3 they are rejected whenever `reasoning_effort` is not `none`. `logprobs` / `top_logprobs` are silently ignored on grok-4.20 and newer. See `grok-prompt-api.md` §3.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

- **Streaming function calls behavior:** see `grok-prompt-api.md` §5 for the streaming reasoning and tool-call event shapes.

- **grok-4.5 knowledge cutoff is documented inconsistently** (models page Feb 1 2026 vs system card Jan 2026; §1). For anything after these boundaries, depend on the server-side search tools rather than the model's parametric memory. The cutoff is not documented for grok-4.3.
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]
[source: https://media.x.ai/v1/website/card-7f81d41b.pdf, retrieved 2026-07-19]

- **Caching is best-effort.** Use the optional `x-grok-conv-id` header (or `prompt_cache_key` on the Responses API) to increase the probability of hitting the same cache, but do not assume caching is free-and-perpetual. See `grok-prompt-api.md` §7.
[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-07-19]

## 7. Anti-Patterns

- **Do not send `reasoning_effort: "none"` to grok-4.5.** The value was removed; reasoning cannot be disabled on the flagship. To reduce latency, use `low` (its floor). `none` remains valid ONLY on grok-4.3 and other prior-gen slugs.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

- **Do not assume omitting `reasoning_effort` disables reasoning.** grok-4.5 defaults to `high`; grok-4.3 defaults to `low`. Neither is "off". On grok-4.3, send `reasoning_effort: "none"` for a non-reasoning response; grok-4.5 has no such path.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

- **Do not use `reasoning_effort` values outside the model's set.** grok-4.5: {low, medium, high} only (no `none`). grok-4.3: {none, low, medium, high}. There is no `xhigh` value on Grok — the system card labels competitor models `(xhigh)` but Grok always `(high)`.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
[source: https://media.x.ai/v1/website/card-7f81d41b.pdf, retrieved 2026-07-19]

- **Do not send `presence_penalty`, `frequency_penalty`, or `stop` to a reasoning model.** They return an error — on every grok-4.5 request, and on grok-4.3 whenever effort is not `none`. See `grok-prompt-api.md` §3.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

- **Do not assume grok-4.5 ≥ grok-4.3 on context.** grok-4.5 is 500K; grok-4.3 is 1M. For >500K-token contexts, target grok-4.3.
[source: https://docs.x.ai/developers/models/grok-4.5, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]

- **Do not cross the 200K prompt threshold casually.** It reprices the whole request (input and output) at the long-context tier, not just the excess tokens. See §4.
[source: https://docs.x.ai/developers/pricing, retrieved 2026-07-19]

- **Do not target retired slugs in new work.** `grok-4-1-fast-*`, `grok-4-fast-*`, `grok-4-0709`, `grok-code-fast-1`, `grok-3`, and `grok-imagine-image-pro` were retired 2026-05-15. The six text/reasoning slugs auto-redirect to `grok-4.3` (not the grok-4.5 flagship); `grok-code-fast-1` redirects to `grok-build-0.1`; `grok-imagine-image-pro` redirects to `grok-imagine-image-quality`. Pin `grok-4.5`, `grok-4.3`, or `grok-build-0.1` directly. See `grok-prompt-api.md` §9.
[source: https://docs.x.ai/developers/migration/may-15-retirement, retrieved 2026-07-19]

- **Do not rely on parametric knowledge for post-cutoff facts.** Use the server-side Web Search / X Search tools for current-events grounding.
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]

- **Do not ignore the `x-grok-conv-id` header for multi-turn sessions.** Without it, cache-node routing is best-effort — you may pay full input pricing on a cache that exists but was routed elsewhere.
[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-07-19]

## 8. Gaps

- **grok-4.5 cached-input pricing** is not documented at https://docs.x.ai/developers/pricing (base tiers and the 200K threshold rule are published, but the cached line items are not), checked 2026-07-19. Do not assume a value.
- **grok-build-0.1 pricing** (base input/output and cached) is not documented at https://docs.x.ai/developers/pricing, checked 2026-07-19.
- **Max output tokens per model** is not documented at https://docs.x.ai/developers/models/grok-4.5 (only the 500K context window is stated); confirmed absence, checked 2026-07-19.
- **grok-4.3-specific knowledge cutoff** is not documented at https://docs.x.ai/developers/models, checked 2026-07-19; only the grok-4.5 Feb 1 2026 / Jan 2026 statements are published, and they conflict (§1).
- **Built-in tool parameter shapes** (`web_search`, X Search, code execution, collections) beyond type names were not pulled in depth.
- **Structured Outputs parameter shape** was not targeted in this retrieval pass beyond confirmation of support.
- **Deep / agentic research:** Grok's developer surface is the server-side web-search tool-use shape (returning `response.citations`), not a managed async research agent. For the cross-family async-agent comparison see `resources/deep-research-agents.md`. The consumer "DeepSearch" agent (grok.com / X) is not a developer-API feature.
