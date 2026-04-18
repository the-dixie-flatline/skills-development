---
family: grok
scope: prompt
versions:
  - grok-4.20-0309-reasoning
  - grok-4.20-0309-non-reasoning
  - grok-4.20-multi-agent-0309
  - grok-4-1-fast-reasoning
  - grok-4-1-fast-non-reasoning
retrieved: 2026-04-19
primary_sources:
  - https://docs.x.ai/developers/models
  - https://docs.x.ai/docs/guides/reasoning
  - https://docs.x.ai/docs/guides/function-calling
  - https://docs.x.ai/developers/advanced-api-usage/prompt-caching/how-it-works
  - https://docs.x.ai/developers/release-notes
maturity_note: |
  Grok 4.20 is xAI's current flagship, paired as reasoning / non-reasoning
  variants at the same price point, plus a separate multi-agent variant.
  Grok 4.1 Fast is the cost-tier peer (~10x cheaper). Reasoning on 4.20 and
  4.1 Fast is automatic — the `reasoning_effort` parameter is rejected on
  most Grok models, and on the `multi-agent` variant it controls agent count
  rather than reasoning depth (a semantic divergence from OpenAI
  conventions). Knowledge cutoff: November 2024.
---

# Grok — Prompt-Layer Reference

Portable prompting guidance for the current Grok 4.x generation. API-layer detail (parameter shapes, headers, endpoint specifics) lives in `grok-prompt-api.md`.

## 1. Model Selection

Grok's lineup is organized as reasoning / non-reasoning **pairs** at matching price points, plus specialized variants.

| Model                                      | Context | Reasoning? | Input / Cached / Output per MTok | Notes                                        |
|--------------------------------------------|---------|------------|----------------------------------|----------------------------------------------|
| `grok-4.20-0309-reasoning`                 | 2M      | yes        | $2.00 / $0.20 / $6.00            | Flagship reasoning                           |
| `grok-4.20-0309-non-reasoning`             | 2M      | no         | $2.00 / $0.20 / $6.00            | Same price; lower latency                    |
| `grok-4.20-multi-agent-0309`               | 2M      | yes        | $2.00 / $0.20 / $6.00            | `reasoning_effort` controls agent count      |
| `grok-4-1-fast-reasoning`                  | 2M      | yes        | $0.20 / $0.05 / $0.50            | ~10× cheaper, still 2M context               |
| `grok-4-1-fast-non-reasoning`              | 2M      | no         | $0.20 / $0.05 / $0.50            | Fastest / cheapest text tier                 |

[source: docs.x.ai/developers/models, retrieved 2026-04-19]

Rate limits (text + reasoning tier): **10M TPM / 1,800 RPM**.
[source: docs.x.ai/developers/models, retrieved 2026-04-19]

Specialized (not text-chat):

| Model                         | Modalities           | Pricing          |
|-------------------------------|----------------------|------------------|
| `grok-imagine-image`          | text/image → image   | $0.02/image      |
| `grok-imagine-image-pro`      | text/image → image   | $0.07/image      |
| `grok-imagine-video`          | text/image/video → video | $0.050/sec   |

[source: docs.x.ai/developers/models, retrieved 2026-04-19]

Plus Speech-to-Text and Text-to-Speech APIs (25 languages, batch + streaming support).
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]

### Selection rules

- **Text reasoning at flagship quality**: `grok-4.20-0309-reasoning`.
- **Text generation at flagship quality, low latency**: `grok-4.20-0309-non-reasoning`.
- **Cost-sensitive, still 2M context**: `grok-4-1-fast-reasoning` or `grok-4-1-fast-non-reasoning` — one order of magnitude cheaper than 4.20.
- **Agentic multi-agent reasoning**: `grok-4.20-multi-agent-0309`. See `grok-prompt-api.md` §4 for the agent-count control via `reasoning_effort`.
- **Knowledge cutoff**: November 2024 — do not rely on the model for more recent world state without external grounding (use the built-in `x_search` or `web_search` tools).

## 2. Prompt Structure Conventions

Grok's API is **OpenAI-compatible** at the wire level — messages, tools, and tool_choice follow the OpenAI Chat Completions conventions. Prompts portable from OpenAI Chat Completions generally work on Grok, with two caveats:

- **`reasoning_effort` is not portable.** It is accepted (with different semantics) only on the `multi-agent` variant. On `grok-4.20` and `grok-4-1-fast` it returns an error. See `grok-prompt-api.md` §4.
- **Some parameter defaults and features differ** (cache-hit routing, built-in tools, custom headers). Covered in `grok-prompt-api.md`.

[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

## 3. Instruction Patterns

### Reasoning is automatic on reasoning variants

On `grok-4.20-0309-reasoning` and `grok-4-1-fast-reasoning`, the model "thinks through problems step-by-step before delivering an answer" without caller configuration. There is **no `reasoning_effort` knob** for reasoning depth on these models — the model's internal scheduling handles it.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

### Prefer reasoning variants for harder tasks, non-reasoning for latency-bound work

Because the reasoning and non-reasoning variants are priced identically on 4.20 (and also identically on 4.1 Fast), the trade-off is purely latency vs quality — not cost. Pick by workload shape: reasoning for multi-step reasoning, non-reasoning for quick classification, extraction, rewriting.
[source: docs.x.ai/developers/models, retrieved 2026-04-19]

### Real-time data access via built-in search tools

The differentiating feature of the Grok API is built-in `x_search` (X / Twitter) and `web_search` tools that execute server-side. For tasks requiring current events or live X/Twitter state, declare these tools rather than passing scraped content in the prompt — the built-in tools handle retrieval and attribution.
[source: docs.x.ai/developers/tools/overview, retrieved 2026-04-19]
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]

### Multi-agent parallelism on the multi-agent variant

[applies-to: grok-4.20-multi-agent-0309]
On the `multi-agent` variant, `reasoning_effort` values map to **agent count**, not reasoning depth:

- `reasoning_effort: "low"` or `"medium"` → **4 agents** in parallel.
- `reasoning_effort: "high"` or `"xhigh"` → **16 agents** in parallel.

This is a deliberate semantic reuse — think of it as fan-out control. Raising effort raises parallel agent count (with cost and latency implications), not per-agent thinking depth.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]
[testable: id=grok.multi-agent-effort-controls-count.v1, expected=grok-4.20-multi-agent-0309 with reasoning_effort="high" produces more parallel agent traces than reasoning_effort="low"]

## 4. Context Window Practical Guidance

- **2M tokens** on every current text and reasoning variant (4.20 pair, 4.1 Fast pair, multi-agent variant). Largest of any closed-API family reviewed in this library tier.
- **Automatic prompt caching** is always on; the cached-input rate is 10× cheaper than the base rate (e.g. `grok-4.20` at $2.00/MTok input vs $0.20/MTok cached), a ~90% discount.
- **`x-grok-conv-id` HTTP header** maximizes cache hit rate by pinning a conversation to the same cache node; use it for multi-turn sessions to preserve the cached prefix across requests.

[source: docs.x.ai/developers/models, retrieved 2026-04-19]
[source: docs.x.ai/developers/advanced-api-usage/prompt-caching/how-it-works, retrieved 2026-04-19]

No explicit per-model "long-input surcharge" (as with OpenAI's 272K threshold on GPT-5.4) is documented in the retrieved sources.

## 5. Multimodal Conventions

- **Text + image input → text output** is supported on all current 4.20 and 4.1 Fast variants.
- **Image generation** is a separate model family (`grok-imagine-*`), not an inline capability.
- **Video generation** is a separate model (`grok-imagine-video`).
- **Audio input / output** runs via dedicated Speech-to-Text and Text-to-Speech APIs, not as content parts on the chat models.

[source: docs.x.ai/developers/models, retrieved 2026-04-19]
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]

Exact image-placement conventions within the OpenAI-compatible content-part array were not captured in this retrieval pass; see `grok-prompt-api.md` §10 (Gaps) for flagged items.

## 6. Behavioral Quirks

- **`reasoning_effort` is not a universal parameter.** On `grok-4.20-0309` (non-reasoning) and `grok-4-1-fast` (both variants), setting it **returns an error**, not a quiet no-op. Caller code that always sends `reasoning_effort` — common in OpenAI-compatible wrappers — will fail on Grok.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]
[testable: id=grok.reasoning-effort-rejected-non-reasoning.v1, expected=request to grok-4.20-0309-non-reasoning with reasoning_effort="high" returns HTTP 4xx]

- **On `multi-agent`, `reasoning_effort` means agent count.** If you're piping OpenAI prompts through a wrapper, a caller setting `reasoning_effort: "high"` to mean "think harder" will instead spawn 16 agents. Validate effort usage per model.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

- **Reasoning tokens are billed as part of total consumption** — not separated. `response.usage.reasoning_tokens` surfaces the count, but billing is lumped into output tokens.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

- **Streaming function calls are not chunked.** "With streaming, the function call is returned in whole in a single chunk, not streamed across chunks." Callers that treat every SSE event as incremental text will misparse tool calls. Accumulate by event type.
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

- **Knowledge cutoff is November 2024** — newer than Llama 4's August 2024 but older than Claude Opus 4.7, Gemini 3, and GPT-5.4. For anything after November 2024, depend on the built-in search tools rather than the model's parametric memory.
[source: docs.x.ai/developers/models, retrieved 2026-04-19]

- **Caches can be evicted under memory pressure.** xAI's docs note that "requests may be routed to different servers" and cache entries can drop. Use `x-grok-conv-id` to increase the probability of hitting the same cache, but do not assume caching is free-and-perpetual.
[source: docs.x.ai/developers/advanced-api-usage/prompt-caching/how-it-works, retrieved 2026-04-19]

## 7. Anti-Patterns

- **Do not set `reasoning_effort` on `grok-4.20` (non-reasoning) or `grok-4-1-fast`.** It returns an error. Remove the field.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

- **Do not assume `reasoning_effort` on `multi-agent` behaves like OpenAI.** Treat it as an agent-count selector (`low`/`medium` → 4 agents; `high`/`xhigh` → 16 agents) and budget cost/latency accordingly.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

- **Do not stream-parse function calls character-by-character.** The call arrives whole in one chunk. Buffer and parse by event type.
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

- **Do not start new work on `grok-3` or `grok-2`.** The 4.20 and 4.1 Fast families supersede them for all text use cases.

- **Do not rely on parametric knowledge for post-November-2024 facts.** Use `x_search` or `web_search` tools for current-events grounding.
[source: docs.x.ai/developers/models, retrieved 2026-04-19]

- **Do not ignore the `x-grok-conv-id` header for multi-turn sessions.** Without it, cache-node routing is best-effort — you may pay full input pricing on a cache that exists but was routed elsewhere.
[source: docs.x.ai/developers/advanced-api-usage/prompt-caching/how-it-works, retrieved 2026-04-19]

- **Do not assume custom `reasoning_effort` values beyond `low`/`medium`/`high`/`xhigh` work.** Only these four are documented on the multi-agent variant.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

## 8. Gaps

- **Max output tokens per model** is not quoted in the retrieved model-lineup excerpt.
- **Built-in tool parameter shapes** (`x_search`, `web_search`, `code_execution`, `collections_search`) beyond type names were not pulled in depth.
- **Image input placement conventions** within the OpenAI-compatible `content` array (detail parameters, resolution budgets) are undocumented in the retrieved primary sources.
- **Structured Outputs parameter shape** was not targeted in this retrieval pass beyond confirmation of support.
- **Live-search provider details** (X vs general web, freshness guarantees, citation format) are not captured.
- **Collections API / knowledge-base schema** structure is not covered here.
- **Speech-to-Text and Text-to-Speech** APIs are distinct surfaces with their own contracts, not covered in this reference.
- **Enterprise-tier features** beyond Grok 4.1 Fast enterprise availability were not pulled.
