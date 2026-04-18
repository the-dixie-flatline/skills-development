---
family: grok
scope: api
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
  Grok's API is OpenAI-compatible at the wire level, which makes most
  OpenAI SDK code portable with minor changes. Two Grok-specific behaviors
  matter most: `reasoning_effort` has non-portable semantics (rejected on
  most models; controls agent count on the multi-agent variant), and the
  `x-grok-conv-id` HTTP header is the recommended lever for maximizing
  prompt-cache hit rates. Some caching mechanics (minimum cacheable tokens,
  exact TTL) were not quoted in the retrieved primary sources and appear in
  Gaps.
---

# Grok — API-Layer Reference

API-call-level detail for current Grok 4.x models. Portable prompt-layer content (selection, behavioral quirks, anti-patterns) lives in `grok-prompt.md`.

## 1. API Surface

### Endpoints and SDKs

- **xAI API** at `https://api.x.ai/v1/...` — OpenAI-compatible (same paths: `/chat/completions`, `/responses` where supported, `/batches`).
- **xAI SDK** (`xai_sdk` Python package) — first-party.
- **OpenAI SDK** — works against the xAI API by pointing `base_url` at `api.x.ai/v1` and swapping the API key.

[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

### Specialized endpoints

- **Batch API** — supports chat completions, image generation, image editing, and video generation.
- **Speech-to-Text API** — 25 languages, batch + streaming modes.
- **Text-to-Speech API** — "natural-sounding speech".
- **Grok Imagine API** — bundle for video/audio generative workflows.

[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]

### Model IDs

Dated and aliased IDs both work. Use dated IDs for pinning:

```
grok-4.20-0309-reasoning
grok-4.20-0309-non-reasoning
grok-4.20-multi-agent-0309
grok-4-1-fast-reasoning
grok-4-1-fast-non-reasoning
```

[source: docs.x.ai/developers/models, retrieved 2026-04-19]

## 2. Chat Template / Message Structure

Grok uses OpenAI-compatible JSON messages, not a special-token chat template.

### Basic request

```json
{
  "model": "grok-4.20-0309-reasoning",
  "messages": [
    { "role": "system", "content": "..." },
    { "role": "user", "content": "..." }
  ],
  "stream": false
}
```

[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

Roles: `system`, `user`, `assistant`, `tool` (standard OpenAI shape). Content supports text and image parts in the OpenAI-compatible format.

## 3. Sampling Parameters

Standard OpenAI-compatible fields (`temperature`, `top_p`, etc.) are accepted. The retrieved primary sources do not publish per-model recommended defaults or bounds; rely on per-workload validation rather than hard-coding values.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

## 4. Reasoning / Thinking Control

### Per-model `reasoning_effort` semantics

The `reasoning_effort` parameter is the most Grok-specific behavior to get right.

| Model                                 | Accepts `reasoning_effort`? | Semantics                                          |
|---------------------------------------|-----------------------------|----------------------------------------------------|
| `grok-4.20-0309-reasoning`            | No (reasons automatically)  | Parameter not needed; reasoning depth is internal  |
| `grok-4.20-0309-non-reasoning`        | **No — returns error**      | Do not send the field                              |
| `grok-4.20-multi-agent-0309`          | **Yes — controls agent count** | `low`/`medium` → 4 agents; `high`/`xhigh` → 16 agents |
| `grok-4-1-fast-reasoning`             | No (reasons automatically)  | Reasoning handled internally                       |
| `grok-4-1-fast-non-reasoning`         | **No — returns error**      | Do not send the field                              |

[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]
[testable: id=grok.reasoning-effort-rejected-non-reasoning.v1, expected=request to grok-4.20-0309-non-reasoning with reasoning_effort="high" returns HTTP 4xx]
[testable: id=grok.multi-agent-effort-controls-count.v1, expected=grok-4.20-multi-agent-0309 with reasoning_effort="high" produces more parallel agent traces than reasoning_effort="low"]

This is a **breaking divergence from OpenAI conventions**. Wrappers that always-send `reasoning_effort` must either omit the field or branch on model ID.

### Reasoning output in responses

- **Token count**: `response.usage.reasoning_tokens`.
- **Streamed summary**: `reasoning_content` chunks in the SSE stream, or `response.reasoning_summary_text.delta` events via the OpenAI SDK pathway.
- **Encrypted full reasoning**: opt-in via `include: ["reasoning.encrypted_content"]` in the request.
- **Billing**: reasoning tokens are billed as part of total output consumption — not a separate line item.

[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

### Example request / response (reasoning)

```json
{
  "input": [
    { "role": "system", "content": "You are a highly intelligent AI assistant." },
    { "role": "user",   "content": "What is 101*3?" }
  ],
  "model": "grok-4.20-reasoning",
  "stream": false
}
```

Response surface:

```
Final Response: The result of 101 multiplied by 3 is 303.
Completion tokens: 14
Reasoning tokens: 310
```

[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

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

Tools that execute on xAI servers rather than the caller's code:

- **Web Search** (`web_search`) — general web grounding.
- **X Search** (`x_search`) — real-time X / Twitter data access. This is Grok's differentiating capability vs other frontier families.
- **Code Execution** — server-side code execution.
- **Collections Search** — search uploaded knowledge-base collections.

Parameter shapes for each built-in tool beyond the type name were not captured in this retrieval pass; see §10 (Gaps).

[source: docs.x.ai/developers/tools/overview, retrieved 2026-04-19]
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]

## 6. Structured Outputs

Structured Outputs are supported on current Grok models (per the release notes). Exact parameter shape (field name, schema constraints, strict-mode semantics) was not captured in this retrieval pass — see §10 (Gaps). Community practice aligns with OpenAI's `response_format` / `text.format` shape given the OpenAI-compatible API, but this has not been verified verbatim against Grok's primary docs.
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]
[unverified] `response_format: {"type": "json_schema", ...}` is accepted on current Grok reasoning and non-reasoning variants identically to OpenAI Chat Completions.

## 7. Caching, Batch, Streaming

### Automatic prompt caching

- **Always on** — no explicit enablement required.
- **Discount**: cached-input pricing is ~10× cheaper than base input (e.g. `grok-4.20` $2.00/MTok input vs $0.20/MTok cached; `grok-4-1-fast` $0.20 vs $0.05). Effectively a 90% discount.
- **Reporting**: cached token count is in the response `usage` object.
- **Cache-hit maximization**: set the `x-grok-conv-id` HTTP header on multi-turn sessions to pin routing to the same cache node.
- **Eviction**: cache entries can be evicted under memory pressure, and requests may be routed to different servers. Caching is best-effort, not guaranteed.

[source: docs.x.ai/developers/models, retrieved 2026-04-19]
[source: docs.x.ai/developers/advanced-api-usage/prompt-caching/how-it-works, retrieved 2026-04-19]

Minimum cacheable tokens, cache TTL, and exhaustive invalidation rules were not quoted in the retrieved primary excerpt; see §10 (Gaps).

### Batch API

Batch API supports chat completions plus image generation, image editing, and video generation (expanded in recent releases). Latency-insensitive pricing discounts apply (standard industry pattern; exact percentage not in retrieved excerpt).
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]

### Streaming

Standard SSE streaming on the OpenAI-compatible chat completions endpoint. Notable rule: **function calls arrive as whole chunks, not streamed incrementally**.
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

## 8. Deployment Flags (closed-platform routing)

- **`x-grok-conv-id` HTTP header** — pins routing to the cache node serving a conversation; maximizes cache-hit rate.
- **Rate limits** — 10M TPM / 1,800 RPM on text and reasoning tiers.
- **Enterprise tier** — `grok-4.1-fast` is available in the xAI Enterprise API.

[source: docs.x.ai/developers/advanced-api-usage/prompt-caching/how-it-works, retrieved 2026-04-19]
[source: docs.x.ai/developers/models, retrieved 2026-04-19]
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]

No region-routing (Bedrock-style) or data-residency flags are surfaced in the retrieved primary sources — Grok is served from xAI infrastructure directly.

## 9. Deprecations and Breaking Changes

### `reasoning_effort` semantic divergence from OpenAI

[applies-to: grok-4.20-0309-non-reasoning, grok-4-1-fast-reasoning, grok-4-1-fast-non-reasoning]
Sending `reasoning_effort` on non-multi-agent, non-reasoning models **returns an error**. Callers porting OpenAI patterns must branch on model ID or strip the field.

[applies-to: grok-4.20-multi-agent-0309]
`reasoning_effort` controls **agent count**, not reasoning depth. Values map: `low`/`medium` → 4 agents, `high`/`xhigh` → 16 agents.
[source: docs.x.ai/docs/guides/reasoning, retrieved 2026-04-19]

### Function call streaming shape

[applies-to: grok-4.20-0309-reasoning, grok-4.20-0309-non-reasoning, grok-4.20-multi-agent-0309, grok-4-1-fast-reasoning, grok-4-1-fast-non-reasoning]
Function calls arrive whole in a single SSE chunk. Callers that assumed incremental streaming (some OpenAI-era code) will misparse.
[source: docs.x.ai/docs/guides/function-calling, retrieved 2026-04-19]

### Legacy models

`grok-3` remains GA via the API but is superseded by 4.20 and 4.1 Fast for new work. `grok-2` is older still. No explicit retirement date is quoted in the retrieved primary sources.
[source: docs.x.ai/developers/release-notes, retrieved 2026-04-19]

## 10. Gaps

- **Minimum cacheable tokens** for Grok's automatic caching is not quoted in the retrieved primary caching excerpt.
- **Cache TTL** and explicit invalidation rules are not quoted (beyond "evicted under memory pressure, may be routed to different servers").
- **Structured Outputs parameter shape** is not verified verbatim against Grok's primary docs; OpenAI-compatible shape is assumed but flagged unverified above.
- **Built-in tool parameter shapes** (`x_search`, `web_search`, `code_execution`, `collections_search`) beyond type names were not captured.
- **Image-content-part shape** in request messages (detail parameter, resolution budgets, file vs URL) is not quoted.
- **Max output tokens per model** is not listed in the retrieved lineup excerpt.
- **Batch API discount percentage** and exact request/response contract are not covered here.
- **Speech-to-Text / Text-to-Speech / Grok Imagine parameter surfaces** are separate endpoints and not covered in this reference.
- **Knowledge-cutoff verification date** beyond "November 2024" (e.g. exact day) is not stated.
- **Multi-agent variant trace shape** — how 4 or 16 parallel agents' work is exposed to the caller — is not captured.
