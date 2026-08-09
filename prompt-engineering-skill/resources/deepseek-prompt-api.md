---
family: deepseek
scope: api
versions:
  - deepseek-v4-flash
  - deepseek-v4-pro
  - deepseek-ai/DeepSeek-V4-Flash-0731
  - deepseek-ai/DeepSeek-V3.2
retrieved: 2026-08-09
primary_sources:
  - https://api-docs.deepseek.com/updates
  - https://api-docs.deepseek.com/quick_start/pricing
  - https://api-docs.deepseek.com/guides/thinking_mode
  - https://api-docs.deepseek.com/guides/tool_calls
  - https://api-docs.deepseek.com/guides/responses_api
  - https://api-docs.deepseek.com/guides/anthropic_api
  - https://api-docs.deepseek.com/guides/kv_cache
  - https://api-docs.deepseek.com/api/create-chat-completion
  - https://api-docs.deepseek.com/quick_start/agent_integrations/codex/
  - https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.2
maturity_note: |
  DeepSeek V4 is the production generation. The 2026-07-31 update replaced the model
  behind `deepseek-v4-flash` with DeepSeek-V4-Flash-0731 (the official release; the
  April 2026 launch is retroactively labeled Preview) without changing the API model
  ID. `deepseek-v4-pro` still serves the Preview build; its official release is
  promised "soon". New since the last pass: a native OpenAI Responses API surface
  (flash-only), a documented default of thinking-on at effort high, a real `low`
  effort level on flash, USD pricing with much steeper cache-hit discounts, and a
  vendor pre-announcement of a significant future price increase. Legacy
  `deepseek-chat` / `deepseek-reasoner` retired 2026-07-24 and are gone from the
  model enum. Docs restructured: /guides/function_calling and /guides/reasoning_model
  now 404 (moved to /guides/tool_calls and /guides/thinking_mode).
---

# DeepSeek — API-Layer Reference

API-call-level detail for the current DeepSeek generation. Portable prompt-layer content (selection, thinking semantics, session management, anti-patterns) lives in `deepseek-prompt.md`.

## 1. API Surface

### Base URLs

- **OpenAI ChatCompletions format**: `https://api.deepseek.com`.
- **OpenAI Responses API format**: same base; `wire_api="responses"`. Flash-only (see §4).
- **Anthropic format**: `https://api.deepseek.com/anthropic`.
- **Beta**: `https://api.deepseek.com/beta` — strict-mode tool calls and features in validation.

[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]
[source: api-docs.deepseek.com/guides/responses_api, retrieved 2026-08-09]
[source: api-docs.deepseek.com/guides/anthropic_api, retrieved 2026-08-09]

Cross-family reasoning field-name and round-trip hazards are tabulated in `resources/openai-compatibility-surface.md`. DeepSeek's own statements are kept here.

### Model IDs

| ID                  | Serving model version   | Notes                                                       |
|---------------------|-------------------------|--------------------------------------------------------------|
| `deepseek-v4-flash` | DeepSeek-V4-Flash-0731  | Official release (public beta since 2026-07-31); 284B total / 13B active; effort low/high/max |
| `deepseek-v4-pro`   | DeepSeek-V4-Pro (Preview build) | 1.6T total / 49B active; official release "will follow soon"; effort high/max only |

The 0731 release did **not** introduce a new API model ID. The stable alias `deepseek-v4-flash` was repointed to the new checkpoint: "The API calling method remains unchanged — simply set the model name to `deepseek-v4-flash` to use the latest version." The pricing page documents the per-ID serving version explicitly.
[source: api-docs.deepseek.com/updates, retrieved 2026-08-09]
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]

The Chat Completions `model` enum now contains **only** the two V4 IDs. `deepseek-chat` and `deepseek-reasoner` retired 2026-07-24, 15:59 UTC and no longer appear in the enum or on the pricing page (see §9).
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]

Third-party aggregators pin the checkpoint under dated slugs (OpenRouter: `deepseek/deepseek-v4-flash-0731`) even though the native API does not.
[tier: 2, source: openrouter.ai/deepseek/deepseek-v4-flash-0731, retrieved 2026-08-09]

### Concurrency limits

Documented per model: **2500** concurrent for `deepseek-v4-flash`, **500** for `deepseek-v4-pro`.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]

### Pricing (USD, captured 2026-08-09)

| Per 1M tokens        | deepseek-v4-flash | deepseek-v4-pro |
|----------------------|-------------------|-----------------|
| Input, cache hit     | $0.0028           | $0.003625       |
| Input, cache miss    | $0.14             | $0.435          |
| Output               | $0.28             | $0.87           |

Cache-hit input is ~2% of cache-miss input on flash (~50x discount; ~120x on pro) — much steeper than the prior generation's 10x framing. DeepSeek has pre-announced a **significant across-the-board price increase** with no published amount or date: "We plan to raise the overall pricing for DeepSeek API services in the near future, with a significant increase expected." Treat the figures above as time-limited.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]

### SDKs

Any OpenAI-compatible SDK works by pointing `base_url` at `https://api.deepseek.com` (or `/beta`). The Responses API surface is reachable with `wire_api="responses"`-style clients (vendor documents a Codex `config.toml` using exactly that against the same base URL, with `model_reasoning_effort = "high"` as the shipped default). The Anthropic interface uses `https://api.deepseek.com/anthropic`.
[source: api-docs.deepseek.com/quick_start/agent_integrations/codex/, retrieved 2026-08-09]

### Open weights

- **`deepseek-ai/DeepSeek-V4-Flash-0731`** — MIT. Official release superseding the preview; same architecture and size (re-post-train only). Ships with a bundled **DSpark speculative-decoding module**, which is why HF safetensors metadata reads 304B params against the 284B-total/13B-active base architecture.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]
[source: api-docs.deepseek.com/updates, retrieved 2026-08-09]
- **`deepseek-ai/DeepSeek-V3.2`** — MIT, 685B total, DeepSeek Sparse Attention (DSA). Prior generation.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

## 2. Chat Template / Message Structure

### Special tokens (open weights)

DeepSeek uses full-width characters for control tokens — **not** the ASCII `<|...|>` form used by most open-weights families.

| Token                               | Purpose                                  |
|-------------------------------------|------------------------------------------|
| `<｜begin▁of▁sentence｜>`          | Beginning-of-sequence (BOS)              |
| `<｜end▁of▁sentence｜>`            | End-of-sequence (EOS)                    |
| `<｜User｜>`                        | User-turn delimiter                      |
| `<｜Assistant｜>`                   | Assistant-turn delimiter                 |
| `<｜Developer｜>`                   | Developer role (V3.2: search-agent scenarios only) |
| `<think>` / `</think>`              | Reasoning block wrapper                  |

[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

Note the characters: `｜` is U+FF5C (FULLWIDTH VERTICAL LINE), not ASCII `|`; `▁` is U+2581 (LOWER ONE EIGHTH BLOCK), not ASCII `_`. Hand-built chat strings substituting ASCII look right on screen but tokenize differently. Use the provided encoder. Whether 0731 changed any individual special tokens vs V3.2 is unverified (the 0731 `encoding/README.md` was not retrieved); the encoder interface is the stable contract.

### No Jinja template; `encoding_dsv4` encoder

The 0731 release again ships **no Jinja-format chat template**. It provides an `encoding` folder with Python `encode_messages(...)` / `parse_message_from_completion_text(...)`. New on 0731: `encode_messages` takes `thinking_mode` and `reasoning_effort` arguments — the effort level is a **chat-template-level control**, rendered into the token stream, not only an API parameter.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]

[applies-to: deepseek-ai/DeepSeek-V3.2] V3.2 likewise ships no Jinja template; use `encoding/encoding_dsv32.py`.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### API-layer message shape

```json
{
  "model": "deepseek-v4-flash",
  "messages": [
    { "role": "user",      "content": "..." },
    { "role": "assistant", "content": "..." },
    { "role": "tool",      "content": "...", "tool_call_id": "..." }
  ]
}
```

Roles on Chat Completions: `system`, `user`, `assistant`, `tool`. On the **Responses API**, input roles are `user` / `assistant` / `system` / `developer`, and `developer` **is treated as `system`** — a general-purpose alias on that surface, unlike the narrow V3.2 search-agent `<｜Developer｜>` role.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]
[source: api-docs.deepseek.com/guides/responses_api, retrieved 2026-08-09]

## 3. Sampling Parameters

- **In thinking mode, `temperature`, `top_p`, `presence_penalty`, and `frequency_penalty` are all unsupported** — passed values are ignored without error. Since thinking is now on by default (§4), a default-configuration caller has no live sampling knobs.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]
- `frequency_penalty` / `presence_penalty` are hard-deprecated on the native API in general: "This parameter is no longer supported. It will not take effect if you pass it to the API."
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]
  Verified 2026-07-19 (pre-0731): through OpenRouter this is **serving-stack-dependent**, not universal — max penalties were inert on Fireworks and Novita but measurably reduced repetition on Parasail (mean max repeated 4-gram 2.13 → 1.00, N=20/arm). Whether the penalties do anything for OpenRouter callers depends on the routed provider.
- Recommended local sampling for the 0731 open weights: **temperature=1.0, with top_p=0.95 for agentic scenarios and top_p=1.0 otherwise** — superseding the flat 1.0/1.0 recommendation of the preview generation.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]

## 4. Reasoning / Thinking Control

### Defaults and the effort ladder

**Thinking mode is enabled by default, with default effort `high`.**
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

Controls, via either parameter:

- `thinking: {"type": "enabled" | "disabled"}`, or
- `reasoning_effort: "low" | "high" | "max"`.

`deepseek-v4-flash` supports all three effort levels — `low` is a **real tier** on 0731, not an alias. `deepseek-v4-pro` temporarily supports only `high` and `max` (`low` is treated as `high`, `xhigh` as `max`).
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]

Doc inconsistency, flagged: create-chat-completion also says "medium and xhigh are mapped to high" generically, while the same paragraph and the thinking_mode table give the pro-only mapping xhigh→max. Off-ladder values are accepted and mapped rather than rejected; the exact mapping of `medium`/`xhigh` per model is [disputed: create-chat-completion vs thinking_mode tables disagree on where medium/xhigh land].

[field-observed] Via OpenRouter (Novita, fp8, N=1 per value, 2026-08-09): all of none/low/medium/high/xhigh/max returned 200; `reasoning_effort: "none"` disabled thinking entirely (0 reasoning tokens). On a nontrivial prompt, low produced ~180 reasoning tokens vs ~840 (high) and ~620 (max) — low is materially lighter; high vs max did not separate at N=1.

[field-observed] OpenRouter control mapping (Novita, N=1 per cell, 2026-08-09): OpenRouter's normalized controls work (`reasoning: {"enabled": false}` and `reasoning_effort: "none"` both disable thinking), but the DeepSeek-native `thinking: {"type": "disabled"}` body passed through OpenRouter did **not** disable thinking. Through OpenRouter, use OpenRouter's `reasoning` parameter family, not DeepSeek's native `thinking` field. Note also a catalog bug: OpenRouter's static catalog briefly exposed only a "high" tier for the 0731 slug while the live API advertised low/high/max — at least one agent CLI shipped a patch for it.
[tier: 2, source: github.com/can1357/oh-my-pi/issues/7307, retrieved 2026-08-09]

### Response shape

Thinking responses carry `reasoning_content` at the same level as `content` (thinking mode only):

```json
{
  "choices": [{
    "message": {
      "role": "assistant",
      "reasoning_content": "The chain-of-thought reasoning text...",
      "content": "The final answer text..."
    }
  }]
}
```

[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

### Critical multi-turn rule: `reasoning_content` round-trip (V4)

Unchanged on 0731. The round-trip behavior depends on whether the conversation carries tools:

- On turns that do **not** perform tool calls, passing `reasoning_content` back in `messages` is **ignored** (no error).
- For requests carrying the `tools` parameter, `reasoning_content` **must be fully passed back** in subsequent requests, or the native API returns **HTTP 400**. This is a **native-only** (`api.deepseek.com`) assertion — see the OpenRouter caveat below.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

Verified 2026-07-19 (DeepInfra) and re-verified 2026-08-09 on 0731 (Novita, N=2): the hard-400 does **not** reproduce via OpenRouter — a tool-call turn resubmitted with reasoning stripped returned HTTP 200, as did the intact resubmit. OpenRouter-routed code that strips reasoning on tool turns does not break, but the same code moved to the native API will 400. The first-party DeepSeek endpoint on OpenRouter could not be pinned from the test account (404 "No endpoints found" with `provider.order: ["deepseek"]`), so the native contract remains verified against DeepSeek's docs, not directly re-observed.

```python
# Tool-call turn: reasoning_content REQUIRED on round-trip (native API) or HTTP 400
messages.append({
    "role": "assistant",
    "content": prior_response.content,
    "reasoning_content": prior_response.reasoning_content,
    "tool_calls": prior_response.tool_calls,
})

# Non-tool-call turn: reasoning_content is ignored if included
messages.append({"role": "assistant", "content": prior_response.content})
```

[testable: id=deepseek.reasoning-content-roundtrip-tool-turn.v2, expected=on native api.deepseek.com a V4 tool-call turn omitting prior reasoning_content returns HTTP 400; a non-tool-call turn including it is accepted. Native-only: the 400 does not reproduce via OpenRouter (DeepInfra 2026-07-19 2/2; Novita on 0731 2026-08-09 2/2)]

### `max_tokens` includes CoT

`max_tokens >= reasoning_tokens + content_tokens`. Max output is 384K. On 0731 the vendor recommends a **maximum output length of 384K for both `high` and `max` effort** — reframed from the preview generation's Think-Max-only context-window recommendation.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]

[field-observed] When the CoT exhausts `max_tokens`, the response is a **200 with empty `content`** and `completion_tokens_details.reasoning_tokens == max_tokens` — not an error (observed 4/63 analysis-style turns at `max_tokens: 4096`, 2026-08-09, via OpenRouter/Novita). Detect via the reasoning-token count, and budget 8K+ for analysis turns with thinking on.

### Responses API (new surface, flash-only)

`deepseek-v4-flash` natively supports the OpenAI Responses API format, specifically adapted for Codex. `deepseek-v4-pro` support was promised for "early August 2026" and was **not yet live as of 2026-08-09**. Key contract points:

- **Stateless**: `previous_response_id` and `conversation` are not supported; `store` is always `false`. The caller manages history.
- `parallel_tool_calls` is ignored — parallel tool calling is **always enabled**; `max_tool_calls` is also ignored.
- Server-side **`web_search`** tool (executed on DeepSeek's servers; `search_context_size` and `user_location` ignored) and a custom **`apply_patch`** tool for Codex compatibility.
- `text.format` is fully supported (structured output path; `verbosity` accepted but no effect).
- Streaming events are fully enumerated (`response.created` … `response.reasoning_text.delta` … `response.completed` / `response.incomplete` / `response.failed`); there is **no `data: [DONE]`** terminator on this surface.

[source: api-docs.deepseek.com/guides/responses_api, retrieved 2026-08-09]

## 5. Tool Use / Function Calling

### Request shape (OpenAI-compatible)

```json
"tools": [
  {
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "Get current weather",
      "parameters": {
        "type": "object",
        "properties": { "location": { "type": "string" } },
        "required": ["location"],
        "additionalProperties": false
      },
      "strict": true
    }
  }
]
```

[source: api-docs.deepseek.com/guides/tool_calls, retrieved 2026-08-09]

### `tool_choice` and limits

- `tool_choice` accepts `none` / `auto` / `required` plus the named-function form. Defaults: `none` when no tools are present, `auto` when tools are present.
- **Maximum 128 functions** per request (now vendor-documented).
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]

### Parallel tool calls

Verified 2026-07-19 (pre-0731, OpenRouter/DeepInfra, 3/3): default behavior is parallel — two independent tools produced 2 `tool_calls` in one assistant turn with no parameter set. Handle multi-entry `tool_calls` arrays. On the Responses API and Anthropic surfaces, parallel tool use is always on and **cannot be disabled** (`parallel_tool_calls` / `disable_parallel_tool_use` ignored). The native Chat Completions surface documents no `parallel_tool_calls` parameter at all.
[source: api-docs.deepseek.com/guides/responses_api, retrieved 2026-08-09]
[source: api-docs.deepseek.com/guides/anthropic_api, retrieved 2026-08-09]

### Strict mode

Available via the **beta endpoint** (`base_url="https://api.deepseek.com/beta"`), supported in **both thinking and non-thinking mode**. Set `"strict": true` per function; requires all object properties `required` and `additionalProperties: false`. Supported JSON Schema features remain richer than OpenAI's subset — `enum`, `anyOf`, `$def`, `$ref` (recursive structures) — plus string `pattern` and `format` (email, hostname, ipv4, ipv6, uuid). **Not** supported: `minLength`/`maxLength`, `minItems`/`maxItems`.
[source: api-docs.deepseek.com/guides/tool_calls, retrieved 2026-08-09]

### Tool-result roundtrip

Append a `tool` role message with `tool_call_id`:

```json
{ "role": "tool", "tool_call_id": "call_abc", "content": "20C, sunny" }
```

[source: api-docs.deepseek.com/guides/tool_calls, retrieved 2026-08-09]

Community telemetry note: OpenRouter's provider table for the 0731 slug showed a 1.78-5.39% (avg 3.56%) tool-call error rate on DeepSeek's own first-party endpoint, vs 0.23-0.72% on the best third-party hosts of the same weights. Aggregator telemetry, not a vendor number.
[tier: 2, source: openrouter.ai/deepseek/deepseek-v4-flash-0731, retrieved 2026-08-09]

## 6. Structured Outputs

- **Chat Completions**: `response_format` supports **only `text` and `json_object`** — there is no dedicated `json_schema` path on this surface.
- **Responses API**: `text.format` is fully supported — the first vendor-documented schema-shaped output path outside function calling (flash-only, since the surface is flash-only).
- **Strict function calling** (beta endpoint) remains the most featureful path: force the extraction function via `tool_choice` and rely on strict-schema validation.

[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]
[source: api-docs.deepseek.com/guides/responses_api, retrieved 2026-08-09]

[field-observed] Via OpenRouter, `response_format` acceptance is provider-dependent: Novita rejected even `json_object` with a 400 while other hosts advertise `structured_outputs` support (N=1, 2026-08-09). Check the routed provider's `supported_parameters` before relying on it.

## 7. Caching, Batch, Streaming

### Context caching (automatic, rewritten mechanics)

The kv_cache guide was rewritten for the sliding-window-attention serving stack. Caching is still automatic (no parameter), but:

- Cached prefixes are stored and matched as **independent, complete "cache prefix units"** — a hit requires fully matching a stored unit, not merely sharing an arbitrary prefix. Units are cut at request boundaries, detected common prefixes, and fixed token intervals.
- The previously documented **64-token minimum no longer appears** anywhere in the guide.
- TTL unchanged: cleared automatically "within a few hours to a few days" after last use.
- Usage reporting unchanged: `usage.prompt_cache_hit_tokens` / `usage.prompt_cache_miss_tokens`.
- Pricing: cache-hit input $0.0028/M vs miss $0.14/M on flash (§1) — a ~50x discount, materially steeper than the prior 10x framing.

[source: api-docs.deepseek.com/guides/kv_cache, retrieved 2026-08-09]
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]

A `user_id` request parameter now exists and provides **KVCache isolation** (plus content-safety attribution and scheduling isolation) for multi-tenant deployments.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]

[field-observed] Cache interaction with thinking controls (OpenRouter/Novita, N=1 per cell, 2026-08-09, OpenAI-style `cached_tokens` reporting): a ~24K-token stable prefix hit the cache on repeat calls; changing `reasoning_effort` (high → low → max → high) **preserved** the cache hit, while toggling thinking **off** invalidated it (cached=0) and shrank the rendered prompt by ~79 tokens — consistent with a thinking-mode-specific template preamble. Session rule: hold the thinking on/off state constant mid-session; effort changes were cache-safe on this provider. Native-API cache-unit behavior may differ; not re-tested there.

### Streaming

- **Chat Completions**: standard SSE terminated by `data: [DONE]`. In thinking mode, delta events carry `reasoning_content` and `content` as separate fields at the same level. `finish_reason` enum: `stop`, `length`, `content_filter`, `tool_calls`, `insufficient_system_resource`. Logprobs are available on `reasoning_content` tokens as well as `content` tokens.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]
- **Responses API**: typed event stream, no `[DONE]` (§4).

### Batch

Still undocumented — no batch page exists on api-docs.deepseek.com and the pricing/API reference make no mention.

## 8. Deployment Flags (open weights)

- **`deepseek-ai/DeepSeek-V4-Flash-0731`** — MIT; the official flash release. Bundled **DSpark speculative-decoding module** drafting from the same checkpoint (no separate draft model): vLLM enables it with `--speculative-config` (`method: dspark`); SGLang with `--speculative-algorithm DSPARK` and no `--speculative-draft-model-path`. The bundle is why safetensors metadata reads 304B params vs the 284B/13B base architecture. Chat rendering requires `encoding_dsv4` (`encode_messages(...)` with `thinking_mode` / `reasoning_effort`); no Jinja template. Recommended sampling: temperature=1.0, top_p=0.95 agentic / 1.0 otherwise.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]
- **`deepseek-ai/DeepSeek-V3.2`** — MIT, 685B, DSA; prior generation; `encoding_dsv32.py`.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]
- Community-reported serving defect, unresolved as of 2026-08-09: a load-dependent output-corruption bug for self-hosted 0731 under **concurrent agentic traffic** (30-70 tools/request, 50K-500K contexts) — wrong-language insertions, DSML tool-call markup leaking into `reasoning_content` without a closing `</think>`, token-soup at the extreme. Reproduced by three operators on different hardware (2x H200, 8x H100, 1x B300) on **both SGLang and vLLM 0.26.0**; one single-GPU repro at ~12.5% of sequential requests. Clean on single-request smoke tests — load-test before production self-hosting.
[tier: 2, source: github.com/sgl-project/sglang/issues/33397, retrieved 2026-08-09]
- vLLM/SGLang minimum versions for 0731 are still not quoted by the vendor; the model card links a vLLM recipe and SGLang cookbook (not fetched this pass).

## 9. Deprecations and Breaking Changes

### Legacy model IDs retired (executed)

[applies-to: deepseek-chat, deepseek-reasoner]
`deepseek-chat` and `deepseek-reasoner` retired **2026-07-24, 15:59 UTC**. The retirement executed on schedule: both IDs are gone from the Chat Completions model enum and the pricing page. (No doc page states the retirement in past tense; the evidence is the deadline passing plus removal from the enum — a live-probe confirmation was not run.)
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-08-09]
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]

### April 2026 release retroactively labeled Preview

The April V4 launch is now titled "DeepSeek V4 Preview Release". The production generation is: flash = 0731 official release; pro = Preview build pending its own official release. Several pro-side promises ("early August 2026": Responses API support, Codex support, three-level effort) were still unfulfilled as of 2026-08-09 — re-check before relying on them.
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-08-09]
[source: api-docs.deepseek.com/guides/responses_api, retrieved 2026-08-09]

### Documentation URLs moved

`/guides/function_calling` and `/guides/reasoning_model` now return 404. Content lives at `/guides/tool_calls` and `/guides/thinking_mode`. Update any pinned citations.
[source: api-docs.deepseek.com/guides/function_calling (404), retrieved 2026-08-09]

### Caching contract changes

The 64-token minimum cacheable size is no longer documented; cache matching is now unit-based (§7). Pricing framing moved from yuan (10x discount) to USD (~50x/~120x discounts).
[source: api-docs.deepseek.com/guides/kv_cache, retrieved 2026-08-09]

### Pre-announced price increase

A significant across-the-board API price increase is announced with no amount or date. Budget models built on the $0.14/$0.28 figures should carry this as a known risk.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]

### V3.2-Speciale API endpoint expired

[applies-to: deepseek-ai/DeepSeek-V3.2-Speciale]
Weights-only deep-reasoning variant, no tool calls; its temporary API endpoint expired 2025-12-15. Weights remain on HF.
[source: api-docs.deepseek.com/updates, retrieved 2026-08-09]

### License

**MIT** (weights), unchanged on 0731 — still the most permissive license of any frontier-scale family in this reference library.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]

## 10. Gaps

- **Knowledge cutoff** — still undocumented for any V4 model, on both api-docs and the 0731 model card.
- **Batch API** — still undocumented.
- **`parallel_tool_calls` on native Chat Completions** — the parameter is not documented at all on that surface (Responses/Anthropic: always-on, cannot disable). Observed-parallel-by-default stands (2026-07-19, 3/3 via OpenRouter).
- **Effort mapping for `medium`/`xhigh`** — vendor pages disagree (§4); treat off-ladder values as best-effort mapped.
- **0731 special-token deltas** — `encoding/README.md` and `inference/README.md` on HF were not fetched; token-level changes vs `encoding_dsv32` unverified.
- **Native-API re-observation of the tool-turn 400** — doc-verified on 0731, not re-observed live (first-party OpenRouter endpoint unpinnable from the test account; native API not called this pass).
- **vLLM/SGLang minimum versions** for 0731 — not vendor-quoted.
- **DeepSeek Harness** (benchmark framework, "minimal mode") — "to be released soon"; vendor benchmark configs are not yet independently reproducible.
- **Param-count reconciliation** — news260424 still describes flash as 284B total / 13B active while HF safetensors metadata shows 304B for 0731; the model card attributes the difference to the bundled DSpark module, but no single vendor page states both numbers together.
