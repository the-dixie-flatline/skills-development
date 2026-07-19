---
family: grok
scope: api
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
  - https://docs.x.ai/developers/rest-api-reference/inference/chat
  - https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing
  - https://docs.x.ai/developers/migration/may-15-retirement
  - https://docs.x.ai/llms.txt
maturity_note: |
  Grok's API is OpenAI-compatible at the wire level, which makes most
  OpenAI SDK code portable with minor changes. The current flagship is
  grok-4.5 (500,000-token context); its `reasoning_effort` set is
  {low, medium, high}, default `high`, and `none` has been removed —
  reasoning cannot be disabled. grok-4.3 remains live as prior gen with the
  LARGER 1,000,000-token context and the {none, low, medium, high} set,
  default `low`. grok-build-0.1 is a fast coding model. Effective 2026-05-15
  the prior fast / 4.x / 3 slugs were retired; the migration page still routes
  the six retired text/reasoning slugs to grok-4.3 (not grok-4.5). The
  `x-grok-conv-id` header (Chat Completions) and `prompt_cache_key`
  (Responses API) are the optional levers for maximizing prompt-cache hit rate.
---

# Grok — API-Layer Reference

API-call-level detail for current Grok models. Portable prompt-layer content (selection, behavioral quirks, anti-patterns) lives in `grok-prompt.md`. For the cross-family OpenAI-compatibility matrix, see `resources/openai-compatibility-surface.md` (not duplicated here).

## 1. API Surface

### Endpoints and SDKs

- **xAI API** at `https://api.x.ai/v1` — OpenAI-compatible (same paths: `/chat/completions`, `/responses` where supported, `/batches`).
- **xAI SDK** (`xai_sdk` Python package) — first-party.
- **OpenAI SDK** — works against the xAI API by pointing `base_url` at `https://api.x.ai/v1` and swapping the API key.

[source: https://docs.x.ai/developers/rest-api-reference/inference/chat, retrieved 2026-06-01]
[source: https://docs.x.ai/llms.txt, retrieved 2026-06-01]

### Model IDs

Use the flagship, prior-gen, and coding model IDs directly:

```
grok-4.5          # flagship, 500K context
grok-4.3          # prior gen, still live, 1M context
grok-build-0.1    # fast coding model, 256k context
```

Aliases: `grok-4.5-latest` and `grok-build-latest` resolve to grok-4.5. The alias convention is `<name>` = latest stable, `<name>-latest` = latest (new features), `<name>-<date>` = pinned release. Additional current lineup IDs (1M context): `grok-4.20-0309-reasoning`, `grok-4.20-0309-non-reasoning`, `grok-4.20-multi-agent-0309`. The prior fast slugs and `grok-3` are retired and auto-redirect (see §9 for the 2026-05-15 retirement).

[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models/grok-4.5, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/migration/may-15-retirement, retrieved 2026-07-19]

## 2. Chat Template / Message Structure

Grok uses OpenAI-compatible JSON messages, not a special-token chat template.

### Basic request

```json
{
  "model": "grok-4.5",
  "messages": [
    { "role": "system", "content": "..." },
    { "role": "user", "content": "..." }
  ],
  "stream": false
}
```

[source: https://docs.x.ai/developers/rest-api-reference/inference/chat, retrieved 2026-06-01]

Roles: `system`, `user`, `assistant`, `tool` (standard OpenAI shape). Content supports text and image parts in the OpenAI-compatible format.

## 3. Sampling Parameters

Standard OpenAI-compatible fields (`temperature`, `top_p`, etc.) are accepted. The retrieved primary sources do not publish per-model recommended defaults or bounds; rely on per-workload validation rather than hard-coding values.

**Reasoning-model incompatibilities:**

- `presencePenalty` / `frequencyPenalty` / `stop` **cannot be used with reasoning models** and return an error. grok-4.5 is a reasoning model on every request (reasoning cannot be disabled), so these fields are rejected on every grok-4.5 call. On grok-4.3 they are rejected whenever `reasoning_effort` is not `none`.
- `logprobs` / `top_logprobs` are **silently ignored** on grok-4.20 and newer.

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

## 4. Reasoning / Thinking Control

### `reasoning_effort` semantics (generation-specific)

The accepted value set and default differ by generation. Split any wrapper mapping accordingly.

| Model      | Accepted values              | Default | `none` available? |
|------------|------------------------------|---------|-------------------|
| `grok-4.5` | {low, medium, high}          | `high`  | no — removed; reasoning cannot be disabled |
| `grok-4.3` | {none, low, medium, high}    | `low`   | yes — `none` = non-reasoning response |

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
[testable: id=grok.reasoning-effort-none-disables.v1, expected=grok-4.3 with reasoning_effort="none" returns no reasoning content / zero reasoning tokens] [applies-to: grok-4.3]
[testable: id=grok.reasoning-effort-none-rejected-45.v1, expected=grok-4.5 with reasoning_effort="none" returns an error] [applies-to: grok-4.5]
Verified 2026-07-19: grok-4.3 with `reasoning_effort="none"` returned 0 reasoning tokens and no reasoning content (3/3, native xAI API); grok-4.5 with `reasoning_effort="none"` returned HTTP 400 `"This model does not support reasoning_effort value 'none'."` (3/3, native xAI API). The accept-vs-reject divergence is confirmed on both halves.

Per-value effect (both models): `low` = least effort / lowest latency (grok-4.5 floor); `medium` = more deliberation; `high` = maximum reasoning effort (grok-4.5 default). Only grok-4.3 has `none` (no reasoning; lowest latency).

Wrappers porting OpenAI's `reasoning.effort`: the enums differ. Constrain to {low, medium, high} for grok-4.5 and {none, low, medium, high} for grok-4.3. A portable `none` assumption is stale for the flagship.

There is no undocumented `xhigh` effort on Grok. The system card labels competitor models `(xhigh)` but Grok 4.5 always `(high)`; the documented {low, medium, high} set stands.
[source: https://media.x.ai/v1/website/card-7f81d41b.pdf, retrieved 2026-07-19]

### Reasoning output in responses

- **Reasoning content (non-streaming, Chat Completions)**: `message.reasoning_content`.
- **Reasoning token count**: `usage.completion_tokens_details.reasoning_tokens`.
- **Streamed reasoning**: `chunk.reasoning_content` (Chat Completions / xAI SDK), or the `response.reasoning_summary_text.delta` SSE event (Responses API).
- **Encrypted full reasoning**: opt-in via `include: ["reasoning.encrypted_content"]` (Responses API only).

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/rest-api-reference/inference/chat, retrieved 2026-06-01]

## 5. Tool Use / Function Calling

### Request shape (OpenAI-compatible)

```json
"tools": [
  {
    "type": "function",
    "name": "get_weather",
    "description": "Get current weather",
    "parameters": {
      "type": "object",
      "properties": { "location": { "type": "string" } },
      "required": ["location"]
    }
  }
],
"tool_choice": "auto"
```

`tool_choice` accepts `"auto"` (default), `"required"`, `"none"`, or `{"type": "function", "function": {"name": "..."}}`.

Max **200 tools per request**; each `name` ≤200 characters.

[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

### Response shape

Tool calls appear as a `tool_calls` array on the assistant message:

```json
{
  "tool_calls": [
    {
      "id": "call_abc",
      "type": "function",
      "function": {
        "name": "get_weather",
        "arguments": "{\"location\":\"Paris, FR\"}"
      }
    }
  ]
}
```

[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

### Tool-result roundtrip

- **xAI SDK**: `tool_result(json.dumps(result))`.
- **OpenAI SDK**: append `{"type": "function_call_output", "call_id": "...", "output": json.dumps(result)}` to the conversation.

[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

### Parallel tool calls

Enabled by default. Disable with `"parallel_tool_calls": false` to force sequential invocation.
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

### Streaming behavior for function calls

Function calls are **not streamed progressively** — the call arrives whole in a single SSE chunk. Accumulating SSE deltas as if they were free-form text will misparse tool calls. Handle event types explicitly.
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]
[testable: id=grok.function-call-whole-chunk-streaming.v1, expected=SSE stream for a response with a tool call emits the tool call in one complete chunk rather than streaming arguments incrementally]
Verified 2026-07-19: streaming a tool call, the arguments arrived in one SSE delta chunk (`arg_fragment_chunks=1`), not fragmented across many deltas (3/3, native xAI API). Reassembly of fragmented tool-call arguments is unnecessary for Grok, but harmless.

### Built-in tools

Tools that execute on xAI servers rather than the caller's code. The verified developer-API surface is the server-side Web Search / X Search tools (`web_search`), which return `response.citations`. This is Grok's differentiating capability vs other frontier families.

The consumer "DeepSearch" agent (grok.com / X) is a separate product surface, not a developer-API feature. For the cross-family async-research-agent comparison, see `resources/deep-research-agents.md` (Grok is documented there as the web-search tool-use shape, not a managed async agent).

Parameter shapes for the built-in tools beyond the type name were not captured in this retrieval pass; see §10 (Gaps).

[source: https://docs.x.ai/developers/tools/web-search, retrieved 2026-06-01]

## 6. Structured Outputs

Structured Outputs are supported on current Grok models (per the release notes). Exact parameter shape (field name, schema constraints, strict-mode semantics) was not captured in this retrieval pass — see §10 (Gaps). Community practice aligns with OpenAI's `response_format` / `text.format` shape given the OpenAI-compatible API, but this has not been verified verbatim against Grok's primary docs.
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]
Verified 2026-07-19: `response_format: {"type": "json_schema", strict}` was accepted (200) and produced conformant output on grok-4.3 (3/3, native xAI API), confirming json_schema parity with OpenAI Chat Completions.

## 7. Caching, Batch, Streaming

### Automatic prompt caching

- **Always on** — no explicit enablement required.
- **Cache-pin field**: set `x-grok-conv-id` (or `prompt_cache_key`) to pin routing to the same cache node. `x-grok-conv-id` is the Chat Completions HTTP header; `prompt_cache_key` is the Responses-API request field.
- **Reporting**: cached token count is at `usage.prompt_tokens_details.cached_tokens` (Chat Completions) / `usage.input_tokens_details.cached_tokens` (Responses API).
- **grok-4.3 cached input**: $0.20/1M. **grok-4.5 cached input**: not confirmed on the pricing page (§10) — do not assume a value.

[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models/grok-4.3, retrieved 2026-06-01]

Minimum cacheable tokens, cache TTL, and exhaustive invalidation rules were not quoted in the retrieved primary excerpt; see §10 (Gaps).

### Pricing tiers and the 200K repricing cliff

Both grok-4.5 and grok-4.3 are bifurcated at a 200K prompt threshold, and crossing it reprices the WHOLE request (not just tokens above 200K) at the higher rate.

| Model      | Standard (prompt ≤200K) | Long-context (prompt >200K) |
|------------|-------------------------|------------------------------|
| `grok-4.5` | $2.00 in / $6.00 out per 1M | $4.00 in / $12.00 out per 1M |
| `grok-4.3` | $1.25 in / $2.50 out per 1M | $2.50 in / $5.00 out per 1M  |

[source: https://docs.x.ai/developers/pricing, retrieved 2026-07-19]

### Streaming

Standard SSE streaming on the OpenAI-compatible chat completions endpoint. Streamed reasoning surfaces as `chunk.reasoning_content` (Chat Completions / xAI SDK) or the `response.reasoning_summary_text.delta` event (Responses API). Notable rule: **function calls arrive as whole chunks, not streamed incrementally**.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

## 8. Deployment Flags (closed-platform routing)

- **`x-grok-conv-id` HTTP header (Chat Completions) / `prompt_cache_key` field (Responses API)** — optional; pin routing to the cache node serving a conversation to maximize cache-hit rate.

[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-07-19]

No region-routing (Bedrock-style) or data-residency flags are surfaced in the retrieved primary sources — Grok is served from xAI infrastructure directly.

## 9. Deprecations and Breaking Changes

### grok-4.5 supersedes grok-4.3 (flagship inversion)

grok-4.5 is the current flagship. Two breaking behavior changes vs grok-4.3 for callers migrating up:

- **`reasoning_effort` set and default changed.** grok-4.5 removes `none` and defaults to `high`; grok-4.3 kept `none` and defaulted to `low`. Code that sent `reasoning_effort: "none"` for a fast path will error on grok-4.5. See §4.
- **Context shrank.** grok-4.5 is 500K vs grok-4.3's 1M. A prompt sized for grok-4.3 may not fit grok-4.5. See `grok-prompt.md` §1.

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models/grok-4.5, retrieved 2026-07-19]

### 2026-05-15 slug retirement

Effective **2026-05-15 at 12:00 PM PT**, these slugs were retired from the xAI API:

```
grok-4-1-fast-reasoning
grok-4-1-fast-non-reasoning
grok-4-fast-reasoning
grok-4-fast-non-reasoning
grok-4-0709
grok-code-fast-1
grok-3
grok-imagine-image-pro
```

Redirect targets (note: NOT all to grok-4.5):

- The six text/reasoning slugs (`grok-4-1-fast-*`, `grok-4-fast-*`, `grok-4-0709`, `grok-3`) auto-redirect to **`grok-4.3`** — retired reasoning slugs forced to `reasoning_effort: low`, retired non-reasoning slugs forced to `reasoning_effort: none`.
- `grok-code-fast-1` redirects to **`grok-build-0.1`**.
- `grok-imagine-image-pro` redirects to **`grok-imagine-image-quality`** (image-gen).

**Vendor self-inconsistency worth flagging:** xAI now promotes grok-4.5 as the flagship, yet the May-15 migration page still falls the retired text/reasoning slugs back to grok-4.3, not grok-4.5. Do not assume a retired slug now lands on the current flagship — it lands on prior-gen grok-4.3. Pin `grok-4.5`, `grok-4.3`, or `grok-build-0.1` explicitly for new work.

[source: https://docs.x.ai/developers/migration/may-15-retirement, retrieved 2026-07-19]
[source: https://docs.x.ai/developers/models, retrieved 2026-07-19]

### Reasoning-model parameter incompatibilities

`presencePenalty` / `frequencyPenalty` / `stop` return an error on reasoning models (every grok-4.5 request; grok-4.3 when effort is not `none`); `logprobs` / `top_logprobs` are silently ignored on grok-4.20 and newer. See §3.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]

### Function call streaming shape

Function calls arrive whole in a single SSE chunk. Callers that assumed incremental streaming (some OpenAI-era code) will misparse.
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

## 10. Gaps

- **grok-4.5 cached-input pricing** is not documented at https://docs.x.ai/developers/pricing (base tiers and the 200K threshold rule are published; the cached line items are not), checked 2026-07-19.
- **grok-build-0.1 pricing and parameter surface** are not documented at https://docs.x.ai/developers/pricing, checked 2026-07-19.
- **Max output tokens per model** is not documented at https://docs.x.ai/developers/models/grok-4.5 (only the 500K context window is stated); confirmed absence, checked 2026-07-19.
- **Minimum cacheable tokens** for Grok's automatic caching is not quoted in the retrieved primary caching excerpt.
- **Cache TTL** and explicit invalidation rules are not quoted.
- **Structured Outputs parameter shape** is not verified verbatim against Grok's primary docs; OpenAI-compatible shape is assumed but flagged unverified above.
- **Built-in tool parameter shapes** (`web_search`, X Search, code execution, collections) beyond type names were not captured.
- **Image-content-part shape** in request messages (detail parameter, resolution budgets, file vs URL) is not quoted.
- **grok-4.3-specific knowledge cutoff** is not documented at https://docs.x.ai/developers/models, checked 2026-07-19; only the grok-4.5 statements are published and they conflict (Feb 1 2026 vs Jan 2026).
