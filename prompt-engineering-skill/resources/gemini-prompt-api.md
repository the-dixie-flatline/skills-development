---
family: gemini
scope: api
versions:
  - gemini-3.5-flash
  - gemini-3.1-flash-lite
  - gemini-3.1-pro-preview
  - gemini-2.5-pro
  - gemini-2.5-flash
  - gemini-2.5-flash-lite
retrieved: 2026-06-01
primary_sources:
  - https://ai.google.dev/gemini-api/docs/models
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.5-flash
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.1-flash-lite
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview
  - https://ai.google.dev/gemini-api/docs/thinking
  - https://ai.google.dev/gemini-api/docs/function-calling
  - https://ai.google.dev/gemini-api/docs/caching
  - https://ai.google.dev/gemini-api/docs/structured-output
  - https://ai.google.dev/gemini-api/docs/openai
  - https://ai.google.dev/gemini-api/docs/thought-signatures
  - https://ai.google.dev/gemini-api/docs/deprecations
maturity_note: |
  Gemini 3.5 Flash reached GA on 2026-05-19 and Gemini 3.1 Flash-Lite reached
  GA on 2026-05-07; Gemini 3.1 Pro remains Preview. Field paths differ between
  REST (`generationConfig`) and SDKs (`config`); this file uses REST shapes and
  flags SDK aliases inline. The Gemini 3 line uses `thinkingLevel`, replacing
  the 2.5-era `thinkingBudget`. Deprecation pressure is high: Gemini 2.0 Flash /
  Flash-Lite GA shut down 2026-06-01; Gemini 2.5 Pro / Flash / Flash-Lite GA
  sunset 2026-10-16. Cache discount percentages, per-model context windows, and
  several rate-limit specifics are partial in the retrieved sources and appear
  in Gaps.
---

# Gemini — API-Layer Reference

API-call-level detail for the current Gemini 3 generation (and 2.5 where still relevant). Portable prompt-layer content lives in `gemini-prompt.md`.

## 1. API Surface

### Endpoints

- **Gemini API** (ai.google.dev) — public developer API, consumer-facing billing.
- **Vertex AI** (cloud.google.com/vertex-ai) — enterprise control plane, regional routing, IAM, billing via Google Cloud.
- **OpenAI compatibility layer** — Gemini API exposes an OpenAI-compatible endpoint at base URL `https://generativelanguage.googleapis.com/v1beta/openai/` for drop-in migrations. Deviations are detailed in §4 and §5.

[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/openai, retrieved 2026-06-01]

The full cross-family OpenAI-compat hazard matrix lives in `resources/openai-compatibility-surface.md`; it is not duplicated here.

### SDKs

First-party Google GenAI SDKs: Python, JavaScript/TypeScript, Go, Java. SDK field names differ from REST:

- REST: `generationConfig`, `systemInstruction`, `safetySettings`.
- Python / JS SDK: `config` (which nests the REST equivalents).

Translate between the two when copying examples.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

### Model IDs

Use dated model IDs when pinning. Base preview IDs (e.g. `gemini-3.1-pro-preview`) rotate to newer snapshots without notice on the same string. Google has shut down superseded preview IDs (see §9). `gemini-3.1-pro-preview-customtools` is a separate endpoint tuned to prioritize custom tools over built-in tools.
[source: ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview, retrieved 2026-06-01]

## 2. Chat Template / Message Structure

Gemini does not use a special-token chat template (no `<|im_start|>` equivalent). The protocol is a parts-based JSON structure.

### Basic shape

```json
{
  "model": "models/gemini-3.5-flash",
  "systemInstruction": {
    "parts": [{ "text": "You are a helpful coding assistant." }]
  },
  "contents": [
    { "role": "user", "parts": [{ "text": "..." }] },
    { "role": "model", "parts": [{ "text": "..." }] }
  ],
  "generationConfig": { "... see below ..." }
}
```

- `systemInstruction` is a **top-level** field (not a `contents` entry).
- Valid roles in `contents`: `user`, `model`.
- `parts` is always an array; each part is one of: `{text}`, `{inlineData: {mimeType, data}}`, `{fileData: {mimeType, fileUri}}`, `{functionCall: {id, name, args}}`, `{functionResponse: {id, name, response}}`, plus the thinking-related `thought: true` + `thoughtSignature`.

[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

### Multimodal parts

- `inlineData` — base64-encoded payload with MIME type. Good for small images/audio.
- `fileData` — URI reference to content uploaded via the Files API. Required for large files.

[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

## 3. Sampling Parameters

Inside `generationConfig` (REST) / `config` (SDK):

```json
{
  "generationConfig": {
    "temperature": 1.0,
    "topP": 0.95,
    "topK": 40,
    "maxOutputTokens": 8192,
    "candidateCount": 1,
    "stopSequences": ["..."],
    "responseMimeType": "application/json",
    "responseSchema": { "... " }
  }
}
```

Defaults and recommended values are not uniformly documented across Gemini 3 Preview models in the retrieved primary sources. Empirically validate for each model; do not assume a single default.

[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

## 4. Reasoning / Thinking Control

### Field path

REST: `generationConfig.thinkingConfig`. SDK: `config.thinkingConfig`.

### `thinkingLevel` (Gemini 3)

```json
"generationConfig": {
  "thinkingConfig": {
    "thinkingLevel": "minimal" | "low" | "medium" | "high",
    "includeThoughts": true
  }
}
```

Valid values and defaults:

`thinkingLevel` is the recommended reasoning control for Gemini 3 models and onward; it replaces `thinkingBudget` (retained for the 2.5 series only).

| Model                       | Default `thinkingLevel` | Notes                                                                       |
|-----------------------------|--------------------------|-----------------------------------------------------------------------------|
| `gemini-3.1-pro-preview`    | `high`                   | Cannot set `minimal`; cannot disable thinking                               |
| `gemini-3.5-flash`          | `medium`                 |                                                                             |
| `gemini-3.1-flash-lite`     | `minimal`                | Does not think by default; raise level for reasoning tasks                  |

[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-06-01]
[testable: id=gemini.flash-lite-thinking-default-minimal.v1, expected=request to gemini-3.1-flash-lite with no thinkingConfig produces response with thoughtsTokenCount near zero]

### `thinkingBudget` (Gemini 2.5)

```json
"generationConfig": {
  "thinkingConfig": {
    "thinkingBudget": -1 | 0 | <int>,
    "includeThoughts": true
  }
}
```

- `-1` — dynamic thinking (model decides budget from complexity).
- `0` — thinking disabled (supported on 2.5 Flash / Flash-Lite; **not** supported on 2.5 Pro).

The Gemini 2.5 series does not support `thinkingLevel` — "2.5 series models don't support thinkingLevel; use thinkingBudget instead."
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-06-01]

Per-model bounds:

| Model                   | Range            | Default              |
|-------------------------|------------------|----------------------|
| `gemini-2.5-pro`        | 128 – 32,768     | `-1` (dynamic)       |
| `gemini-2.5-flash`      | 0 – 24,576       | `-1` (dynamic)       |
| `gemini-2.5-flash-lite` | 0, or 512 – 24,576 | `0` (no thinking)  |

[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

[applies-to: gemini-3.5-flash, gemini-3.1-flash-lite, gemini-3.1-pro-preview]
`thinkingLevel` is the control for Gemini 3 and onward; `thinkingBudget` is retained for the 2.5 series only. Do not carry 2.5 `thinkingBudget` code forward to 3.x — use `thinkingLevel`.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-06-01]
[testable: id=gemini.thinking-level-replaces-budget.v1, expected=request to gemini-3.1-pro-preview using thinkingLevel high produces non-zero thoughtsTokenCount]

### Thought summaries

Set `includeThoughts: true` to receive summarized thinking as parts with `thought: true`:

```json
"candidates": [{
  "content": {
    "parts": [
      { "text": "Thought summary text...", "thought": true },
      { "text": "Final answer text..." }
    ]
  }
}],
"usageMetadata": { "thoughtsTokenCount": 500, "candidatesTokenCount": 200 }
```

Billing: thinking tokens count toward output tokens via `thoughtsTokenCount` (separate usage field). Summaries themselves are not separately billed.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

### Thinking via the OpenAI-compatibility layer

On the OpenAI-compatible endpoint, reasoning is controlled by `reasoning_effort` {minimal, low, medium, high}, which maps to `thinking_level` / `thinking_budget` under the hood. The value `none` is accepted for 2.5 models only; reasoning cannot be turned off for Gemini 2.5 Pro or any Gemini 3 model. There is **no documented `reasoning_content` response field** on this layer. To receive thought summaries, set `extra_body.google.thinking_config.include_thoughts=true` (the retrieval field/shape for the summary is not specified in the docs). The full cross-family OpenAI-compat hazard matrix lives in `resources/openai-compatibility-surface.md`.
[source: ai.google.dev/gemini-api/docs/openai, retrieved 2026-06-01]

### `thoughtSignature` handling

When thinking is enabled and function calling is in play, responses carry opaque `thoughtSignature` strings on parts. Gemini 3 returns signatures more broadly than 2.5. Multi-turn handling rules:

- **Return the response parts intact** — do not concatenate, re-order, or strip signatures.
- **Required** for preserving thought context across turns in function-calling conversations.
- **Cross-platform compatible** between Gemini API and Vertex AI.

[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

On the OpenAI-compatibility layer, tool-call thought signatures travel in `extra_content.google.thought_signature` and **must be echoed** back for Gemini 3 function calls.
[source: ai.google.dev/gemini-api/docs/openai, retrieved 2026-06-01]
[source: ai.google.dev/gemini-api/docs/thought-signatures, retrieved 2026-06-01]

## 5. Tool Use / Function Calling

### Request shape

```json
{
  "tools": [{
    "functionDeclarations": [
      {
        "name": "get_weather",
        "description": "Fetch current weather for a location.",
        "parameters": {
          "type": "object",
          "properties": {
            "location": { "type": "string" }
          },
          "required": ["location"]
        }
      }
    ]
  }],
  "toolConfig": {
    "functionCallingConfig": {
      "mode": "AUTO" | "ANY" | "NONE" | "VALIDATED",
      "allowedFunctionNames": ["get_weather"]
    }
  }
}
```

Mode semantics:

- `AUTO` — default with function declarations alone; model decides between natural-language and function calls.
- `ANY` — model must always call a function.
- `NONE` — function calling disabled.
- `VALIDATED` — default when tools are combined with structured outputs; schema-conformance enforced on every function call.

[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

### Response shape

Function calls arrive as parts with a `functionCall` field:

```json
{
  "functionCall": {
    "id": "8f2b1a3c",
    "name": "get_weather",
    "args": { "location": "Paris, FR" }
  }
}
```

[applies-to: gemini-3.5-flash, gemini-3.1-flash-lite, gemini-3.1-pro-preview]
Gemini 3 generates a unique `id` on **every** function call. Use it to match tool results — parallel calls are order-agnostic on return.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

### Tool-result roundtrip

```json
{
  "role": "user",
  "parts": [{
    "functionResponse": {
      "name": "get_weather",
      "id": "8f2b1a3c",
      "response": { "result": "20°C, sunny" }
    }
  }]
}
```

Multimodal tool results are supported via `inlineData` inside `functionResponse.parts`.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

### Parallel and compositional calls

- **Parallel**: multiple `functionCall` parts in one assistant turn. Results can be returned in any order — the `id` is the match key.
- **Compositional**: sequential calls where each result informs the next. Supported by the model natively; no special flag required.

[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

### Built-in server-side tools

Combinable with custom `functionDeclarations` on Gemini 3:

```json
"tools": [
  { "googleSearch": {} },
  { "urlContext": {} },
  { "codeExecution": {} },
  { "functionDeclarations": [ /* ... */ ] }
]
```

Set `"includeServerSideToolInvocations": true` in config to receive traces of the server-side invocations.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

## 6. Structured Outputs

### Request shape

```json
"generationConfig": {
  "responseMimeType": "application/json",
  "responseSchema": {
    "type": "object",
    "properties": {
      "summary": { "type": "string" },
      "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
    },
    "required": ["summary", "confidence"]
  }
}
```

The Python and JavaScript SDKs also accept `responseJsonSchema` as an alternative; REST uses `responseSchema`.

### Supported JSON Schema subset

Supported: `string`, `number`, `integer`, `boolean`, `object`, `array`, `null`; `title`, `description`, `enum`, `format` (`date-time`, `date`, `time`) on strings; `minimum`, `maximum` on numbers/integers; `properties`, `required`, `additionalProperties` on objects; `items`, `prefixItems`, `minItems`, `maxItems` on arrays.

**Unsupported constructs are silently ignored** (not rejected). Examples: `minLength`, `maxLength`, complex `pattern` regex, `multipleOf`, recursive schemas, external `$ref` URLs.
[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]
[testable: id=gemini.unsupported-schema-silent.v1, expected=schema with minLength constraint returns response that may violate the constraint; no 400 error]

### Model support

| Model                             | Structured output | Notes                                              |
|-----------------------------------|-------------------|----------------------------------------------------|
| `gemini-3.5-flash`                | ✓                 | Combines with function calling                     |
| `gemini-3.1-flash-lite`           | ✓                 | Combines with function calling                     |
| `gemini-3.1-pro-preview`          | ✓                 | Combines with function calling                     |
| `gemini-2.5-pro`                  | ✓                 | GA sunset 2026-10-16                               |
| `gemini-2.5-flash`                | ✓                 | GA sunset 2026-10-16                               |
| `gemini-2.5-flash-lite`           | ✓                 | GA sunset 2026-10-16                               |

[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-06-01]

### Function calling + structured output

Gemini 3 supports combining them. The model may call a function or emit schema-conforming JSON; both are valid outputs. Useful when the fallback from "no tool required" should still be structured.
[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

## 7. Caching, Batch, Streaming

### Context caching

Two tiers:

**Implicit caching** — automatic, no cost guarantee. Enabled by default on supported models.

| Model                     | Minimum cacheable tokens |
|---------------------------|--------------------------|
| `gemini-3.5-flash`        | 1024                     |
| `gemini-3.1-pro-preview`  | 4096                     |
| `gemini-2.5-flash`        | 1024                     |
| `gemini-2.5-pro`          | 4096                     |

**Explicit caching** — manually created caches with predictable savings.

```python
cache = client.caches.create(
    model="gemini-3.5-flash",
    contents=[...],
    system_instruction="...",
    ttl="3600s",
    display_name="my-cache",
)
```

- `ttl` — ISO 8601 duration string. Default **1 hour** when not specified. No minimum or maximum documented.
- Reference on subsequent calls via the `cachedContent` field (value = `cache.name` returned at creation).
- Operations: `client.caches.list()`, `client.caches.update(name, config)` (`ttl` / `expire_time` only), `client.caches.delete(cache.name)`.

[source: ai.google.dev/gemini-api/docs/caching, retrieved 2026-04-18]

```json
{
  "contents": [{ "parts": [{ "text": "new user query" }], "role": "user" }],
  "cachedContent": "cachedContents/abcdef12345"
}
```

[source: ai.google.dev/gemini-api/docs/caching, retrieved 2026-04-18]

On the OpenAI-compatibility layer, reference cached content via `extra_body.cached_content`.
[source: ai.google.dev/gemini-api/docs/openai, retrieved 2026-06-01]

### Streaming

Server-Sent Events (SSE) streaming on `generateContentStream` (SDK) / `:streamGenerateContent` (REST endpoint suffix). Streams progressively arriving parts, including incremental thought summaries when `includeThoughts: true`. Function calls arrive as complete parts (not character-by-character).
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

### Batch

Batch API exists on the Gemini API and Vertex AI surfaces; detailed parameter shape is not covered in the retrieved excerpts. Consult `ai.google.dev/gemini-api/docs/batch-mode` before use.

## 8. Deployment Flags (closed-platform routing)

Closed-platform routing for Gemini is platform-level (there is no self-hosted path). Key distinctions:

- **Gemini API (ai.google.dev)** — global endpoint, consumer-facing pricing, free tier available.
- **Vertex AI** — region-bound; regional endpoints for data-residency, IAM integration, enterprise billing.
- **Model ID prefix differences** — Gemini API uses bare IDs (`gemini-3.5-flash`); Vertex AI uses full resource paths (e.g. `projects/.../locations/.../publishers/google/models/gemini-3.5-flash`).

[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

There are no Gemini-specific beta headers; feature flags are surfaced via per-field config (e.g. `includeServerSideToolInvocations`) or via model ID (preview variants).

## 9. Deprecations and Breaking Changes

### Gemini 3 vs Gemini 2.5 thinking control

[applies-to: gemini-3.5-flash, gemini-3.1-flash-lite, gemini-3.1-pro-preview]
`thinkingLevel` replaces `thinkingBudget`. `thinkingBudget` is retained for the 2.5 series only; callers migrating from 2.5 must rework thinking controls onto `thinkingLevel`.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-06-01]

### Gemini 2.0 GA shutdown (2026-06-01)

`gemini-2.0-flash` / `gemini-2.0-flash-lite` GA, including the `-001` variants, shut down **2026-06-01**.
[source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-06-01]

### Gemini 2.5 GA sunset (2026-10-16)

`gemini-2.5-pro` / `gemini-2.5-flash` / `gemini-2.5-flash-lite` GA sunset **2026-10-16**.
[source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-06-01]

### Shut-down preview IDs

- **`gemini-3-pro-preview`** (the *original* 3 Pro, not `gemini-3.1-pro-preview`) shut down **2026-03-09**. Migrate to `gemini-3.1-pro-preview`.
- **`gemini-3.1-flash-lite-preview`** shut down **2026-05-25**, superseded by GA `gemini-3.1-flash-lite`.
- **`gemini-3-flash-preview`** was the preview that GA `gemini-3.5-flash` replaced.

[source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-06-01]

### Function-call `id` field (Gemini 3 addition)

Gemini 3 responses carry a unique `id` on every `functionCall`. Code from the Gemini 2.5 era that ignored the `id` field will still work for single-call cases but must respect `id` for parallel-call result matching on Gemini 3.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

## 10. Gaps

- **Exact per-model context window sizes** for the Gemini 3 Preview family are not quoted in the retrieved models-page excerpt. Confirm at integration time.
- **Explicit-cache discount percentage** (often cited as 90% on Gemini 2.5+) is not replicated verbatim in the current caching-page primary excerpt; treat the percentage as community-reported until re-verified.
- **Vertex AI batch API parameter shape** is not covered here.
- **Live API** bi-directional audio protocol — session setup, VAD, turn-detection signals, interruption handling — is out of scope.
- **Gemini Deep Research, the Antigravity managed agent, and Computer Use** are documented in `resources/deep-research-agents.md` and `resources/agent-orchestration-surfaces.md`.
- **Safety settings** (`safetySettings` array, category / threshold enums) were not targeted in this retrieval pass.
- **OpenAI-compat thought-summary retrieval shape** — `extra_body.google.thinking_config.include_thoughts=true` enables summaries, but the response field/shape carrying them is not specified in the docs.
- **Rate-limit and quota specifics** per model / per tier are not covered.
