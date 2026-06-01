---
family: deepseek
scope: api
versions:
  - deepseek-v4-flash
  - deepseek-v4-pro
  - deepseek-ai/DeepSeek-V3.2
  - deepseek-ai/DeepSeek-V3.2-Speciale
retrieved: 2026-06-01
primary_sources:
  - https://api-docs.deepseek.com/news/news260424
  - https://api-docs.deepseek.com/quick_start/pricing
  - https://api-docs.deepseek.com/updates
  - https://api-docs.deepseek.com/guides/thinking_mode
  - https://api-docs.deepseek.com/guides/reasoning_model
  - https://api-docs.deepseek.com/guides/function_calling
  - https://api-docs.deepseek.com/guides/kv_cache
  - https://api-docs.deepseek.com/api/create-chat-completion
  - https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.2
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.2-Speciale
maturity_note: |
  DeepSeek V4 is the current production generation, exposed as `deepseek-v4-flash`
  and `deepseek-v4-pro` over both OpenAI ChatCompletions and Anthropic interfaces.
  Open weights are MIT-licensed, context length is 1M tokens (default across all
  DeepSeek services), and max output is 384K. The beta endpoint exposes strict-mode
  function calling with richer schema features than OpenAI's subset. The V4 release
  ships WITHOUT a Jinja chat template — callers must use DeepSeek's Python encoder.
  Context caching on disk is automatic and materially cheaper than most competitors.
  Legacy API model names `deepseek-chat` / `deepseek-reasoner` map onto V4-Flash but
  hard-retire and become inaccessible after Jul 24th, 2026, 15:59 UTC.
---

# DeepSeek — API-Layer Reference

API-call-level detail for the current DeepSeek generation. Portable prompt-layer content (selection, thinking semantics, anti-patterns) lives in `deepseek-prompt.md`.

## 1. API Surface

### Base URLs

- **OpenAI ChatCompletions format**: `https://api.deepseek.com`.
- **Anthropic format**: `https://api.deepseek.com/anthropic`.

[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]

- **Beta**: `https://api.deepseek.com/beta` — same OpenAI shape plus strict-mode tool calls and newer features in validation.
[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

Cross-family reasoning field-name and round-trip-400 hazards (how DeepSeek's `reasoning_content` differs from OpenAI/Anthropic reasoning fields) are tabulated in `resources/openai-compatibility-surface.md`. DeepSeek's own statements are kept here.

### Model IDs

| ID                  | Notes                                                                       |
|---------------------|-----------------------------------------------------------------------------|
| `deepseek-v4-flash` | 284B total / 13B active; tri-state reasoning; legacy `deepseek-chat`/`deepseek-reasoner` map here |
| `deepseek-v4-pro`   | 1.6T total / 49B active; flagship; tri-state reasoning                       |

`deepseek-chat` and `deepseek-reasoner` still resolve (mapping to V4-Flash non-thinking and thinking respectively) but hard-retire and become inaccessible after **Jul 24th, 2026, 15:59 UTC**.
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-06-01]
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]
[source: api-docs.deepseek.com/updates, retrieved 2026-06-01]

### SDKs

Any OpenAI-compatible SDK works by pointing `base_url` at `https://api.deepseek.com` (or `/beta`) and supplying a DeepSeek API key. The Anthropic interface uses `https://api.deepseek.com/anthropic`.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]

### Open weights

`deepseek-ai/DeepSeek-V4-Pro` — MIT-licensed, 1.6T total / 49B active params.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

`deepseek-ai/DeepSeek-V3.2` — MIT-licensed, 685B total parameters, DeepSeek Sparse Attention (DSA) architecture. Prior generation.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

## 2. Chat Template / Message Structure

### Special tokens (open weights)

DeepSeek-V3.2 uses full-width characters for control tokens — **not** the ASCII `<|...|>` form used by most open-weights families.

| Token                               | Purpose                                  |
|-------------------------------------|------------------------------------------|
| `<｜begin▁of▁sentence｜>`          | Beginning-of-sequence (BOS)              |
| `<｜end▁of▁sentence｜>`            | End-of-sequence (EOS)                    |
| `<｜User｜>`                        | User-turn delimiter                      |
| `<｜Assistant｜>`                   | Assistant-turn delimiter                 |
| `<｜Developer｜>`                   | Developer role (search-agent scenarios only) |
| `<think>` / `</think>`              | Reasoning block wrapper                  |

[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

Note the characters:
- `｜` is U+FF5C (FULLWIDTH VERTICAL LINE), not ASCII `|` (U+007C).
- `▁` is U+2581 (LOWER ONE EIGHTH BLOCK), not ASCII `_`.

Hand-built chat strings substituting ASCII look right on screen but tokenize differently. Use the provided encoder.

### No Jinja template

The DeepSeek V4 release ships **no Jinja-format chat template**. It provides an `encoding` folder with Python `encode_messages(...)` and `parse_message_from_completion_text(...)` helpers; use those rather than `apply_chat_template`. Recommended local sampling for the open weights: temperature=1.0, top_p=1.0.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

[applies-to: deepseek-ai/DeepSeek-V3.2] DeepSeek-V3.2 likewise ships no Jinja template; its model card directs callers to `encoding/encoding_dsv32.py` with `encode_messages()` and `parse_message_from_completion_text()`.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### Example template rendering

[applies-to: deepseek-ai/DeepSeek-V3.2]
```
<｜begin▁of▁sentence｜><｜User｜>hello<｜Assistant｜></think>Hello! I am DeepSeek.<｜end▁of▁sentence｜>
```

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

Roles: `user`, `assistant`, `tool`. OpenAI-compatible otherwise.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]

## 3. Sampling Parameters

- `frequency_penalty` and `presence_penalty` are **hard-deprecated** — they have no effect.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]

For the V4 open weights, DeepSeek recommends local sampling of temperature=1.0, top_p=1.0.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

## 4. Reasoning / Thinking Control

### Tri-state reasoning toggle

V4 exposes three reasoning levels: Non-think, Think (High), and Think Max. Set them via either parameter:

- `thinking: {"type": "enabled" | "disabled"}`, or
- `reasoning_effort: "high" | "max"`.

`reasoning_effort` values `low`/`medium` map to `high`; `xhigh` maps to `max`. Some complex agent harnesses (for example Claude Code and OpenCode) automatically set effort to `max`.

```python
response = client.chat.completions.create(
    model="deepseek-v4-flash",
    messages=[...],
    extra_body={"thinking": {"type": "enabled"}},  # or reasoning_effort="high" | "max"
)
```

[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-06-01]

Think Max is the maximum reasoning-effort mode. For it, DeepSeek recommends a context window of at least 384K tokens (a recommendation, not a hard gate); it uses a special system prompt.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

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

[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]

### Critical multi-turn rule: `reasoning_content` round-trip (V4)

On V4 the round-trip behavior depends on whether the turn performs tool calls:

- On turns that do **not** perform tool calls, passing `reasoning_content` back in `messages` is **ignored** (no error).
- On turns that **do** perform tool calls, `reasoning_content` **must** be passed back or the API returns **HTTP 400**.

```python
# Tool-call turn: reasoning_content REQUIRED on round-trip or HTTP 400
messages.append({
    "role": "assistant",
    "content": prior_response.content,
    "reasoning_content": prior_response.reasoning_content,
    "tool_calls": prior_response.tool_calls,
})

# Non-tool-call turn: reasoning_content is ignored if included
messages.append({"role": "assistant", "content": prior_response.content})
```

This inverts the legacy `deepseek-reasoner` behavior, where **any** `reasoning_content` in input returned 400. It also diverges from Claude (which requires signature-authenticated `thinking` blocks to persist in tool-use multi-turn) and OpenAI (which expects reasoning items via `previous_response_id`).
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-06-01]
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]
[testable: id=deepseek.reasoning-content-roundtrip-tool-turn.v2, expected=on V4 a tool-call turn omitting prior reasoning_content returns HTTP 400; a non-tool-call turn including it is accepted]

### `max_tokens` includes CoT

```
max_tokens >= reasoning_tokens + content_tokens
```

`max_tokens` includes the chain-of-thought portion. Budget output accordingly when thinking is enabled. Max output is 384K.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]

### Thinking with tool use

The model can interleave reasoning and tool calls within a single thinking-enabled turn:

- Emit a thought.
- Call a tool.
- Receive the result.
- Emit another thought.
- Call another tool or emit the final answer.

[source: api-docs.deepseek.com/news/news260424, retrieved 2026-06-01]

**Exception**: `deepseek-ai/DeepSeek-V3.2-Speciale` does **not** support tool calls (weights-only variant).
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2-Speciale, retrieved 2026-06-01]

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

[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

### Strict mode

Available via the **beta endpoint** (`base_url="https://api.deepseek.com/beta"`). Set `"strict": true` on each function definition. Strict mode requires:

- All object properties marked required.
- `"additionalProperties": false` on every object.

[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

### Supported JSON Schema features

Richer than OpenAI's subset:

- Basic: `object`, `string`, `number`, `integer`, `boolean`, `array`.
- `enum`, `anyOf`.
- `$def` — define reusable sub-schemas.
- `$ref` — reference them; can be used for **recursive structures**.

Recursive structures are the notable addition: data extraction over tree-shaped content (ASTs, org hierarchies, nested JSON) can be modeled directly.
[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

### Response shape

```json
{
  "choices": [{
    "message": {
      "role": "assistant",
      "content": null,
      "tool_calls": [
        {
          "id": "call_abc",
          "type": "function",
          "function": {
            "name": "get_weather",
            "arguments": "{\"location\":\"Paris\"}"
          }
        }
      ]
    }
  }]
}
```

[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

### Tool-result roundtrip

Append a `tool` role message with `tool_call_id`:

```json
{ "role": "tool", "tool_call_id": "call_abc", "content": "20°C, sunny" }
```

[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

### Notable gaps in the retrieved function-calling excerpt

- **Max tools per request** — not quoted in the retrieved primary excerpt (earlier search results suggested up to 128; not verified).
- **`tool_choice` values** — not quoted.
- **Parallel tool-call default / disable flag** — not quoted.

See §10.

## 6. Structured Outputs

DeepSeek's documented path for structured output is **through function calling with `strict: true`** on the beta endpoint. Define the extraction target as a function, force its invocation via `tool_choice`, and the strict-schema validation gives you schema-conforming `arguments`.

A dedicated `response_format: {"type": "json_schema", ...}` path (as on OpenAI) is not explicitly documented in the retrieved primary sources. Treat as partial.
[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

## 7. Caching, Batch, Streaming

### Context caching (automatic, disk-backed)

- **Always on** — no parameter needed.
- **Minimum cacheable**: **64 tokens**.
- **TTL**: "a few hours to a few days" after last use.
- **Pricing**: cache hit **0.1 yuan / MTok**, miss 1 yuan / MTok — **10× discount** (approximately 90% off).
- **Scope**: matches **prefix only** — system prompts, earlier conversation turns, initial examples.
- **Response reporting**:
  - `usage.prompt_cache_hit_tokens` — tokens served from cache.
  - `usage.prompt_cache_miss_tokens` — tokens not cached.

[source: api-docs.deepseek.com/guides/kv_cache, retrieved 2026-04-19]
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]

No `cache_control` field or explicit breakpoints — the cache operates on prefix hashing automatically. This is closer to OpenAI's automatic caching than to Anthropic's explicit `cache_control` model.

### Streaming

Standard OpenAI-compatible SSE streaming. In thinking mode, both `reasoning_content` and `content` are streamed — delta events carry them in separate fields at the same level.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]

Exact SSE event-type enumeration was not quoted in the retrieved excerpts.

### Batch

Not covered in the retrieved primary sources for this pass.

## 8. Deployment Flags (open weights)

Self-hosted deployment paths:

- **`deepseek-ai/DeepSeek-V4-Pro`** — MIT, 1.6T total / 49B active params. The release provides an `encoding` folder with `encode_messages(...)` / `parse_message_from_completion_text(...)`; required for chat-template rendering because no Jinja template is shipped. Recommended local sampling: temperature=1.0, top_p=1.0.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]
- **`deepseek-ai/DeepSeek-V3.2`** — MIT, 685B total parameters, DSA architecture. Prior generation; uses `encoding/encoding_dsv32.py`.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]
- **Community deployments**: vLLM and SGLang support DeepSeek models via their implementations; exact minimum versions for the V4 chat-encoding helper were not quoted in the retrieved primary sources. Verify against release notes at deployment time.

## 9. Deprecations and Breaking Changes

### Legacy model IDs hard-retire 2026-07-24

[applies-to: deepseek-chat, deepseek-reasoner]
`deepseek-chat` and `deepseek-reasoner` are hard-retired and inaccessible after **Jul 24th, 2026, 15:59 UTC**. Until then they map to the non-thinking and thinking modes of `deepseek-v4-flash` respectively. Migrate to explicit `deepseek-v4-flash` / `deepseek-v4-pro` IDs plus a reasoning toggle.
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-06-01]
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]
[source: api-docs.deepseek.com/updates, retrieved 2026-06-01]

### `reasoning_content` round-trip rule changed on V4

[applies-to: deepseek-v4-flash, deepseek-v4-pro]
On V4, `reasoning_content` is ignored on non-tool-call turns but **required** (or HTTP 400) on tool-call turns. This inverts the legacy `deepseek-reasoner` rule, where any input `reasoning_content` returned 400. Code that unconditionally strips `reasoning_content` will break tool-call round-trips on V4.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]

### `frequency_penalty` / `presence_penalty` hard-deprecated

`frequency_penalty` and `presence_penalty` have no effect.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]

### V3.2-Speciale API endpoint expired

[applies-to: deepseek-ai/DeepSeek-V3.2-Speciale]
Speciale is a weights-only deep-reasoning variant with no tool-calling. Its temporary API endpoint (`https://api.deepseek.com/v3.2_speciale_expires_on_20251215`) **expired 2025-12-15, 15:59 UTC** — it is no longer an active API model. The open weights remain available.
[source: api-docs.deepseek.com/updates, retrieved 2026-06-01]
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2-Speciale, retrieved 2026-06-01]

### V3.2 vs V3.1 chat template

[applies-to: deepseek-ai/DeepSeek-V3.2]
V3.2 introduced a revised chat template with new tool-calling format and the "thinking with tools" capability. Code paths built against V3.1's chat template will not render correctly. The safest path: use DeepSeek's `encode_messages()` helper from `encoding_dsv32.py`.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### License

**MIT License** (weights) — most permissive of any frontier-scale model family in this reference library. No 700M MAU clause, no regional restriction, no attribution requirement beyond MIT's standard disclaimer.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

## 10. Gaps

- **Knowledge cutoff dates** for any V3.x or V4 model are not documented in primary sources.
- **Maximum tools per request** is not quoted in the retrieved function-calling primary excerpt.
- **`tool_choice` values** (`auto`, `required`, `none`, named-function form) are not quoted.
- **Parallel tool-call default** and disable flag are not quoted.
- **Dedicated `response_format: {"type": "json_schema", ...}` parameter** — a path independent of function calling — is not documented in retrieved sources.
- **Streaming event-type enumeration** on `reasoning_content` vs `content` deltas is not quoted.
- **vLLM / SGLang minimum versions** supporting the V4 chat-encoding helper are not quoted.
- **Batch API** shape / availability is not covered.
- **`Developer` role full behavior** in search-agent scenarios is not detailed.
- **Chat Prefix Completion (Beta)** feature exists per `thinking_mode` docs but is not covered here.
- **Exact FIM (Fill-in-the-Middle) support status** — noted as Beta-unsupported in thinking mode; broader availability unclear.
