---
family: openai
scope: api
versions:
  - gpt-5.6-sol
  - gpt-5.6-terra
  - gpt-5.6-luna
  - gpt-5.5
  - gpt-5.5-pro
  - gpt-5.4
  - gpt-5.4-mini
  - gpt-5.4-nano
  - gpt-5.3-codex
retrieved: 2026-07-19
primary_sources:
  - https://developers.openai.com/api/docs/models/all
  - https://developers.openai.com/api/docs/models/gpt-5.6-sol
  - https://developers.openai.com/api/docs/models/gpt-5.6-terra
  - https://developers.openai.com/api/docs/models/gpt-5.6-luna
  - https://developers.openai.com/api/docs/models/gpt-5.5
  - https://developers.openai.com/api/docs/models/gpt-5.5-pro
  - https://developers.openai.com/api/docs/guides/latest-model
  - https://developers.openai.com/api/docs/guides/reasoning
  - https://developers.openai.com/api/docs/guides/responses-multi-agent
  - https://developers.openai.com/api/docs/guides/tools-programmatic-tool-calling
  - https://developers.openai.com/api/docs/guides/prompt-caching
  - https://developers.openai.com/api/docs/api-reference/responses/create
  - https://developers.openai.com/api/docs/guides/migrate-to-responses
  - https://developers.openai.com/api/docs/guides/structured-outputs
  - https://developers.openai.com/api/docs/guides/deep-research
  - https://developers.openai.com/api/docs/deprecations
  - https://developers.openai.com/api/docs/changelog
maturity_note: |
  GPT-5.6 (Sol / Terra / Luna) is the current flagship generation; the `gpt-5.6`
  alias routes to Sol; Terra/Luna have no generic alias. GPT-5.5 / gpt-5.5-pro and
  the GPT-5.4 line remain Active. The primary endpoint for new work is the Responses
  API (`POST /v1/responses`). GPT-5.6 adds `max` reasoning effort, a per-request
  `reasoning.mode` (standard | pro) billed at standard token rates, a
  `reasoning.context` control, an assistant-message `phase` field, Programmatic Tool
  Calling, explicit prompt-caching controls (new cache-write fee, required
  `prompt_cache_key`), and a Multi-agent orchestration beta on the Responses API.
  Lifecycle dates: o3-deep-research / o4-mini-deep-research / gpt-5.2-codex and several
  legacy snapshots shut down 2026-07-23 (imminent); a second snapshot wave (incl.
  o3-2025-04-16) shuts down 2026-12-11; dall-e-2 / dall-e-3 / Realtime Beta were removed
  2026-05-12; the Assistants API is removed 2026-08-26; Agent Builder and the legacy
  Evals platform shut down 2026-11-30. The effort enum conflicts across two live vendor
  pages (see §4). OpenAI's open-weight line (gpt-oss, gpt-oss-safeguard) is covered
  separately in `resources/gpt-oss-prompt-api.md`. Fields marked `[unverified]` were
  inferred and not quoted verbatim in the retrieved excerpts.
---

# OpenAI — API-Layer Reference

API-call-level detail for the current GPT-5.x generation. Portable prompt-layer content lives in `openai-prompt.md`.

## 1. API Surface

### Endpoints

- **Responses API** — `POST /v1/responses` — primary for new work.
- **Chat Completions** — `POST /v1/chat/completions` — still supported, not deprecated, not recommended for new work.
- **Realtime** — `POST /v1/realtime` plus WebSocket support (added 2026-02-23). Note: the Realtime API **Beta** surface was removed 2026-05-12 (see §9).
- **Batch** — `POST /v1/batch` — 50% discount, latency insensitive.
- **Flex Processing** — same 50% discount as Batch but runs through the Responses API. Preferred over Batch when caching is required (pre-GPT-5 models do not support caching on the Batch API).
- **Compaction** — `POST /responses/compact` — client-side compaction endpoint added 2025-12-11. Not supported when the Multi-agent orchestration beta is enabled (§8).
- **Conversations API** — the named state-management replacement for Assistants threads, used alongside the Responses API. The dedicated endpoint reference was not fetched in this pass, so the endpoint path and field schema are not stated here.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
- **Assistants API** — deprecated 2025-08-26, removed **2026-08-26**. Migration target is the Responses API plus the Conversations API. Do not start new work here.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]

[source: developers.openai.com/api/docs/models/all, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/changelog, retrieved 2026-07-19]

### SDKs and model IDs

First-party SDKs: Python, TypeScript. Model IDs are bare strings on the OpenAI endpoint (`gpt-5.6`, `gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`, `gpt-5.5`, `gpt-5.5-pro`); the `gpt-5.6` alias routes to `gpt-5.6-sol`. Terra and Luna have no generic alias (confirmed absence) and must be addressed by full ID. Azure OpenAI uses deployment-name indirection.
[source: developers.openai.com/api/docs/models/gpt-5.6-sol, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/models/all, retrieved 2026-07-19]

## 2. Chat Template / Message Structure

There is no special-token chat template. The Responses API uses an `input` array of typed items.

### Basic request shape

```json
{
  "model": "gpt-5.6-sol",
  "instructions": "You are a helpful assistant.",
  "input": [
    { "role": "user", "content": "..." }
  ],
  "reasoning": { "effort": "medium", "mode": "standard", "summary": "auto" },
  "tools": [ /* ... */ ],
  "text": { "format": { /* structured-output schema */ } },
  "max_output_tokens": 16000,
  "store": true
}
```

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

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

### Assistant-message `phase` field

Assistant messages carry a `phase` field with values `commentary` or `final_answer`, separating interim narration from the answer in tool-heavy Responses flows. It is **assistant-only**: "Don't add `phase` to user messages." Documentation enumerates support for `gpt-5.4` and `gpt-5.5`; support on codex snapshots and "subsequent mainline" models is not enumerated on the reasoning guide — treat as unverified beyond 5.4/5.5. When replaying conversation history manually (stateless mode), preserve `phase` on the assistant items you carry forward.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

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
- `cache_write_tokens` — tokens written to cache at the 1.25× rate on GPT-5.6+ (see §7)

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-07-19]

## 3. Sampling Parameters

`temperature`, `top_p`, and `max_output_tokens` are accepted at the top level on Responses API requests. Per-model constraints:

- **Reasoning-enabled effort levels** (`low` / `medium` / `high` / `xhigh` / `max`): sampling parameters interact with reasoning and have reduced practical effect; they are not rejected.
- **`reasoning.effort: "none"`**: sampling parameters behave as on a non-reasoning model.

Specific numeric defaults per model are not uniformly documented in the retrieved primary excerpts. Rely on per-model defaults rather than hard-coding temperature values.

[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

## 4. Reasoning / Thinking Control

### Parameter shape

```json
"reasoning": {
  "effort": "none" | "low" | "medium" | "high" | "xhigh" | "max",
  "mode": "standard" | "pro",
  "context": "auto" | "current_turn" | "all_turns",
  "summary": "auto" | "concise" | "detailed"
}
```

`minimal` appears in the effort enum on the general `guides/reasoning` page but not on the model-specific `guides/latest-model` page (see the disputed note below). `max` appears only on `guides/latest-model`.

### `reasoning.effort`

| Model            | Valid effort values                              | Default        |
|------------------|--------------------------------------------------|----------------|
| `gpt-5.6-*`      | `none`, `low`, `medium`, `high`, `xhigh`, `max`  | `medium`       |
| `gpt-5.5`        | `none`, `low`, `medium`, `high`, `xhigh`         | `medium`       |
| `gpt-5.3-codex`  | `low`, `medium`, `high`, `xhigh`                 | model-specific |

[source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.3-codex, retrieved 2026-06-01]

- **GPT-5.6 adds `max` above `xhigh`** and the vendor doc states it "defaults to `medium` in both standard and pro modes."
  [source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]
  [testable: id=openai.gpt56-default-effort-medium.v1, expected=default effort behaves like medium on a reasoning-token count] Not behaviorally reproduced; checkpoint-dependent. Native reasoning-token measurement (`output_tokens_details.reasoning_tokens`, N=5/level, 2026-07-19, native OpenAI Responses API): on `gpt-5.6-sol` the default (≈328 tokens) sits closest to `high` (353), not `medium` (262), and `low` (263) ≈ `medium`; on `gpt-5.6-luna` the default (437) sits near `medium` (464, between `low` 334 and `high` 478). The two checkpoints disagree and both ladders are compressed, so reasoning-token counts cannot behaviorally verify "defaults to medium." Keep the vendor-doc statement as a Tier-1 claim, not a behaviorally-verified fact.

- **[disputed: the effort enum differs across two live vendor pages]** `guides/latest-model` (model-specific): `none, low, medium, high, xhigh, max`. `guides/reasoning` (general): `none, minimal, low, medium, high, xhigh`. Two live vendor surfaces conflict; for GPT-5.6 author against the model-specific scale (`max`, no `minimal`). Do not silently pick one enum for all models.
  [source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]
  [source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

- **[verified: native probe corrects the live enum on `gpt-5.6-sol`]** The sol checkpoint's own API error message advertises supported values as `{none, low, medium, high, xhigh}`: `minimal` is rejected with HTTP 400 ("'minimal' is not supported with 'gpt-5.6-sol'. Supported values are: 'none', 'low', 'medium', 'high', and 'xhigh'."), while `max` is TOLERATED (HTTP 200) but does NOT appear in that advertised list — likely aliased/coerced. On sol, treat `max` as tolerated-but-unadvertised and do not send `minimal`. Verified 2026-07-19: native OpenAI Responses API.
  [source: gpt-5.6-sol API error message, native OpenAI Responses API, retrieved 2026-07-19]

Per-model effort defaults for `gpt-5.4`, `gpt-5.4-mini`, and `gpt-5.4-nano` were not re-verified in this pass; set `reasoning.effort` explicitly rather than relying on the implicit default.

### `reasoning.mode` (standard | pro) — GPT-5.6

"GPT-5.6 models support `standard` and `pro` reasoning modes"; `standard` is the default. Pro mode is a **per-request** path to the higher-compute behavior on the 5.6 base models.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

- **Billing**: "Pro mode aggregates the model work performed to produce the final answer and bills those tokens at the selected model's standard token rates." No multiplier.
  [source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]
- **Additive, not a replacement**: "Existing Pro model IDs keep their current behavior and pricing." `reasoning.mode: "pro"` does NOT deprecate the distinct `-pro` model IDs (e.g. `gpt-5.5-pro`, which remains Active). It is an additional way to reach pro-grade compute on the 5.6 base models, not a migration off the `-pro` IDs.
  [source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]
- **Independent of `reasoning.effort`**: default effort is `medium` in both standard and pro modes.
  [source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]

### `reasoning.context` (auto | current_turn | all_turns)

Controls how much prior reasoning the model may draw on. Support is "model-dependent."
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

- The response echoes the effective mode: "The response's `reasoning.context` field contains the effective mode, either `current_turn` or `all_turns`." Read the response value rather than assuming the request value took effect.
  [source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]
- **[partially verified]** `all_turns` requires replaying prior reasoning items (including any encrypted reasoning content) or supplying `previous_response_id`. Verified 2026-07-19 (native OpenAI Responses API): the `previous_response_id` path is confirmed — a 2-turn Responses chain with `previous_response_id` preserved the turn-1 reasoning item and correctly recalled a value withheld from turn 2. The manual replay path (carrying encrypted reasoning items forward without `previous_response_id`) was not exercised and remains unverified.

### Reasoning tokens

- **Billed as output tokens** at the model's output rate.
- **Context-local**: reasoning tokens occupy context during generation and are dropped from the assistant turn after the final visible response is emitted.
- **Surfaced in response**: `usage.output_tokens_details.reasoning_tokens`.
- **Budgeting tip**: reserve a token buffer in `max_output_tokens` when experimenting with high/`max` effort to avoid truncation.

[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

### Reasoning summaries

`reasoning.summary` produces an optional summary of the internal reasoning:

- `auto` — currently resolves to `detailed` on most models.
- `concise` — supported by some computer-use models.
- `detailed` — supported on o4-mini and similar.

Summaries appear in a `summary` array within the reasoning output items. **Not supported when the Multi-agent orchestration beta is enabled** (§8).
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/responses-multi-agent, retrieved 2026-07-19]

### Preserving reasoning across turns

Reasoning items must be preserved across tool-call boundaries. Two paths:

- **`previous_response_id: "rs_..."`** — server-side lookup; continue from a stored response.
- **Explicit inclusion** — include all output items from the prior response (reasoning items + function-call items + function-call outputs) since the last user message in the next `input` array unchanged. Preserve the assistant `phase` field on carried-forward assistant messages (§2).

Skipping this step after a function call degrades multi-step reasoning quality. Both Responses and Chat Completions support it; Responses with `store: true` is the lowest-friction path.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]
[testable: id=openai.reasoning-items-preserved.v1, expected=multi-step tool-using tasks score lower when reasoning items are dropped from follow-up input than when preserved]

### Chat Completions limitation on GPT-5.4

[applies-to: gpt-5.4]
Function calling with `reasoning: none` is **not supported** on Chat Completions for GPT-5.4. This combination requires the Responses API. On Chat Completions, raising effort above `none` re-enables tool calling for GPT-5.4. Not re-verified for `gpt-5.5` / `gpt-5.6`.
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

Hosted tools available on the current flagship: web search, file search, code interpreter, hosted shell, computer use, MCP, and tool search. These plus the Responses API, the Conversations API (state), and Background mode constitute the hosted/server-side agent surface.
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]

### `tool_choice`

String values (`"auto"`, `"none"`, `"required"`) or an object specifying a named function. Exact shape of the object variant was not quoted verbatim in the retrieved excerpt.
[testable: id=openai.tool-choice-required-always-calls.v1, expected=tool_choice:"required" emits a tool call even when the prompt needs no tool] Verified 2026-07-19: `tool_choice: "required"` forced a tool call on every response to a no-tool-needed prompt ("What is 2+2?") at BOTH reasoning effort none and high (10/10 each, 20/20 total; via OpenRouter→OpenAI). No reasoning-effort exception was observed — high-effort reasoning did not route around "required".

### Strict function tools

Set `strict: true` on a function tool to enforce schema validation on `input`. Strict mode requires `additionalProperties: false` on the schema and imposes the same subset restrictions as Structured Outputs (§6).
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

### Programmatic Tool Calling (GPT-5.6)

"Programmatic Tool Calling lets a model write and run JavaScript that coordinates the tools in a Responses API request. A program can call tools in parallel, use loops and conditions, and keep intermediate results in the hosted runtime."
[source: developers.openai.com/api/docs/guides/tools-programmatic-tool-calling, retrieved 2026-07-19]

- **Runtime**: each generated program runs in "a fresh, isolated V8 runtime" supporting JavaScript with top-level `await`, but with no Node.js, no package installation, no direct network access, no general-purpose filesystem, no subprocess execution, no console, and no persistent JavaScript state between executions.
- **Enable it** by adding a hosted `programmatic_tool_calling` tool plus a per-tool `allowed_callers` gate:

```json
[
  {
    "type": "function",
    "name": "get_inventory",
    "parameters": { "...": "..." },
    "output_schema": { "...": "..." },
    "allowed_callers": ["programmatic"]
  },
  { "type": "programmatic_tool_calling" }
]
```

`allowed_callers` values: omitted or `["direct"]` — model calls the tool directly; `["programmatic"]` — only code in a `program` item may call it; `["direct", "programmatic"]` — both. Supported tool types for `["programmatic"]`: `function` and `custom`, `mcp`, `apply_patch`, local and hosted `shell`, `code_interpreter`.

- **New response item types**: `program` (generated JS, `call_id`, opaque `fingerprint` for replay); a `function_call` nested under a program (its `caller.caller_id` matches the program's `call_id`); `program_output` (final result plus `status`: `completed` | `incomplete`).
- **ZDR**: "Programmatic Tool Calling supports Zero Data Retention (ZDR) workflows without requiring a persistent code-execution container." ZDR must be enabled for the org/project; `store: false` enables stateless continuation but does not enable ZDR by itself.
- Not beta-flagged in the docs text (no `betas=[...]` requirement shown), but gate it on GPT-5.6 per "Check the model page before enabling Programmatic Tool Calling."

[source: developers.openai.com/api/docs/guides/tools-programmatic-tool-calling, retrieved 2026-07-19]

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
  "output": "20C, sunny"
}
```

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]

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

### Prompt caching — GPT-5.6 generation break

GPT-5.6 changes caching semantics from prior generations. Author for the target model's generation explicitly.
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-07-19]

- **Cache-write fee (new on GPT-5.6+)**: "Cache writes have no additional fee on models before the GPT-5.6 family. For GPT-5.6 models and later model families, cache writes cost 1.25× the uncached input token rate. On these models, both implicit and explicit caching are more consistent and reliable." Written tokens are reported in a new `usage.cache_write_tokens` field.
- **`prompt_cache_key` now required for reliable matching (GPT-5.6+)**: "On GPT-5.6 models and later model families, you must set `prompt_cache_key` to use the more reliable matching for both implicit and explicit caching." Without a key, requests may still get automatic hits but do not use the improved matching. Keep total traffic across all prefixes for each key to ~15 requests per minute.
- **Explicit breakpoints (GPT-5.6+)**: mark the end of a reusable prefix with an explicit cache breakpoint, available in both the Responses API and Chat Completions.
  - Request-wide mode via `prompt_cache_options.mode`: `implicit` (default) places a breakpoint on the latest message and also uses any explicit breakpoints; `explicit` disables the implicit breakpoint so only explicit breakpoints are used (with no explicit breakpoints, the request does not use prompt caching or incur cache-write charges).
  - Per-block marker: `prompt_cache_breakpoint: { "mode": "explicit" }`, supported on Responses `input_text` / `input_image` / `input_file` blocks and Chat Completions `text` / `image_url` / `input_audio` / `file` / `refusal` blocks. Only `explicit` is valid for `prompt_cache_breakpoint.mode`; a marker on an unsupported or non-cacheable block returns `400 invalid_request_error`.
  - Write slots: up to four new cache writes per request. In `implicit` mode the latest-message breakpoint uses one slot (so up to the latest three explicit breakpoints are written); in `explicit` mode up to the latest four explicit breakpoints are written. Reads consider up to the latest 50 breakpoints.
- **Retention control split by generation**: on GPT-5.6+, `prompt_cache_options.ttl` sets a minimum cache lifetime (only supported value `30m`, also the default; a cached prefix stays eligible for at least 30 minutes and may be retained longer). On earlier models, `prompt_cache_retention` selects a maximum-retention policy; `prompt_cache_retention` is **deprecated for GPT-5.6 and later**. Older models reject `prompt_cache_options` / `prompt_cache_breakpoint` — continue using their existing automatic caching.

[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-07-19]

### Prompt caching — prior-generation behavior (pre-5.6)

- **Minimum cacheable**: 1024 tokens.
- **TTL**: 5-10 minutes of inactivity, up to 1 hour. Extended 24-hour retention was documented on the GPT-5 family and `gpt-4.1` variants as of the 2026-04-18 retrieval.
- **Discount**: up to 90% off input-token cost on cache hit; latency reduced by up to 80%.
- **Cached content**: messages, images (including base64, with `detail` matching required), tool definitions, structured-output schemas.
- **Response field**: `usage.prompt_tokens_details.cached_tokens`.
- **Routing**: requests are routed to a cache node by a hash of the initial ~256-token prefix. The `prompt_cache_key` parameter overrides this to give callers explicit routing control.
- **Batch API caveat**: pre-GPT-5 models do not support caching on the Batch API. For cache-sensitive workloads needing batch discounts, use Flex Processing (same 50% discount via Responses) instead of Batch.

[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

### Structured-output schema caching

The structured-output schema serves as a prefix to the system message and **is** cached as part of the cacheable prefix. Changing the schema resets the cache.
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

### Long-input surcharge

[applies-to: gpt-5.4]
Prompts exceeding **272,000 input tokens** incur **2× input** and **1.5× output** pricing. Below 272K, standard pricing applies. Not re-verified for `gpt-5.5` / `gpt-5.6` in this pass.
[source: developers.openai.com/api/docs/models/gpt-5.4, retrieved 2026-04-18]

### Streaming

Server-Sent Events (SSE) on the Responses API. The changelog references WebSocket mode support for the Responses API as of 2026-02-23. Exact event types emitted over SSE were not quoted verbatim in the retrieved excerpt.
[source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]

**`gpt-5.5-pro` does not support streaming.** It is available for the Responses API (including Batch) but responses cannot be streamed. Streaming support on the 5.6 tiers was not re-verified this pass.
[source: developers.openai.com/api/docs/models/gpt-5.5-pro, retrieved 2026-06-01]

## 8. Deployment Flags (closed-platform routing)

There is no self-hosted path; "deployment" on OpenAI means choosing among endpoints, processing modes, and (on Azure OpenAI) regional deployments.

- **Sync vs async**: `background: true` runs a Responses request asynchronously.
- **WebSocket mode**: supported on Responses API; useful for long-running agentic sessions.
- **Flex Processing**: 50% discount via Responses API for latency-insensitive work with full caching support.
- **Batch**: 50% discount via `/v1/batch`; limited caching on pre-GPT-5 models.
- **Server-side context management**: `context_management: [{type: "...", compact_threshold: N}]` configures server-side compaction (added 2026-02-10). The exact `type` enum values were not quoted verbatim in the retrieved API-reference excerpt, and native probing rules out the previously-guessed values. Verified 2026-07-19 (native OpenAI Responses API): `context_management` type `"compact"` and type `"summarize"` are both REJECTED with HTTP 400 ("Unsupported context_management type: 'compact'" / "'summarize'"), and a top-level `phase` field on the request is REJECTED with HTTP 400 ("Unknown parameter: 'phase'"). Do not assert these values — the correct `type` enum (if any), `compact_threshold` scope, and any top-level `phase` usage remain unknown.

### Deep Research

Deep Research is invoked via the Responses API in background mode. The `guides/deep-research` page still documents model IDs `o3-deep-research` / `o4-mini-deep-research`, which **shut down 2026-07-23** (replacement `gpt-5.5-pro`) — a confirmed doc/model mismatch. Requires at least one data source (web search, remote MCP, or file search with vector stores); citations are returned as `annotations`; function calling is NOT supported. Full detail in `resources/deep-research-agents.md`.
[source: developers.openai.com/api/docs/guides/deep-research, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]

- **Successor invocation (changelog-only)**: "Added `return_token_budget` for the Responses API web search tool. Use it to opt in to longer GPT-5+ reasoning web search runs for high-effort research and evaluation workloads." This is the forward path once the dedicated `*-deep-research` IDs retire; it is documented only in the changelog, and `guides/deep-research` has not yet been updated to reference it.
  [source: developers.openai.com/api/docs/changelog, retrieved 2026-07-19]

### Multi-agent orchestration (Responses API beta)

A hosted multi-agent/handoff surface exists in beta — the prior "no hosted multi-agent handoff API endpoint" statement is retracted. "Multi-agent lets a model spin up and coordinate subagents in parallel... available as a beta feature with all GPT-5.6 models."
[source: developers.openai.com/api/docs/guides/responses-multi-agent, retrieved 2026-07-19]

- **Beta gate**: use `client.beta.responses` and pass `responses_multi_agent=v1` in the `betas` argument; for raw HTTP / WebSocket, pass header `OpenAI-Beta: responses_multi_agent=v1`. Item schemas may change while in beta.
- **Request shape**: a `multi_agent` object on the request, e.g. `{ "enabled": true, "max_concurrent_subagents": 3 }`. `max_concurrent_subagents` bounds concurrently-active subagents across the whole tree (excluding the root); no fixed upper bound; default `3` (recommended). No fixed limit on tree depth or total subagents.
- **Architecture**: a root agent (`/root`) spawns subagents at hierarchical paths (`/root/researcher`, etc.) via six hosted "collaboration actions" surfaced as `multi_agent_call` items the model — not the application — handles: `spawn_agent`, `send_message`, `followup_task`, `wait_agent`, `interrupt_agent`, `list_agents`. Your application should not execute these or submit outputs for them.
- **New output item types**: `multi_agent_call`, `multi_agent_call_output`, `agent_message` (the last carries `encrypted_content` between agents); each carries an `agent` attribute identifying `agent.agent_name`.
- **Transport**: HTTP and WebSocket; WebSocket recommended for tool-heavy / long-running workflows via a `response.inject` event that streams function outputs into an in-flight response.
- **Documented limitations**: the `/responses/compact` endpoint is not supported (server-side compaction is enabled implicitly when `multi_agent.enabled` is true); `reasoning.summary` is not supported; `max_tool_calls` is not supported; `max_concurrent_subagents` defaults to `3`.

[source: developers.openai.com/api/docs/guides/responses-multi-agent, retrieved 2026-07-19]

The Agents SDK (Python `agents` / JS `@openai/agents`) remains a client-side orchestration option; see `resources/agent-orchestration-surfaces.md`.

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/changelog, retrieved 2026-07-19]

## 9. Deprecations and Breaking Changes

### Reasoning effort defaults across versions

Default `reasoning.effort` differs between models: GPT-5 defaulted to `medium`, the 5.1-5.4 line moved to `none`, and `gpt-5.5` / `gpt-5.6` document `medium`. Per-model defaults for `gpt-5.4` / `gpt-5.4-mini` / `gpt-5.4-nano` were not re-verified in this pass. Set `effort` explicitly on migration.
[source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]

### GPT-5.4 Chat Completions limitation

[applies-to: gpt-5.4]
Function calling with `reasoning: none` is **not supported** on Chat Completions. Either raise effort or migrate to Responses API. Not re-verified for `gpt-5.5` / `gpt-5.6`.
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

### Prompt-caching field changes (GPT-5.6+)

- `prompt_cache_key` is required for reliable matching (§7).
- `prompt_cache_retention` (max-retention policy) is **deprecated for GPT-5.6 and later**; use `prompt_cache_options.ttl` (only value `30m`) instead.
- New cache-write fee (1.25× uncached input rate) and new `usage.cache_write_tokens` field.
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-07-19]

### API and model deprecations

Two dated snapshot-removal announcements are live; they cover disjoint model-ID sets.

- **Snapshot wave — shutdown 2026-07-23** (announced 2026-04-22, imminent as of retrieval): `computer-use-preview` / `-2025-03-11` (→ `gpt-5.4-mini`), `gpt-4o-mini-search-preview-2025-03-11` (→ `gpt-5.4-mini`), `gpt-4o-mini-tts-2025-03-20` (→ `gpt-4o-mini-tts-2025-12-15`), `gpt-4o-search-preview-2025-03-11` (→ `gpt-5.4-mini`), `gpt-5-chat-latest` / `gpt-5-codex` / `gpt-5.1-chat-latest` / `gpt-5.1-codex` / `gpt-5.1-codex-max` (→ `gpt-5.5`), `gpt-5.1-codex-mini` (→ `gpt-5.4-mini`), `gpt-audio-mini-2025-10-06` (→ `gpt-audio-1.5`), `gpt-realtime-mini-2025-10-06` (→ `gpt-realtime-mini`), **`o3-deep-research-2025-06-26` / `o3-deep-research` and `o4-mini-deep-research-2025-06-26` / `o4-mini-deep-research`** (→ `gpt-5.5-pro`), **`gpt-5.2-codex`** (→ `gpt-5.5`).
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
- **Snapshot wave — shutdown 2026-12-11** (announced 2026-06-11): `gpt-5-2025-08-07` (→ `gpt-5.5`), `gpt-5-mini-2025-08-07` (→ `gpt-5.4-mini`), `gpt-5-nano-2025-08-07` (→ `gpt-5.4-nano`), `gpt-5-pro-2025-10-06` (→ `gpt-5.5-pro`), **`o3-2025-04-16`** (→ `gpt-5.5`), `o3-pro-2025-06-10` (→ `gpt-5.5-pro`). Note: the plain `o3-2025-04-16` snapshot carries the Dec 11 date, distinct from the deep-research `o3-deep-research-*` IDs above which retire 2026-07-23.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
- **Completed removal 2026-05-12**: `dall-e-2`, `dall-e-3`, and the Realtime API **Beta** surface.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
- **Assistants API** — deprecated 2025-08-26, removed **2026-08-26**. Migration target: Responses API plus Conversations API.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
- **Agent Builder** — deprecated 2026-06-03, scheduled shutdown **2026-11-30**; ChatKit remains available. Migration: Agents SDK or ChatGPT Workspace Agents (see "Migrate from Agent Builder"). Its Legacy-APIs docs remain published pending shutdown.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
- **Legacy Evals platform** — deprecated 2026-06-03; existing evals become read-only **2026-10-31**; dashboard and API shut down **2026-11-30**. Migration target: Promptfoo.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
- **Reusable prompts (`v1/prompts` and reusable prompt objects)** — deprecated 2026-06-03, scheduled shutdown **2026-11-30**; move reusable prompt content into application code (see "Migrate from prompt objects"). Same June-2026 announcement wave as Agent Builder and Evals.
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
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

- **Effort-enum two-surface conflict** — `guides/latest-model` (`max`, no `minimal`) vs `guides/reasoning` (`minimal`, no `max`). Documented as disputed in §4; not resolvable from the vendor pages as of 2026-07-19.
- **`phase` model scope** — enumerated only for `gpt-5.4` / `gpt-5.5` on the reasoning guide; codex snapshots and "subsequent mainline" support are not documented at `developers.openai.com/api/docs/guides/reasoning`, checked 2026-07-19.
- **`reasoning.context` `all_turns` replay mechanics** — exact requirement (replayed reasoning items incl. encrypted content, or `previous_response_id`) is inferred from reasoning-preservation docs, not re-quoted verbatim; marked `[unverified]` in §4.
- **Full Responses API request shape** — several fields (`tool_choice` object variants, full `include` key enumeration, `context_management.type` enum) were not quoted verbatim in this pass.
- **Streaming event-type enumeration** for SSE on Responses API was not retrieved.
- **Conversations API endpoint schema** — confirmed to exist as the named Assistants replacement, but the dedicated endpoint reference (path, fields) was not fetched and is not stated here.
- **Multi-agent orchestration** item schemas may change while in beta (§8); re-verify before depending on exact shapes.
- **GPT-5.6 per-tier caching/long-input/streaming behavior** was not re-verified against the per-model pages beyond §7 generational notes.
- **Per-model temperature / top_p defaults** are not quoted numerically.
- **Realtime API parameter shape**, **Batch API request/response shape**, and **Azure OpenAI deployment-indirection specifics** are separate and not covered here.
- **OpenAI's open-weight line (gpt-oss, gpt-oss-safeguard)** is covered separately in `resources/gpt-oss-prompt.md` / `resources/gpt-oss-prompt-api.md`.
