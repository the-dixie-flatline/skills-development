---
family: deepseek
scope: api
versions:
  - deepseek-chat
  - deepseek-reasoner
  - deepseek-ai/DeepSeek-V3.2
  - deepseek-ai/DeepSeek-V3.2-Speciale
retrieved: 2026-04-19
primary_sources:
  - https://api-docs.deepseek.com/news/news251201
  - https://api-docs.deepseek.com/guides/reasoning_model
  - https://api-docs.deepseek.com/guides/thinking_mode
  - https://api-docs.deepseek.com/guides/function_calling
  - https://api-docs.deepseek.com/guides/kv_cache
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.2
  - https://api-docs.deepseek.com/api/create-chat-completion
maturity_note: |
  DeepSeek's API is OpenAI-compatible. Two model IDs (`deepseek-chat` and
  `deepseek-reasoner`) expose the same V3.2 weights in different modes. The
  beta endpoint at `api.deepseek.com/beta` exposes strict-mode function
  calling with richer schema features than OpenAI's subset. Open weights
  ship WITHOUT a Jinja chat template — callers must use DeepSeek's provided
  Python encoder or build their own. Context caching on disk is automatic
  and materially cheaper than most competitors.
---

# DeepSeek — API-Layer Reference

API-call-level detail for the current DeepSeek generation. Portable prompt-layer content (selection, thinking semantics, anti-patterns) lives in `deepseek-prompt.md`.

## 1. API Surface

### Endpoints

- **Stable**: `https://api.deepseek.com/v1/chat/completions` — OpenAI-compatible.
- **Beta**: `https://api.deepseek.com/beta/chat/completions` — same shape plus strict-mode tool calls and newer features in validation.

[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

### Model IDs

| ID                          | Endpoint    | Notes                                                |
|-----------------------------|-------------|------------------------------------------------------|
| `deepseek-chat`             | stable/beta | V3.2 non-thinking                                    |
| `deepseek-reasoner`         | stable/beta | V3.2 thinking; returns `reasoning_content`           |
| `deepseek-v3.2-speciale`    | API-only    | High-compute reasoning; **no tool calls**            |

[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]

### SDKs

Any OpenAI-compatible SDK works by pointing `base_url` at `https://api.deepseek.com/v1` (or `/beta`) and supplying a DeepSeek API key.

### Open weights

`deepseek-ai/DeepSeek-V3.2` — MIT-licensed, 685B total parameters, DeepSeek Sparse Attention (DSA) architecture.
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

The HF model card explicitly states:

> "This release does not include a Jinja-format chat template. Please refer to the Python code mentioned above."

Use `encoding/encoding_dsv32.py` with `encode_messages()` and `parse_message_from_completion_text()`. HuggingFace `apply_chat_template` with defaults does not produce correct V3.2 output.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### Example template rendering

```
<｜begin▁of▁sentence｜><｜User｜>hello<｜Assistant｜></think>Hello! I am DeepSeek.<｜end▁of▁sentence｜>
```

[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### API-layer message shape

```json
{
  "model": "deepseek-reasoner",
  "messages": [
    { "role": "user",      "content": "..." },
    { "role": "assistant", "content": "..." },
    { "role": "tool",      "content": "...", "tool_call_id": "..." }
  ],
  "max_tokens": 32000
}
```

Roles: `user`, `assistant`, `tool`. Optional `developer` for search-agent workflows. OpenAI-compatible otherwise.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-04-19]

## 3. Sampling Parameters

| Model                    | Accepted sampling parameters                                 | Default                    |
|--------------------------|--------------------------------------------------------------|----------------------------|
| `deepseek-chat`          | `temperature`, `top_p`, `presence_penalty`, `frequency_penalty`, `logprobs`, `top_logprobs` | T=1.0, top_p=0.95 |
| `deepseek-reasoner`      | **None of the above — all rejected**                         | n/a                        |
| `deepseek-v3.2-speciale` | Partial (not fully documented in retrieved excerpts)         | n/a                        |

[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-04-19]
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]
[testable: id=deepseek.reasoner-rejects-sampling.v1, expected=request to deepseek-reasoner with temperature=0.5 is rejected or silently ignored]

On `deepseek-reasoner`, thinking-mode outputs are deterministically optimized — the model controls sampling internally.

## 4. Reasoning / Thinking Control

### Enabling thinking

Two equivalent mechanisms:

1. **Model ID**: `"model": "deepseek-reasoner"`.
2. **Parameter**: set `thinking: {"type": "enabled"}` via `extra_body` while on `deepseek-chat`:

```python
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[...],
    extra_body={"thinking": {"type": "enabled"}},
)
```

[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-04-19]

### Response shape

Thinking responses carry two distinct fields at the message level:

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

[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]

### Critical multi-turn rule: strip `reasoning_content` before sending back

Including `reasoning_content` in any subsequent request's `messages` **returns HTTP 400**. Strip it; keep only `content`.

```python
# OK
messages.append({"role": "assistant", "content": prior_response.content})

# ERROR — returns 400
messages.append({
    "role": "assistant",
    "content": prior_response.content,
    "reasoning_content": prior_response.reasoning_content,
})
```

[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]
[testable: id=deepseek.reasoning-content-rejected-on-input.v1, expected=request with reasoning_content in any message returns HTTP 400]

This is a material divergence from Claude (which requires `thinking` blocks to persist in tool-use multi-turn) and OpenAI (which expects `reasoning_items` via `previous_response_id`). DeepSeek is the only reasoning family in this library that **forbids** send-back.

### `max_tokens` includes CoT

```
max_tokens >= reasoning_tokens + content_tokens
```

On `deepseek-reasoner`, the upper bound is **64K**; default is 32K. Budget accordingly.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]

### Thinking with tool use (new in V3.2)

V3.2 is DeepSeek's first model to integrate thinking directly into tool-use. The model can:

- Emit a thought.
- Call a tool.
- Receive the result.
- Emit another thought.
- Call another tool or emit the final answer.

All within a single thinking-enabled turn. Tool loops and reasoning interleave without requiring the caller to ping-pong between reasoning and non-reasoning model IDs.
[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]

**Exception**: `deepseek-v3.2-speciale` does **not** support tool calls — even in thinking mode.
[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]

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

No `cache_control` field or explicit breakpoints — the cache operates on prefix hashing automatically. This is closer to OpenAI's automatic caching than to Anthropic's explicit `cache_control` model.

### Streaming

Standard OpenAI-compatible SSE streaming. On `deepseek-reasoner`, both `reasoning_content` and `content` are streamed — delta events carry them in separate fields at the same level.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]

Exact SSE event-type enumeration was not quoted in the retrieved excerpts.

### Batch

Not covered in the retrieved primary sources for this pass.

## 8. Deployment Flags (open weights)

Self-hosted deployment paths from `deepseek-ai/DeepSeek-V3.2`:

- **HuggingFace weights** — 685B total parameters, DSA architecture.
- **Encoding script**: `encoding/encoding_dsv32.py` from the HF repo; required for chat-template rendering because no Jinja template is shipped.
- **Community deployments**: vLLM and SGLang both support DeepSeek-V3.x via their model implementations; exact minimum versions for V3.2 with the new chat template and thinking-with-tools were not quoted in the retrieved primary sources.

[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

[unverified] vLLM ≥ 0.10 and SGLang ≥ 0.5 support DeepSeek-V3.2 with the new chat template. Verify against release notes at deployment time.

## 9. Deprecations and Breaking Changes

### V3.2 vs V3.1 chat template

[applies-to: deepseek-ai/DeepSeek-V3.2, deepseek-chat, deepseek-reasoner]
V3.2 introduces a revised chat template with new tool-calling format and the "thinking with tools" capability. Code paths built against V3.1's chat template will not render correctly. The safest path: use DeepSeek's `encode_messages()` helper from `encoding_dsv32.py`.
[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### V3.2-Speciale tool-call restriction

[applies-to: deepseek-v3.2-speciale, deepseek-ai/DeepSeek-V3.2-Speciale]
Speciale does not support tool calls. Callers migrating from `deepseek-reasoner` to Speciale for higher-compute reasoning must remove `tools` arrays.
[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]

### `reasoning_content` send-back rejection

[applies-to: deepseek-reasoner]
Sending `reasoning_content` back in `messages` returns 400. Strip it before re-submitting. This is unique to DeepSeek in this reference library.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]

### Sampling parameter rejection on reasoner

[applies-to: deepseek-reasoner]
`temperature`, `top_p`, `presence_penalty`, `frequency_penalty`, `logprobs`, `top_logprobs` are rejected. Remove them.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-04-19]

### License

**MIT License** — most permissive of any frontier-scale model family in this reference library. No 700M MAU clause, no regional restriction, no attribution requirement beyond MIT's standard disclaimer.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

## 10. Gaps

- **Total context window size** for V3.2 is not quoted in the retrieved HF card excerpt (only the reasoner's 64K `max_tokens` is documented).
- **Active MoE parameter count** is not in the retrieved HF card excerpt (685B total confirmed; active count referenced as in the V3.2-Exp repo but not quoted).
- **Maximum tools per request** is not quoted in the retrieved function-calling primary excerpt.
- **`tool_choice` values** (`auto`, `required`, `none`, named-function form) are not quoted.
- **Parallel tool-call default** and disable flag are not quoted.
- **Dedicated `response_format: {"type": "json_schema", ...}` parameter** — a path independent of function calling — is not documented in retrieved sources.
- **Streaming event-type enumeration** on `reasoning_content` vs `content` deltas is not quoted.
- **V3.2-Speciale current availability status** — a December 15, 2025 temporary-endpoint expiration was noted at release; whether it was extended, made permanent, or deprecated is not clear in retrieved sources.
- **V4 release details** — unreleased as of 2026-04-19; leaks referenced but no confirmed date in primary sources.
- **vLLM / SGLang minimum versions** supporting V3.2's new chat template and thinking-with-tools capability are not quoted.
- **Batch API** shape / availability is not covered.
- **`Developer` role full behavior** in search-agent scenarios is not detailed.
- **Chat Prefix Completion (Beta)** feature exists per `thinking_mode` docs but is not covered here.
- **Exact FIM (Fill-in-the-Middle) support status** — noted as Beta-unsupported in thinking mode; broader availability unclear.
