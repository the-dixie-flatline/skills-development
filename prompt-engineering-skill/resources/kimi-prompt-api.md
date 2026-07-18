---
family: kimi
scope: api
versions:
  - kimi-k3
  - kimi-k2.6
  - kimi-k2.7-code
  - kimi-k2.7-code-highspeed
retrieved: 2026-07-19
primary_sources:
  - https://platform.kimi.ai/docs/guide/kimi-k3-quickstart
  - https://platform.kimi.ai/docs/api/models-overview
  - https://platform.kimi.ai/docs/guide/use-thinking-effort
  - https://platform.kimi.ai/docs/guide/response_format
  - https://platform.kimi.ai/docs/guide/use-dynamic-tool-loading
  - https://platform.kimi.ai/docs/guide/use-official-tools
  - https://platform.kimi.ai/docs/guide/kimi-k3-tool-calling-best-practice
  - https://platform.kimi.ai/docs/guide/claude-code-kimi
  - https://platform.kimi.ai/docs/pricing/chat-k3
  - https://platform.kimi.ai/docs/models
  - https://www.kimi.com/blog/kimi-k3
  - https://huggingface.co/moonshotai
  - https://github.com/moonshotai
maturity_note: |
  Kimi K3 (`kimi-k3`, Moonshot AI) launched ~2026-07-17 and is three days old
  at this retrieval date. All "currently" / "coming soon" vendor qualifiers are
  preserved and dated 2026-07-19; treat launch-week specifics as volatile.
  Reasoning is always on; `reasoning_effort` accepts only `max` today. Five
  sampling parameters are fixed and error on deviation. Open weights are
  announced for release by 2026-07-27 but NOT yet published, so the chat-template
  / special-token / inference-stack sections are declared gaps rather than
  omissions. Portable prompt-layer content (selection, multimodal, quirks,
  anti-patterns) lives in `kimi-prompt.md`.
---

# Kimi (Moonshot AI) — API-Layer Reference

API-call-level detail for the current Kimi generation. Portable prompt-layer content (model selection, multimodal conventions, behavioral quirks, anti-patterns) lives in `kimi-prompt.md`.

## 1. API Surface

### Platform base URLs

- **OpenAI-compatible**: `https://api.moonshot.ai/v1`, model `kimi-k3`.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]
- **Anthropic-compatible**: `ANTHROPIC_BASE_URL="https://api.moonshot.ai/anthropic"`, `ANTHROPIC_MODEL="kimi-k3[1m]"`. The `/status` command surfaces both. Serves K3.
[source: platform.kimi.ai/docs/guide/claude-code-kimi, retrieved 2026-07-19]

Note the `[1m]` suffix on the Anthropic-surface model string (`kimi-k3[1m]`) — it is part of the model id on that surface, not a typo.

The cross-family OpenAI-compat divergence matrix lives in `resources/openai-compatibility-surface.md`; Kimi's own divergences (fixed sampling params, `reasoning_content` field, dynamic-tool system message, `[1m]` suffix) are stated in full below so this file stands alone.

### Kimi Code is a separate product

`api.kimi.com/coding` is a distinct, membership-billed product ("Kimi Code"), documented at `www.kimi.com/code/docs`. It exposes its own base URLs (e.g. `https://api.kimi.com/coding/v1` OpenAI-proto, `https://api.kimi.com/coding/` Anthropic-proto) and a model id `k3`. It is NOT a variant of the platform `api.moonshot.ai` surface; billing and documentation are separate. Do not conflate the two.
[source: www.kimi.com/code/docs, retrieved 2026-07-19]

### Model IDs

| ID | Notes |
|---|---|
| `kimi-k3` | Flagship; 2.8T params, KDA; 1M context; always-on reasoning |
| `kimi-k2.6` | 256K context; optional thinking |
| `kimi-k2.7-code` | Coding; own always-on thinking contract (`{"type":"enabled","keep":"all"}`) |
| `kimi-k2.7-code-highspeed` | High-throughput coding variant (~180 tok/s short-context, up to ~260 tok/s) |

[source: platform.kimi.ai/docs/models, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/api/models-overview, retrieved 2026-07-19]

## 2. Chat Template / Message Structure

### Roles

OpenAI-shaped: `system`, `user`, `assistant`, `tool`.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

### Multi-turn replay contract (load-bearing)

The vendor states verbatim: "For multi-turn conversations and tool calls, K3 requires the complete assistant message returned by the API to be passed back to `messages` as-is, including `reasoning_content` and `tool_calls`." Omitting parts of the returned assistant message causes errors or quality degradation.
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

```python
# Replay the full assistant message returned by the API, as-is.
messages.append(prior_response.choices[0].message)  # includes reasoning_content + tool_calls
```

This aligns K3 with the "preserve the full reasoning turn" camp (Claude, OpenAI reasoning models), not the "strip reasoning before replay" camp.

### Multimodal message shape

Image and video inputs require `content` to be an **array of objects**, not a string. Image references use `image_url.url` = a base64 data URI (`data:image/...;base64,...`) or an upload reference (`ms://<file-id>`). **Public HTTP/HTTPS image URLs are rejected.** Video: upload with `purpose="video"`, then reference `ms://{video.id}`.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

### Partial (prefix-continuation) mode

Append an assistant message with `partial: true` to make K3 continue from a supplied text prefix. When displaying the final result, prepend that prefix to the continuation.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

### Open-weights chat template

No chat template, special tokens, tokenizer config, or Jinja template is published for K3 at this retrieval date (weights unreleased — §8). There is no canonical encoder to cite. Do not reconstruct a token stream from memory.
[source: huggingface.co/moonshotai, retrieved 2026-07-19]
[source: github.com/moonshotai, retrieved 2026-07-19]

## 3. Sampling Parameters

Five parameters are **fixed and cannot be modified** on K3:

| Parameter | Fixed value |
|---|---|
| `temperature` | 1.0 |
| `top_p` | 0.95 |
| `n` | 1 |
| `presence_penalty` | 0 |
| `frequency_penalty` | 0 |

Passing any other value returns an error; the vendor's guidance is "do not pass it explicitly" / "Omit them from requests."
[source: platform.kimi.ai/docs/api/models-overview, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

The exact error type/label returned for sending a fixed sampling value (and whether sending the value that equals the fixed default is treated identically) is not stated verbatim on the retrieved pages. [unverified]

### `max_completion_tokens`

Defaults to **131072**, settable up to **1048576**. Output token accounting includes reasoning-trace tokens (reasoning is always on), so budget the completion window with the trace in mind.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/pricing/chat-k3, retrieved 2026-07-19]

## 4. Reasoning / Thinking Control

### Always on; controlled by `reasoning_effort`

K3 always reasons and may return a separate `reasoning_content` field alongside `content`. Reasoning is controlled by the top-level string parameter `reasoning_effort`, default **`max`**. **Only `max` is currently supported** as of 2026-07-19; the vendor states "more reasoning-effort levels are coming soon."
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]

```python
response = client.chat.completions.create(
    model="kimi-k3",
    messages=[...],
    reasoning_effort="max",   # only accepted value as of 2026-07-19; default is "max"
)
```

The K2.x `thinking` parameter must **not** be used on K3. (K2.7-code retains its own always-on thinking object, `{"type":"enabled","keep":"all"}` — a different model's contract.)
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/models, retrieved 2026-07-19]

### Response shape

```json
{
  "choices": [{
    "message": {
      "role": "assistant",
      "reasoning_content": "internal reasoning trace...",
      "content": "final answer..."
    }
  }]
}
```

[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]

### Multi-turn round-trip

Replay the complete assistant message (including `reasoning_content` and `tool_calls`) as-is on the next turn. Dropping `reasoning_content` or `tool_calls` risks errors or degraded quality. See §2.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

## 5. Tool Use / Function Calling

Standard OpenAI-shaped `tools` array with function definitions and `tool_call_id` results. Two K3-specific mechanisms extend this.

### Dynamic tool loading (K3-only)

Inject tools mid-conversation by adding a `role:"system"` message carrying a `tools` array. That system message **must NOT carry `content`** — if it does, the request returns **400** ("cannot be used with content"). It coexists with the top-level `tools` array. It is **not server-persisted**; re-send it on each request that needs the injected tools. On non-K3 models (e.g. `kimi-k2.6`), the construct fails with "tokenization failed."
[source: platform.kimi.ai/docs/guide/use-dynamic-tool-loading, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-tool-calling-best-practice, retrieved 2026-07-19]

```json
{
  "role": "system",
  "tools": [ { "type": "function", "function": { "name": "lookup", "parameters": { "...": "..." } } ]
}
```

The system message above intentionally has **no** `content` key.

### Formula official tools

Kimi ships a set of hosted "official tools" (Formula tools). Exactly 12 are documented: `convert`, `web-search`, `rethink`, `random-choice`, `mew`, `memory`, `excel`, `date`, `base64`, `fetch`, `quickjs`, `code_runner`. The `memory` tool supports persistent storage of conversation history / user preferences. Official tools are "currently free for a limited time" (as of 2026-07-19), with capacity-based rate limiting under load.
[source: platform.kimi.ai/docs/guide/use-official-tools, retrieved 2026-07-19]

Wiring:
- Fetch tool declarations: `GET /formulas/{formula_uri}/tools`.
- Execute a call: `POST /formulas/{formula_uri}/fibers` (arguments passed as a JSON string).
- The client maps `function.name` → `formula_uri`.

[source: platform.kimi.ai/docs/guide/use-official-tools, retrieved 2026-07-19]

**`web-search` caveat.** The vendor states verbatim: "The web search (web_search) is currently being updated. We do not recommend using this functionality in the near term." (as of 2026-07-19)
[source: platform.kimi.ai/docs/guide/use-official-tools, retrieved 2026-07-19]

## 6. Structured Outputs

Use `response_format` with a JSON Schema:

```json
{
  "response_format": {
    "type": "json_schema",
    "json_schema": { "name": "...", "schema": { "...": "..." }, "strict": true }
  }
}
```

Under **MFJS (Moonshot Flavored JSON Schema)**, nested objects, arrays, and `anyOf` are supported. With `strict: true`, output adheres to the schema; with `strict: false` or omitted, only valid JSON is guaranteed, not schema adherence.
[source: platform.kimi.ai/docs/guide/response_format, retrieved 2026-07-19]

**Parse only `choices[0].message.content`.** Do not `json.loads` the whole response object; `reasoning_content` sits outside the schema's scope, so deserializing the entire response will not match the schema.
[source: platform.kimi.ai/docs/guide/response_format, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

Truncation on an over-long structured response is expected to surface as `finish_reason: "length"` (standard OpenAI-shaped behavior); the exact error label for an invalid schema is not stated verbatim on the retrieved pages. [unverified]

## 7. Caching, Batch, Streaming

### Caching

Prefix caching is automatic — no cache ID and no client-side cache parameters are exposed. Pricing distinguishes cache-hit from cache-miss input (see below). Numeric cache mechanics beyond the price row (minimum cacheable prefix length, TTL/eviction, per-request hit/miss token reporting in the `usage` field) are not documented.
[source: platform.kimi.ai/docs/pricing/chat-k3, retrieved 2026-07-19]

### Pricing (K3), per 1M tokens

| Line | Price |
|---|---|
| Input, cache hit | $0.30 |
| Input, cache miss | $3.00 |
| Output | $15.00 |

Context 1,048,576. Flat pay-as-you-go, no tiering by context length. Output pricing includes reasoning-trace tokens.
[source: platform.kimi.ai/docs/pricing/chat-k3, retrieved 2026-07-19]

### Streaming

Streaming emits `reasoning_content` deltas separately from `content` deltas. Exact SSE event-type enumeration is not quoted in the retrieved sources.
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]

### Batch

Batch API shape / availability is not covered in the retrieved sources (§10).

## 8. Deployment Flags

### Anthropic-compatible (Claude Code) env vars

- `ANTHROPIC_BASE_URL="https://api.moonshot.ai/anthropic"`
- `ANTHROPIC_MODEL="kimi-k3[1m]"`
- Per-tier alias env vars (the OPUS/SONNET/HAIKU/FABLE model aliases and `CLAUDE_CODE_SUBAGENT_MODEL`) are documented on the Claude-Code integration page.
- `ENABLE_TOOL_SEARCH` (Claude Code Tool Search) is **not supported** by the Kimi endpoint.

[source: platform.kimi.ai/docs/guide/claude-code-kimi, retrieved 2026-07-19]

### Open-weights inference stack

Not applicable at this retrieval date — no weights, no chat template, no tokenizer, so vLLM / SGLang / TGI launch flags cannot be authored. Vendor weights target: by 2026-07-27 (§10).
[source: www.kimi.com/blog/kimi-k3, retrieved 2026-07-19]

## 9. Deprecations and Breaking Changes

### K2.x `thinking` removed on K3

K3 does not accept the K2.x-era `thinking` parameter; reasoning is controlled by `reasoning_effort` (only `max` supported today). Code paths that pass `thinking` to K3 must migrate.
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]

### `kimi-k2.5` and `moonshot-v1` series sunset 2026-08-31

`kimi-k2.5` and the `moonshot-v1` series are no longer available to newly registered users and reach full platform sunset on **August 31**. Migration guidance directs to `kimi-k3`.
[source: platform.kimi.ai/docs/models, retrieved 2026-07-19]

### `kimi-k2` preview series discontinued 2026-05-25

The `kimi-k2` preview series (`kimi-k2-0905-preview`, `-0711-preview`, `-turbo-preview`, `-thinking`, `-thinking-turbo`) was **discontinued May 25, 2026**.
[source: platform.kimi.ai/docs/models, retrieved 2026-07-19]

### Still active post-launch

`kimi-k2.6` (256K context, optional thinking), `kimi-k2.7-code`, and `kimi-k2.7-code-highspeed` remain available. `kimi-k2.7-code` keeps its own always-on thinking contract (`{"type":"enabled","keep":"all"}`), distinct from K3.
[source: platform.kimi.ai/docs/models, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/api/models-overview, retrieved 2026-07-19]

## 10. Gaps

- **Open weights not released.** No public repo, license file, chat template, special tokens, thinking delimiters, or tool-call wire format on HF `moonshotai` or GitHub `moonshotai` as of 2026-07-19. Vendor target: "by July 27, 2026"; technical report pending. Blocks all open-weights sections (special tokens, Jinja/encoder, vLLM/SGLang flags).
[source: www.kimi.com/blog/kimi-k3, retrieved 2026-07-19]
[source: huggingface.co/moonshotai, retrieved 2026-07-19]
[source: github.com/moonshotai, retrieved 2026-07-19]
- **K3 weights license unknown.** "Modified MIT" is third-party rumor only; no vendor license file exists yet. [community-reported]
[source: www.kimi.com/blog/kimi-k3, retrieved 2026-07-19]
- **Rate limits / 429 semantics** are not documented on the pricing / quickstart pages checked 2026-07-19. Specific 429 type names circulating in secondary sources are unverified against the canonical error page.
- **Numeric cache mechanics** beyond the price row (minimum cacheable prefix length, TTL/eviction, `usage`-field hit/miss reporting) are not enumerated.
- **Exact error surface** for contract violations (sending a fixed sampling value, sending K2 `thinking` to K3, invalid `reasoning_effort`, missing `reasoning_content` in multi-turn replay, malformed dynamic-tool system message, invalid `json_schema`) is not stated verbatim on the retrieved pages.
- **Anthropic-compat endpoint feature parity** for K3-specific surfaces (`reasoning_content` streaming, dynamic tools, strict `json_schema`, partial mode, video, Formula tools) is not documented.
- **Batch API** shape / availability is not covered.
- **Streaming SSE event-type enumeration** for `reasoning_content` vs `content` deltas is not quoted.
