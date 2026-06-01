---
family: grok
scope: api
versions:
  - grok-4.3
  - grok-build-0.1
retrieved: 2026-06-01
primary_sources:
  - https://docs.x.ai/developers/models
  - https://docs.x.ai/developers/model-capabilities/text/reasoning
  - https://docs.x.ai/developers/rest-api-reference/inference/chat
  - https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing
  - https://docs.x.ai/developers/migration/may-15-retirement
  - https://docs.x.ai/llms.txt
maturity_note: |
  Grok's API is OpenAI-compatible at the wire level, which makes most
  OpenAI SDK code portable with minor changes. The current flagship is
  grok-4.3 (1,000,000-token context, supports non-reasoning mode); reasoning
  is controlled by `reasoning_effort` ({none, low, medium, high}, default
  "low"). grok-build-0.1 is a fast coding model. Effective 2026-05-15 the
  prior fast / 4.x / 3 slugs were retired and auto-redirect to grok-4.3
  (or grok-build-0.1 for grok-code-fast-1). The `x-grok-conv-id` HTTP header
  is the optional lever for maximizing prompt-cache hit rates.
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

Use the flagship and coding model IDs directly:

```
grok-4.3
grok-build-0.1
```

`grok-4`, `grok-4-latest`, `grok-3`, and the prior fast slugs are listed as aliases that resolve to `grok-4.3` (see §9 for the 2026-05-15 retirement and redirect behavior).

[source: docs.x.ai/developers/models, retrieved 2026-06-01]
[source: https://docs.x.ai/developers/migration/may-15-retirement, retrieved 2026-06-01]

## 2. Chat Template / Message Structure

Grok uses OpenAI-compatible JSON messages, not a special-token chat template.

### Basic request

```json
{
  "model": "grok-4.3",
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

- `presencePenalty` / `frequencyPenalty` / `stop` **cannot be used with reasoning models** and return an error.
- `logprobs` / `top_logprobs` are **silently ignored** on grok-4.20 and newer.

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

## 4. Reasoning / Thinking Control

### `reasoning_effort` semantics

`grok-4.3` supports `reasoning_effort` with values {none, low, medium, high}. It defaults to `"low"`; `"none"` means no reasoning occurs (a non-reasoning response). This is a normal reasoning control, not a per-variant semantic.

| Value      | Effect                                          |
|------------|-------------------------------------------------|
| `none`     | No reasoning; lower latency                      |
| `low`      | Default                                          |
| `medium`   | More deliberation                                |
| `high`     | Maximum reasoning effort                         |

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]
[testable: id=grok.reasoning-effort-none-disables.v1, expected=grok-4.3 with reasoning_effort="none" returns no reasoning content / zero reasoning tokens]

Wrappers porting OpenAI's `reasoning.effort` map cleanly, except OpenAI's value set differs; constrain to {none, low, medium, high} for grok-4.3.

### Reasoning output in responses

- **Reasoning content (non-streaming, Chat Completions)**: `message.reasoning_content`.
- **Reasoning token count**: `usage.completion_tokens_details.reasoning_tokens`.
- **Streamed reasoning**: `chunk.reasoning_content` (Chat Completions / xAI SDK), or the `response.reasoning_summary_text.delta` SSE event (Responses API).
- **Encrypted full reasoning**: opt-in via `include: ["reasoning.encrypted_content"]` (Responses API only).

[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]
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

### Built-in tools

Tools that execute on xAI servers rather than the caller's code. The verified developer-API surface is the server-side Web Search / X Search tools (`web_search`), which return `response.citations`. This is Grok's differentiating capability vs other frontier families.

The consumer "DeepSearch" agent (grok.com / X) is a separate product surface, not a developer-API feature. For the cross-family async-research-agent comparison, see `resources/deep-research-agents.md` (Grok is documented there as the web-search tool-use shape, not a managed async agent).

Parameter shapes for the built-in tools beyond the type name were not captured in this retrieval pass; see §10 (Gaps).

[source: https://docs.x.ai/developers/tools/web-search, retrieved 2026-06-01]

## 6. Structured Outputs

Structured Outputs are supported on current Grok models (per the release notes). Exact parameter shape (field name, schema constraints, strict-mode semantics) was not captured in this retrieval pass — see §10 (Gaps). Community practice aligns with OpenAI's `response_format` / `text.format` shape given the OpenAI-compatible API, but this has not been verified verbatim against Grok's primary docs.
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]
[unverified] `response_format: {"type": "json_schema", ...}` is accepted on current Grok reasoning and non-reasoning variants identically to OpenAI Chat Completions.

## 7. Caching, Batch, Streaming

### Automatic prompt caching

- **Always on** — no explicit enablement required.
- **`grok-4.3` cached input**: $0.20/MTok.
- **Reporting**: cached token count is at `usage.prompt_tokens_details.cached_tokens` (Chat Completions) / `usage.input_tokens_details.cached_tokens` (Responses API).
- **Cache-hit maximization**: set the optional `x-grok-conv-id` HTTP header on multi-turn sessions to pin routing to the same cache node.

[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-06-01]
[source: docs.x.ai/developers/models/grok-4.3, retrieved 2026-06-01]

Minimum cacheable tokens, cache TTL, and exhaustive invalidation rules were not quoted in the retrieved primary excerpt; see §10 (Gaps).

### Streaming

Standard SSE streaming on the OpenAI-compatible chat completions endpoint. Streamed reasoning surfaces as `chunk.reasoning_content` (Chat Completions / xAI SDK) or the `response.reasoning_summary_text.delta` event (Responses API). Notable rule: **function calls arrive as whole chunks, not streamed incrementally**.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

## 8. Deployment Flags (closed-platform routing)

- **`x-grok-conv-id` HTTP header** — optional; pins routing to the cache node serving a conversation to maximize cache-hit rate.

[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-06-01]

No region-routing (Bedrock-style) or data-residency flags are surfaced in the retrieved primary sources — Grok is served from xAI infrastructure directly.

## 9. Deprecations and Breaking Changes

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

Retired slugs auto-redirect to `grok-4.3`: reasoning slugs resolve at `low` effort, non-reasoning slugs at `none` effort; `grok-code-fast-1` redirects to `grok-build-0.1`. Separately, `grok-4`, `grok-4-latest`, `grok-3`, and the fast slugs are listed as **aliases** of `grok-4.3` (they already resolve to it). Note `grok-4` itself is not in the retirement list (only `grok-4-0709`).

Pin `grok-4.3` or `grok-build-0.1` directly for new work.

[source: https://docs.x.ai/developers/migration/may-15-retirement, retrieved 2026-06-01]
[source: docs.x.ai/developers/models, retrieved 2026-06-01]

### Reasoning-model parameter incompatibilities

`presencePenalty` / `frequencyPenalty` / `stop` return an error on reasoning models; `logprobs` / `top_logprobs` are silently ignored on grok-4.20 and newer. See §3.
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]

### Function call streaming shape

Function calls arrive whole in a single SSE chunk. Callers that assumed incremental streaming (some OpenAI-era code) will misparse.
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

## 10. Gaps

- **Minimum cacheable tokens** for Grok's automatic caching is not quoted in the retrieved primary caching excerpt.
- **Cache TTL** and explicit invalidation rules are not quoted.
- **Structured Outputs parameter shape** is not verified verbatim against Grok's primary docs; OpenAI-compatible shape is assumed but flagged unverified above.
- **Built-in tool parameter shapes** (`web_search`, X Search, code execution, collections) beyond type names were not captured.
- **`grok-build-0.1` pricing and parameter surface** are not quoted; Playground "coming soon".
- **Image-content-part shape** in request messages (detail parameter, resolution budgets, file vs URL) is not quoted.
- **Max output tokens per model** is not listed in the retrieved lineup excerpt.
- **`grok-4.3`-specific knowledge cutoff** is not documented; only the Grok 3 / Grok 4 November 2024 statement is published.
