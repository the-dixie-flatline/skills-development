---
family: claude
scope: api
versions:
  - claude-opus-4-7
  - claude-sonnet-4-6
  - claude-haiku-4-5
  - claude-haiku-4-5-20251001
retrieved: 2026-04-18
primary_sources:
  - https://platform.claude.com/docs/en/about-claude/models/overview
  - https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7
  - https://platform.claude.com/docs/en/build-with-claude/extended-thinking
  - https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking
  - https://platform.claude.com/docs/en/build-with-claude/effort
  - https://platform.claude.com/docs/en/build-with-claude/prompt-caching
  - https://platform.claude.com/docs/en/build-with-claude/structured-outputs
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
maturity_note: |
  Claude Opus 4.7 introduces substantial API-layer breaking changes from Opus
  4.6: sampling parameters are rejected, manual thinking budgets are
  rejected, thinking content is omitted from responses by default, and a new
  tokenizer changes token counts. This file treats those changes as first-class
  migration concerns. Files API, Citations, MCP connector, and Managed Agents
  are out of scope; see Anthropic docs for those features.
---

# Claude — API-Layer Reference

API-call-level detail for the current Claude 4.x generation. Portable prompt-layer content lives in `claude-prompt.md`.

## 1. API Surface

### Endpoints and SDKs

The Messages API (`POST /v1/messages`) is the primary endpoint. Anthropic provides first-party SDKs in Python, TypeScript, C#, Go, Java, PHP, and Ruby.
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]

The Message Batches API (`/v1/messages/batches`) supports asynchronous batching at discounted pricing. Extended output tokens up to **300K** are available on Opus 4.7, Opus 4.6, and Sonnet 4.6 via the `output-300k-2026-03-24` beta header.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

### Platforms and model IDs

| Platform           | Opus 4.7 ID                   | Sonnet 4.6 ID                  | Haiku 4.5 ID                                      |
|--------------------|-------------------------------|--------------------------------|---------------------------------------------------|
| Claude API         | `claude-opus-4-7`             | `claude-sonnet-4-6`            | `claude-haiku-4-5-20251001` (alias `claude-haiku-4-5`) |
| Amazon Bedrock     | `anthropic.claude-opus-4-7`   | `anthropic.claude-sonnet-4-6`  | `anthropic.claude-haiku-4-5-20251001-v1:0`        |
| Google Vertex AI   | `claude-opus-4-7`             | `claude-sonnet-4-6`            | `claude-haiku-4-5@20251001`                       |

[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

Bedrock exposes **global** (dynamic-routing) and **regional** endpoints from Sonnet 4.5 onward. Vertex AI exposes **global**, **multi-region**, and **regional** endpoints. Claude is also available on Microsoft Foundry.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

The Models API (`GET /v1/models`) returns `max_input_tokens`, `max_tokens`, and a `capabilities` object per model, programmatically.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

## 2. Chat Template / Message Structure

Claude's request shape:

```json
{
  "model": "claude-opus-4-7",
  "max_tokens": 16000,
  "system": "...",
  "messages": [
    { "role": "user", "content": "..." },
    { "role": "assistant", "content": [ /* content blocks */ ] }
  ]
}
```

`system` is a top-level field (string or content-block array), not a message role. `max_tokens` is required.
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]

Content-block types on assistant output include `text`, `thinking`, `tool_use`. On user input they include `text`, `image`, `document`, `tool_result`. Thinking blocks carry `thinking` (summarized text or empty) and `signature` (encrypted full thinking, used for multi-turn continuity).
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]

### Prefilled assistant messages

Prefilled content on the **last** assistant turn is **deprecated on Claude 4.6 and later**; Claude Mythos Preview rejects it with 400. Prefills placed elsewhere in the conversation continue to work. Migrate to Structured Outputs (§6) or system-prompt directives.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 3. Sampling Parameters

[applies-to: claude-opus-4-7]
`temperature`, `top_p`, and `top_k` are **rejected (400 error)** when set to any non-default value. Omit these parameters entirely. If you were using `temperature = 0` for determinism, note that it never produced identical outputs on any Claude model.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]
[testable: id=claude.opus47-sampling-rejected.v1, expected=request to claude-opus-4-7 with temperature=0.5 returns HTTP 400]

On Opus 4.6, Sonnet 4.6, and Haiku 4.5 these parameters remain accepted, but `output_config.effort` is the recommended control for response characteristics.
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-04-18]

## 4. Reasoning / Thinking Control

### The `thinking` field

```json
"thinking": {
  "type": "enabled" | "disabled" | "adaptive",
  "budget_tokens": N,
  "display": "summarized" | "omitted"
}
```

- `type: "adaptive"` — model decides when and how much to think. Required mode on Opus 4.7.
- `type: "enabled"` with `budget_tokens` — manual budget. Rejected on Opus 4.7 (400), deprecated on Opus 4.6 and Sonnet 4.6, still supported on Opus 4.5 / Sonnet 4.5 and older.
- `type: "disabled"` or field omitted — no thinking.
- `budget_tokens` must be `< max_tokens` in manual mode (except with interleaved thinking).
- `display` controls whether the `thinking` field on returned thinking blocks carries summarized reasoning or is empty. Defaults: `"summarized"` on Opus 4.6, Sonnet 4.6, Haiku 4.5, earlier Claude 4; `"omitted"` on Opus 4.7 and Mythos Preview.

[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]
[testable: id=claude.opus47-manual-thinking-rejected.v1, expected=request to claude-opus-4-7 with thinking.type="enabled" returns HTTP 400]
[testable: id=claude.opus47-display-default-omitted.v1, expected=request to claude-opus-4-7 with thinking.type="adaptive" and no display field returns thinking blocks with empty thinking strings]

### `output_config.effort`

```json
"output_config": {
  "effort": "max" | "xhigh" | "high" | "medium" | "low"
}
```

| Level   | Availability                                               | Guidance                                                                                 |
|---------|------------------------------------------------------------|------------------------------------------------------------------------------------------|
| `max`   | Mythos Preview, Opus 4.7, Opus 4.6, Sonnet 4.6             | Frontier problems only; often overthinks on structured tasks                             |
| `xhigh` | Opus 4.7                                                   | Recommended starting point for coding and agentic workloads on Opus 4.7                  |
| `high`  | All models supporting effort; API default                  | Strong quality balance; Opus 4.7 recommendation for intelligence-sensitive workloads      |
| `medium`| All models supporting effort                               | Cost-sensitive workloads; recommended default for Sonnet 4.6                             |
| `low`   | All models supporting effort                               | Short, scoped tasks; latency-sensitive subagents                                         |

[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-04-18]

Setting `effort: "high"` is identical to omitting the parameter. At `high` and above on adaptive-capable models, Claude almost always thinks; at `low` / `medium` it may skip thinking on simple queries.
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-04-18]

### `output_config.task_budget` (beta)

[applies-to: claude-opus-4-7]
Beta header: `task-budgets-2026-03-13`. Shape:

```json
"output_config": {
  "task_budget": { "type": "tokens", "total": 128000 }
}
```

Advisory budget across the full agentic loop (thinking + tool calls + tool results + final output). The model sees a running countdown and uses it to prioritize; it is **not** a hard cap (that is `max_tokens`). Minimum value: 20K tokens. For open-ended agentic tasks where quality matters more than speed, omit `task_budget`.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]

### Interleaved thinking

Adaptive thinking **automatically enables interleaved thinking** — Claude can reason between tool calls. On manual thinking with Sonnet 4.6, interleaved thinking requires the `interleaved-thinking-2025-05-14` beta header; on Opus 4.6 manual thinking, interleaved is not available.
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]

### Preserving thinking blocks in multi-turn tool use

When extended/adaptive thinking is active and tool calls are involved, pass the prior assistant turn's **thinking blocks back unchanged** alongside the `tool_use` block when sending `tool_result`. Omitting them in tool loops raises an error. The `signature` field authenticates the thinking; any text inside a round-tripped `thinking` block with `display: "omitted"` is ignored (server decrypts the signature). Signatures are cross-platform compatible (Claude API, Bedrock, Vertex AI).
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]

[testable: id=claude.thinking-tool-loop-preservation.v1, expected=omitting thinking blocks from the prior assistant turn in a tool_result continuation returns HTTP 400 or downgrades the thinking mode silently]

### `tool_choice` interaction with thinking

When thinking (extended or adaptive) is active, `tool_choice` must be `{type: "auto"}` or unset. `{type: "any"}` and `{type: "tool", name: "..."}` are not supported. The API applies graceful degradation if thinking and `tool_choice` are incompatible: it auto-disables thinking rather than returning an error.
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]

## 5. Tool Use / Function Calling

### Request shape

```json
"tools": [
  {
    "name": "get_weather",
    "description": "Get current weather for a location.",
    "input_schema": {
      "type": "object",
      "properties": { "location": { "type": "string" } },
      "required": ["location"],
      "additionalProperties": false
    },
    "strict": true
  }
],
"tool_choice": { "type": "auto" }
```

`tool_choice` options: `{type: "auto"}` (default), `{type: "any"}`, `{type: "tool", name: "..."}`, `{type: "none"}`.

Set `strict: true` on a tool to guarantee that Claude's `input` payload conforms to the `input_schema` exactly. Strict mode requires `additionalProperties: false` on the schema.
[source: platform.claude.com/docs/en/agents-and-tools/tool-use/overview, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-04-18]

### Response shape

When Claude calls a tool, the response carries `stop_reason: "tool_use"` and a `tool_use` content block:

```json
{
  "type": "tool_use",
  "id": "toolu_01A09q90qw...",
  "name": "get_weather",
  "input": { "location": "Paris, FR" }
}
```

Return results as a `tool_result` content block in the next user message:

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01A09q90qw...",
  "content": "20°C, sunny",
  "is_error": false
}
```

`content` accepts string or an array of text/image blocks. `is_error: true` signals tool failure; Claude adjusts accordingly.
[source: platform.claude.com/docs/en/agents-and-tools/tool-use/overview, retrieved 2026-04-18]

### Parallel tool use

All current 4.x models emit multiple `tool_use` blocks in a single assistant turn when tasks are independent. Prompting can raise this to ~100% when needed; see `claude-prompt.md` §3 for prompt patterns.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Client vs server tools

- **Client tools** run in the caller's application: user-defined tools plus Anthropic-schema tools like `bash`, `text_editor`, and `memory`.
- **Server tools** run in Anthropic's infrastructure: `web_search`, `code_execution`, `web_fetch`, `tool_search`. Server tool types are **versioned** (e.g. `web_search_20260209`). Check the tool reference for the current versioned type string before use.

[source: platform.claude.com/docs/en/agents-and-tools/tool-use/overview, retrieved 2026-04-18]

### Tool-use system-prompt overhead

When a request includes `tools`, a tool-use system prompt is added automatically. Token cost (current 4.x models):

- `tool_choice: auto | none` → **346 tokens**
- `tool_choice: any | tool` → **313 tokens**

No `tools` field + `tool_choice: none` → 0 additional tokens.
[source: platform.claude.com/docs/en/agents-and-tools/tool-use/overview, retrieved 2026-04-18]

## 6. Structured Outputs

```json
"output_config": {
  "format": {
    "type": "json_schema",
    "schema": {
      "type": "object",
      "properties": { "...": { "type": "string" } },
      "required": ["..."],
      "additionalProperties": false
    }
  }
}
```

- **Parameter:** `output_config.format`. (The earlier `output_format` parameter and `structured-outputs-2025-11-13` beta header are deprecated but still accepted during transition.)
- **Required schema constraint:** `additionalProperties: false` on every object.
- **Supported models:** Opus 4.7, Opus 4.6, Sonnet 4.6, Sonnet 4.5, Opus 4.5, Haiku 4.5, Mythos Preview (Claude API). No beta header required.
- **Platform support:** Claude API full; Bedrock supports 4.6 and earlier current-gen directly, Opus 4.7 via Messages-API Bedrock endpoint; Microsoft Foundry beta; Vertex AI does not support Mythos Preview.
- **Combined with tool use:** Max 20 strict tools per request, 24 optional parameters total across schemas, 16 parameters with `anyOf` unions.
- **Schema violation:** returns HTTP 400 (rare due to constrained decoding). Safety refusals return `stop_reason: "refusal"` with status 200; output may not match schema in that case. `stop_reason: "max_tokens"` truncation may also produce non-conforming output.

[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-04-18]

### Supported JSON Schema subset

Supported: `object`, `array`, `string`, `integer`, `number`, `boolean`, `null`, `enum`, `const`, `anyOf`, `allOf`, `$ref`, `$def`, `required`, `additionalProperties: false`, string formats (`date-time`, `date`, `time`, `duration`, `email`, `hostname`, `uri`, `ipv4`, `ipv6`, `uuid`), `minItems` (0 or 1 only), simple `pattern` regex.

Not supported: recursive schemas, complex enum types, external `$ref` URLs, numeric constraints (`minimum`, `maximum`, `multipleOf`), string length constraints (`minLength`, `maxLength`), most array constraints beyond `minItems`, `additionalProperties: true`.
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-04-18]

## 7. Caching, Batch, Streaming

### Prompt caching

```json
"cache_control": { "type": "ephemeral", "ttl": "5m" | "1h" }
```

- **Placement:** on individual blocks in `tools`, `system`, or `messages[].content`; or at request level for automatic caching.
- **Minimum cacheable tokens:** Opus 4.7, Opus 4.6, Opus 4.5 = **4096**; Sonnet 4.6 = **2048**; Sonnet 4.5, Sonnet 3.7, earlier Sonnet 4.x = **1024**; Haiku 4.5 = **4096**; Haiku 3.5 / 3 = 2048.
- **Breakpoints:** max 4 explicit per request (1 slot consumed by automatic caching if used together).
- **Cost model:** cache write = 1.25× base input (5m TTL) or 2.0× base input (1h TTL). Cache read = 0.1× base input for both TTLs.
- **Thinking blocks** cannot carry explicit `cache_control` but are cached alongside content in tool-use loops; they count as input tokens on cache read.
- **Cache invalidation:** `tool_choice` change, images added/removed, thinking parameter change, and non-tool-result user content in a thinking-enabled conversation all bump messages cache. Tools and system cache survive most message-level changes.
- **Workspace isolation** since **2026-02-05**: caches are keyed per workspace, not per organization.

[source: platform.claude.com/docs/en/build-with-claude/prompt-caching, retrieved 2026-04-18]

Response `usage` fields:

```json
"usage": {
  "input_tokens": 50,
  "cache_creation_input_tokens": 0,
  "cache_read_input_tokens": 100000,
  "output_tokens": 503,
  "cache_creation": {
    "ephemeral_5m_input_tokens": 456,
    "ephemeral_1h_input_tokens": 100
  }
}
```

Verify caching occurred by checking `cache_creation_input_tokens` and `cache_read_input_tokens`; both 0 means the prompt fell below the minimum and cached silently as no-op.
[source: platform.claude.com/docs/en/build-with-claude/prompt-caching, retrieved 2026-04-18]

### Streaming (SSE)

Stream events:

- `message_start` → `content_block_start` → `content_block_delta` (multiple) → `content_block_stop` → ... → `message_delta` → `message_stop`.
- Delta types: `text_delta`, `thinking_delta`, `signature_delta`, `input_json_delta` (tool call input streaming).
- With `thinking.display: "omitted"`, **no `thinking_delta` events** are emitted; only `signature_delta` arrives in the thinking block, and text streaming begins immediately after.

[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]

### Batch API

Extended output up to 300K tokens available on Opus 4.7, Opus 4.6, and Sonnet 4.6 with the `output-300k-2026-03-24` beta header. Caching multipliers **stack** with Batch discounts.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/prompt-caching, retrieved 2026-04-18]

## 8. Deployment Flags

For Claude this section covers **beta headers** and **platform routing**, rather than self-hosted inference flags (no open-weights release).

### Beta headers

| Header                              | Purpose                                                    | Applies to                     |
|-------------------------------------|------------------------------------------------------------|--------------------------------|
| `task-budgets-2026-03-13`           | Enable `output_config.task_budget`                         | Opus 4.7                        |
| `interleaved-thinking-2025-05-14`   | Enable interleaved thinking in manual thinking mode        | Sonnet 4.6 manual mode         |
| `output-300k-2026-03-24`            | Extended output up to 300K tokens via Batch API            | Opus 4.7, Opus 4.6, Sonnet 4.6 |
| `structured-outputs-2025-11-13`     | Legacy; Structured Outputs no longer requires a beta header| Deprecated, still accepted     |

[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-04-18]

### Platform routing

Bedrock (Sonnet 4.5+): choose **global** endpoints (dynamic routing across regions) or **regional** endpoints (guaranteed geographic routing).

Vertex AI: choose **global**, **multi-region** (within a geographic area), or **regional** endpoints.

`signature` values on thinking blocks are compatible across Claude API, Bedrock, and Vertex AI — a value generated on one platform validates on another.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]

## 9. Deprecations and Breaking Changes

### Opus 4.7 breaking changes from Opus 4.6

[applies-to: claude-opus-4-7]

- **Sampling parameters rejected.** `temperature`, `top_p`, `top_k` non-default values → 400. Omit them.
- **Manual thinking rejected.** `thinking.type: "enabled"` → 400. Use `{type: "adaptive"}` with `output_config.effort`.
- **`thinking.display` default changed** from `"summarized"` (Opus 4.6) to `"omitted"`. Thinking `thinking` field returns empty unless you set `display: "summarized"` explicitly.
- **New tokenizer.** Token counts for the same text are **1.0× to 1.35× higher** than Opus 4.6. Update `max_tokens` and compaction thresholds with headroom.
- **Behavioral shifts** (more literal instruction following, shorter default verbosity, fewer default tool calls) are not API errors but require prompt tuning — see `claude-prompt.md` §6.

[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]

### Claude 4.6-generation deprecations

- `thinking.type: "enabled"` with `budget_tokens` is **deprecated** on Opus 4.6 and Sonnet 4.6; still functional. Migrate to `{type: "adaptive"}` + `effort`.
- **Prefilled assistant messages** on the last turn are deprecated across 4.6+ models (and rejected by Mythos Preview).

[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Model retirements

- **Claude Sonnet 4** (`claude-sonnet-4-20250514`) retires **2026-06-15**. Migrate to `claude-sonnet-4-6`.
- **Claude Opus 4** (`claude-opus-4-20250514`) retires **2026-06-15**. Migrate to `claude-opus-4-7`.
- **Claude Haiku 3** (`claude-3-haiku-20240307`) retires **2026-04-19**. Migrate to `claude-haiku-4-5`.

[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

## 10. Gaps

- **Files API, PDF support, Citations, MCP connector, Claude Managed Agents, Claude Code–specific API behaviors** are all real product surfaces but out of scope here. See Anthropic's per-feature docs.
- **`redacted_thinking` content-block type** is referenced in some primary sources for safety-redacted reasoning; the retrieval pass did not pin down its exact semantics or trigger conditions. Treat as partial until re-verified.
- **Anthropic SDK per-language parameter name variance** (e.g. `outputConfig` in TypeScript, `OutputConfig` in C#/Java) is predictable from the JSON field names but not exhaustively documented here.
- **Exact current version of each server-tool type** (e.g. `web_search_20260209`) drifts. Fetch the tool reference page at integration time rather than relying on a stamped value.
- **Quantitative `effort`-level token-spend multipliers** are not published per-model per-workload; Anthropic's guidance is qualitative. Measure on your own evals.
