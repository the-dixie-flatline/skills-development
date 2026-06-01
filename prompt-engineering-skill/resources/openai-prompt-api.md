---
family: openai
scope: api
versions:
  - gpt-5.5
  - gpt-5.5-pro
  - gpt-5.4
  - gpt-5.4-mini
  - gpt-5.4-nano
  - gpt-5.3-codex
retrieved: 2026-06-01
primary_sources:
  - https://developers.openai.com/api/docs/models
  - https://developers.openai.com/api/docs/models/gpt-5.5
  - https://developers.openai.com/api/docs/models/gpt-5.5-pro
  - https://developers.openai.com/api/docs/models/gpt-5.3-codex
  - https://developers.openai.com/api/docs/models/compare
  - https://developers.openai.com/api/docs/guides/reasoning
  - https://developers.openai.com/api/docs/api-reference/responses/create
  - https://developers.openai.com/api/docs/guides/migrate-to-responses
  - https://developers.openai.com/api/docs/guides/prompt-caching
  - https://developers.openai.com/api/docs/guides/structured-outputs
  - https://developers.openai.com/api/docs/guides/deep-research
  - https://developers.openai.com/api/docs/guides/agents/orchestration
  - https://developers.openai.com/api/docs/deprecations
  - https://developers.openai.com/codex/changelog
maturity_note: |
  GPT-5.5 (snapshot gpt-5.5-2026-04-23) is the current flagship; gpt-5.5-pro is
  the highest-compute variant (Responses API incl. Batch; streaming NOT
  supported). GPT-5.4 is retained as the cheaper coding/professional tier. The
  primary endpoint for new work is the Responses API (`POST /v1/responses`).
  The Assistants API was deprecated 2025-08-26 and is removed 2026-08-26; its
  replacement is the Responses API plus the Conversations API (state
  management). The o3-deep-research and o4-mini-deep-research model IDs, plus
  gpt-5.2-codex and earlier legacy codex IDs, shut down 2026-07-23. Some
  sections below were drawn from partial retrievals of the API reference;
  fields called out as "inferred" in retrieval are marked `[unverified]` here.
---

# OpenAI — API-Layer Reference

API-call-level detail for the current GPT-5.x generation. Portable prompt-layer content lives in `openai-prompt.md`.

## 1. API Surface

### Endpoints

- **Responses API** — `POST /v1/responses` — primary for new work.
- **Chat Completions** — `POST /v1/chat/completions` — still supported, not deprecated, not recommended for new work.
- **Realtime** — `POST /v1/realtime` plus WebSocket support (added 2026-02-23).
- **Batch** — `POST /v1/batch` — 50% discount, latency insensitive.
- **Flex Processing** — same 50% discount as Batch but runs through the Responses API. Preferred over Batch when caching is required (pre-GPT-5 models do not support caching on the Batch API).
- **Compaction** — `POST /responses/compact` — client-side compaction endpoint added 2025-12-11.
- **Conversations API** — the named state-management replacement for Assistants threads, used alongside the Responses API. Confirmed to exist as the Assistants replacement; the dedicated endpoint reference was not fetched in this pass, so the endpoint path and field schema are not stated here.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]
- **Assistants API** — deprecated 2025-08-26, removed **2026-08-26**. Migration target is the Responses API plus the Conversations API. Do not start new work here.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]

[source: developers.openai.com/api/docs/models, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]

### SDKs and model IDs

First-party SDKs: Python, TypeScript. Model IDs are bare strings on the OpenAI endpoint (`gpt-5.5`, `gpt-5.5-2026-04-23`, `gpt-5.5-pro`, `gpt-5.5-pro-2026-04-23`); Azure OpenAI uses deployment-name indirection.
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.5-pro, retrieved 2026-06-01]

## 2. Chat Template / Message Structure

There is no special-token chat template. The Responses API uses an `input` array of typed items.

### Basic request shape

```json
{
  "model": "gpt-5.5",
  "instructions": "You are a helpful assistant.",
  "input": [
    { "role": "user", "content": "..." }
  ],
  "reasoning": { "effort": "medium", "summary": "auto" },
  "tools": [ /* ... */ ],
  "text": { "format": { /* structured-output schema */ } },
  "max_output_tokens": 16000,
  "store": true
}
```

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

### `input` shapes

`input` accepts either:

- a bare string (simple case), or
- an array of typed items, each a message, prior-response output item, or tool-interaction item.

Message roles: `user`, `assistant`, `system`, `developer`. `system` / `developer` content takes precedence over `user` content within the same `input`; use `developer` for instructions that must resist user override.

Content types inside role items:

- `input_text` — plain text (`{ "type": "input_text", "text": "..." }`).
- `input_image` — `{ "type": "input_image", "image_url": "...", "detail": "low|high|auto" }` or `{ "file_id": "..." }`.
- `input_file` — `{ "type": "input_file", "file_id": "...", "file_url": "...", "file_data": "...", "filename": "..." }`.

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]

### `instructions` vs system-role content

`instructions` is a top-level field providing persistent system-level context. Role-typed `system` content inside `input` also works. The two can coexist; both participate in the precedence rule described above.
[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]

### Response shape

`output` is an array of typed items: `ResponseOutputMessage`, `FileSearchCall`, `ComputerCall`, plus tool-interaction items. Message content includes `output_text` (with optional `annotations`, `logprobs`) and `refusal`.

`usage` surfaces:
- `input_tokens`
- `output_tokens`
- `output_tokens_details.reasoning_tokens` — reasoning tokens (billed as output)
- `prompt_tokens_details.cached_tokens` — cache-read tokens

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

## 3. Sampling Parameters

`temperature`, `top_p`, and `max_output_tokens` are accepted at the top level on Responses API requests. Per-model constraints:

- **Reasoning-enabled effort levels** (`low` / `medium` / `high` / `xhigh`): sampling parameters interact with reasoning and have reduced practical effect; they are not rejected.
- **`reasoning.effort: "none"`**: sampling parameters behave as on a non-reasoning model.

Specific numeric defaults per model are not uniformly documented in the retrieved primary excerpts. Rely on per-model defaults rather than hard-coding temperature values.

[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

## 4. Reasoning / Thinking Control

### Parameter shape

```json
"reasoning": {
  "effort": "none" | "minimal" | "low" | "medium" | "high" | "xhigh",
  "summary": "auto" | "concise" | "detailed"
}
```

Per-model valid values and defaults:

| Model            | Valid effort values                          | Default   |
|------------------|----------------------------------------------|-----------|
| `gpt-5.5`        | `none`, `low`, `medium`, `high`, `xhigh`     | `medium`  |
| `gpt-5.3-codex`  | `low`, `medium`, `high`, `xhigh`             | model-specific |

[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.3-codex, retrieved 2026-06-01]
[testable: id=openai.gpt55-default-effort-medium.v1, expected=request to gpt-5.5 with no reasoning field returns usage with output_tokens_details.reasoning_tokens > 0]

Per-model effort defaults for `gpt-5.4`, `gpt-5.4-mini`, and `gpt-5.4-nano` were not re-verified in this pass; set `reasoning.effort` explicitly rather than relying on the implicit default.

### Reasoning tokens

- **Billed as output tokens** at the model's output rate.
- **Context-local**: reasoning tokens occupy context during generation and are dropped from the assistant turn after the final visible response is emitted.
- **Surfaced in response**: `usage.output_tokens_details.reasoning_tokens`.
- **Budgeting tip**: reserve a ~25K token buffer in `max_output_tokens` when experimenting with high-effort reasoning to avoid truncation.

[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

### Reasoning summaries

`reasoning.summary` produces an optional summary of the internal reasoning:

- `auto` — currently resolves to `detailed` on most models.
- `concise` — supported by some computer-use models.
- `detailed` — supported on o4-mini and similar.

Summaries appear in a `summary` array within the reasoning output items.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

### Preserving reasoning across turns

Reasoning items must be preserved across tool-call boundaries. Two paths:

- **`previous_response_id: "rs_..."`** — server-side lookup; continue from a stored response.
- **Explicit inclusion** — include all output items from the prior response (reasoning items + function-call items + function-call outputs) since the last user message in the next `input` array unchanged.

Skipping this step after a function call degrades multi-step reasoning quality. Both Responses and Chat Completions support it; Responses with `store: true` is the lowest-friction path.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]
[testable: id=openai.reasoning-items-preserved.v1, expected=multi-step tool-using tasks score lower when reasoning items are dropped from follow-up input than when preserved]

### Chat Completions limitation on GPT-5.4

[applies-to: gpt-5.4]
Function calling with `reasoning: none` is **not supported** on Chat Completions for GPT-5.4. This combination requires the Responses API. On Chat Completions, raising effort above `none` re-enables tool calling for GPT-5.4. Not re-verified for `gpt-5.5` in the 2026-06-01 pass.
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

## 5. Tool Use / Function Calling

### `tools` array

Mixes function tools (custom) and built-in server-side tools:

```json
"tools": [
  {
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "...",
      "parameters": { /* JSON Schema */ },
      "strict": true
    }
  },
  { "type": "web_search" },
  { "type": "file_search" },
  { "type": "code_interpreter" },
  { "type": "computer_use" }
]
```

Hosted tools available on `gpt-5.5`: web search, file search, code interpreter, hosted shell, computer use, MCP, and tool search. These plus the Responses API, the Conversations API (state), and Background mode constitute the hosted/server-side agent surface; Agent Builder/ChatKit is the hosted build surface.
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/guides/agents/orchestration, retrieved 2026-06-01]

### `tool_choice`

String values (`"auto"`, `"none"`, `"required"`) or an object specifying a named function. Exact shape of the object variant was not quoted verbatim in the retrieved excerpt.
[unverified] `tool_choice: "required"` forces the model to emit a tool call on every response; this is accepted on the Responses API but specific interactions with reasoning effort are not documented in the retrieved excerpts.

### Strict function tools

Set `strict: true` on a function tool to enforce schema validation on `input`. Strict mode requires `additionalProperties: false` on the schema and imposes the same subset restrictions as Structured Outputs (§6).
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

### Response shape for tool calls

Tool calls appear as output items. Function calls have shape analogous to:

```json
{
  "type": "function_call",
  "id": "fc_...",
  "call_id": "call_abc123",
  "name": "get_weather",
  "arguments": "{\"location\":\"Paris, FR\"}"
}
```

Return results via a `function_call_output` item:

```json
{
  "type": "function_call_output",
  "call_id": "call_abc123",
  "output": "20°C, sunny"
}
```

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

### Parallel tool calling

Multiple tool-call items in a single assistant output are supported across current GPT-5.x models. Return all `function_call_output` items in the next `input` array; order is not strict because `call_id` is the match key.

### Include / server-side tool invocation traces

The `include` parameter surfaces additional output detail, including server-side tool invocation traces. Relevant keys from the retrieved excerpt: `web_search_call.action.sources`, `code_interpreter_call.outputs`, `computer_call_output.output.image_url`, `file_search_call.results`.
[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]

## 6. Structured Outputs

### Request shape

```json
"text": {
  "format": {
    "type": "json_schema",
    "name": "extracted_event",
    "schema": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "date": { "type": "string" }
      },
      "required": ["name", "date"],
      "additionalProperties": false
    },
    "strict": true
  }
}
```

[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

### Schema requirements

- `additionalProperties: false` required for strict mode.
- Supported types: `string`, `number`, `integer`, `boolean`, `array`, `object`, `null`.
- Schema extensions as of 2025-02-27: email-pattern string validation, numeric/array ranges. [source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]
- "Some advanced JSON Schema features are unavailable for performance or technical reasons." Specific omissions are not exhaustively documented in the retrieved excerpt.

[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

### Response shape

Text output arrives as a JSON string in `output_text`; the SDK convenience layer (e.g. `zodTextFormat()` in TypeScript) populates `output_parsed` with the parsed object. Refusals appear as `refusal` content blocks, not as schema-conforming JSON. Parsers must handle the refusal case explicitly.
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

### Legacy JSON mode

`response_format: { "type": "json_object" }` on Chat Completions provides JSON-syntactic output without schema adherence. **Do not use for new work.** Use `text.format` with `type: "json_schema"` and `strict: true`.
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

## 7. Caching, Batch, Streaming

### Prompt caching

- **Minimum cacheable**: 1024 tokens.
- **TTL**: 5–10 minutes of inactivity, up to 1 hour. Extended 24-hour retention was documented on the GPT-5 family and `gpt-4.1` variants as of the 2026-04-18 retrieval; the model list was not re-verified for `gpt-5.5` in this pass.
- **Discount**: up to 90% off input-token cost on cache hit; latency reduced by up to 80%.
- **Cached content**: messages, images (including base64, with `detail` matching required), tool definitions, structured-output schemas.
- **Response field**: `usage.prompt_tokens_details.cached_tokens`.
- **Routing**: requests are routed to a cache node by a hash of the initial ~256-token prefix. The `prompt_cache_key` parameter overrides this to give callers explicit routing control — useful when natural prefix hashes would split cache usage across nodes.
- **Traffic shape**: keep unique prefix-key combinations below ~15 req/min per cache node to avoid cache thrash.
- **Batch API caveat**: pre-GPT-5 models do not support caching on the Batch API. For cache-sensitive workloads needing batch discounts, use Flex Processing (same 50% discount via Responses) instead of Batch.

[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

### Structured-output schema caching

The structured-output schema serves as a prefix to the system message and **is** cached as part of the cacheable prefix. Changing the schema resets the cache.
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

### Long-input surcharge

[applies-to: gpt-5.4]
Prompts exceeding **272,000 input tokens** incur **2× input** and **1.5× output** pricing. Below 272K, standard pricing applies. Not re-verified for `gpt-5.5` in the 2026-06-01 pass.
[source: developers.openai.com/api/docs/models/gpt-5.4, retrieved 2026-04-18]

### Streaming

Server-Sent Events (SSE) on the Responses API. The changelog references WebSocket mode support for the Responses API as of 2026-02-23. Exact event types emitted over SSE were not quoted verbatim in the retrieved excerpt.
[source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]

**`gpt-5.5-pro` does not support streaming.** It is available for the Responses API (including Batch) but responses cannot be streamed.
[source: developers.openai.com/api/docs/models/gpt-5.5-pro, retrieved 2026-06-01]

## 8. Deployment Flags (closed-platform routing)

There is no self-hosted path; "deployment" on OpenAI means choosing among endpoints, processing modes, and (on Azure OpenAI) regional deployments.

- **Sync vs async**: `background: true` runs a Responses request asynchronously.
- **WebSocket mode**: supported on Responses API; useful for long-running agentic sessions.
- **Flex Processing**: 50% discount via Responses API for latency-insensitive work with full caching support.
- **Batch**: 50% discount via `/v1/batch`; limited caching on pre-GPT-5 models.
- **Server-side context management**: `context_management: [{type: "...", compact_threshold: N}]` configures server-side compaction (added 2026-02-10). [unverified] exact `type` enum values not quoted verbatim in the retrieved API-reference excerpt.
- **Deep Research**: invoked via the Responses API in background mode with model `o3-deep-research` or `o4-mini-deep-research`. Requires at least one data source (web search, remote MCP, or file search with vector stores); citations are returned as `annotations`; function calling is NOT supported. These model IDs are deprecating 2026-07-23 (replacement `gpt-5.5-pro`), creating a near-term doc/model mismatch. Full detail in `resources/deep-research-agents.md`.
  [source: developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]
- **Multi-agent orchestration**: handoffs are Agents-SDK-only (Python `agents` / JS `@openai/agents`); there is no hosted multi-agent handoff API endpoint. The hosted surface is the Responses API + Conversations API + Background mode + hosted tools on `gpt-5.5`. See `resources/agent-orchestration-surfaces.md`.
  [source: developers.openai.com/api/docs/guides/agents/orchestration, retrieved 2026-06-01]

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]

## 9. Deprecations and Breaking Changes

### Reasoning effort defaults across versions

Default `reasoning.effort` differs between models: GPT-5 defaulted to `medium`, the 5.1–5.4 line moved to `none`, and `gpt-5.5` documents `medium` as default. Per-model defaults for `gpt-5.4` / `gpt-5.4-mini` / `gpt-5.4-nano` were not re-verified in this pass. Set `effort` explicitly on migration rather than relying on an implicit default.
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]

### GPT-5.4 Chat Completions limitation

[applies-to: gpt-5.4]
Function calling with `reasoning: none` is **not supported** on Chat Completions. Either raise effort or migrate to Responses API. Not re-verified for `gpt-5.5` in the 2026-06-01 pass.
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

### API and model deprecations

- **Assistants API** — deprecated 2025-08-26, removed **2026-08-26**. Migration target is the Responses API plus the Conversations API.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]
- **`o3-deep-research` / `o3-deep-research-2025-06-26` and `o4-mini-deep-research` / `o4-mini-deep-research-2025-06-26`** — shut down **2026-07-23**, replacement `gpt-5.5-pro`.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]
- **`gpt-5.2-codex` and legacy codex IDs (`gpt-5.1-codex-max`, etc.)** — shut down **2026-07-23**, replacement `gpt-5.5`.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]
- **Chat Completions** — not deprecated as of retrieval; Responses API is recommended for new work.
- **Legacy JSON mode (`response_format: {type: "json_object"}`)** — still works; use `text.format` + `json_schema` with `strict: true` for new work.

[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

### Field-name mapping (Chat Completions → Responses)

| Chat Completions                    | Responses API                          |
|-------------------------------------|----------------------------------------|
| `messages`                          | `input`                                |
| `role: "system"` in `messages`      | top-level `instructions` field         |
| `response_format: {...}`            | `text.format: {...}`                   |
| `max_tokens`                        | `max_output_tokens`                    |
| Stateful via Assistants API threads | Conversations API + `store: true` / `previous_response_id` |
| n/a                                 | `background: true`                     |

[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

## 10. Gaps

- **Full Responses API request shape** — the API-reference excerpt retrieved for this pass was partial; several fields (`tool_choice` object variants, `include` key enumeration, `context_management.type` enum, `text.format` additional type variants beyond `json_schema`) are not quoted verbatim.
- **Streaming event-type enumeration** for SSE on Responses API was not retrieved.
- **Conversations API endpoint schema** — confirmed to exist as the named Assistants replacement, but the dedicated endpoint reference (path, fields) was not fetched in this pass and is not stated here.
- **Realtime API parameter shape** is separate and not covered.
- **Batch API request/response shape** is separate; only caching interaction is covered here.
- **Azure OpenAI deployment-indirection specifics** and model-name mappings are not covered.
- **Per-model temperature / top_p defaults** are not quoted numerically.
- **Per-model reasoning defaults for `gpt-5.4` / `gpt-5.4-mini` / `gpt-5.4-nano`** and GPT-5.5 multimodal / caching / long-input details were not re-verified in the 2026-06-01 pass.
- **`gpt-5.4-nano` price and context detail** beyond "low-latency variant" were not retrieved.
- **Deep Research and the hosted agent surface** are documented in `resources/deep-research-agents.md` and `resources/agent-orchestration-surfaces.md`; the Agents SDK orchestration surface is separate from the Responses API.
