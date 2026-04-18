---
family: gemini
scope: api
versions:
  - gemini-3.1-pro-preview
  - gemini-3-flash-preview
  - gemini-3.1-flash-lite-preview
  - gemini-3.1-flash-live-preview
  - gemini-2.5-pro
  - gemini-2.5-flash
  - gemini-2.5-flash-lite
retrieved: 2026-04-18
primary_sources:
  - https://ai.google.dev/gemini-api/docs/models
  - https://ai.google.dev/gemini-api/docs/thinking
  - https://ai.google.dev/gemini-api/docs/function-calling
  - https://ai.google.dev/gemini-api/docs/caching
  - https://ai.google.dev/gemini-api/docs/structured-output
  - https://blog.google/products/gemini/gemini-3-flash/
maturity_note: |
  The Gemini 3 family is mostly Preview as of this retrieval. Field paths
  differ between REST (`generationConfig`) and SDKs (`config`); this file uses
  REST shapes and flags SDK aliases inline. Cache discount percentages,
  per-model context windows, and several rate-limit specifics are partial in
  the retrieved sources and appear in Gaps.
---

# Gemini — API-Layer Reference

API-call-level detail for the current Gemini 3 generation (and 2.5 where still relevant). Portable prompt-layer content lives in `gemini-prompt.md`.

## 1. API Surface

### Endpoints

- **Gemini API** (ai.google.dev) — public developer API, consumer-facing billing.
- **Vertex AI** (cloud.google.com/vertex-ai) — enterprise control plane, regional routing, IAM, billing via Google Cloud.
- **OpenAI compatibility layer** — Gemini API exposes an OpenAI-compatible endpoint for drop-in migrations (see `ai.google.dev/gemini-api/docs/openai`).

[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

### SDKs

First-party Google GenAI SDKs: Python, JavaScript/TypeScript, Go, Java. SDK field names differ from REST:

- REST: `generationConfig`, `systemInstruction`, `safetySettings`.
- Python / JS SDK: `config` (which nests the REST equivalents).

Translate between the two when copying examples.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

### Model IDs

Use dated model IDs when pinning. The base preview IDs (e.g. `gemini-3-flash-preview`) rotate to newer snapshots without notice on the same string. Google has shut down superseded preview IDs (see §9).

## 2. Chat Template / Message Structure

Gemini does not use a special-token chat template (no `<|im_start|>` equivalent). The protocol is a parts-based JSON structure.

### Basic shape

```json
{
  "model": "models/gemini-3-flash-preview",
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

| Model                             | Default `thinkingLevel` | Notes                                                                                           |
|-----------------------------------|--------------------------|-------------------------------------------------------------------------------------------------|
| `gemini-3.1-pro-preview`          | `high`                   | Thinking on by default; `minimal` approximates non-thinking behavior                            |
| `gemini-3-flash-preview`          | `high`                   | Same default as Pro                                                                             |
| `gemini-3.1-flash-lite-preview`   | `minimal`                | Does not think by default; raise level for reasoning tasks                                      |

[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]
[testable: id=gemini.flash-lite-thinking-default-minimal.v1, expected=request to gemini-3.1-flash-lite-preview with no thinkingConfig produces response with thoughtsTokenCount near zero]

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

Per-model bounds:

| Model                   | Range            | Default              |
|-------------------------|------------------|----------------------|
| `gemini-2.5-pro`        | 128 – 32,768     | `-1` (dynamic)       |
| `gemini-2.5-flash`      | 0 – 24,576       | `-1` (dynamic)       |
| `gemini-2.5-flash-lite` | 0, or 512 – 24,576 | `0` (no thinking)  |

[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

[applies-to: gemini-3.1-pro-preview, gemini-3-flash-preview, gemini-3.1-flash-lite-preview]
`thinkingBudget` is accepted on Gemini 3 for backwards compatibility, but `thinkingLevel` is the contract. Mixing `thinkingBudget` on 3.1 Pro may cause unexpected performance.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]
[testable: id=gemini.thinking-level-replaces-budget.v1, expected=request to gemini-3.1-pro-preview with thinkingBudget returns result but documentation recommends thinkingLevel]

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

### `thoughtSignature` handling

When thinking is enabled and function calling is in play, responses carry opaque `thoughtSignature` strings on parts. Gemini 3 returns signatures more broadly than 2.5. Multi-turn handling rules:

- **Return the response parts intact** — do not concatenate, re-order, or strip signatures.
- **Required** for preserving thought context across turns in function-calling conversations.
- **Cross-platform compatible** between Gemini API and Vertex AI.

[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

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

[applies-to: gemini-3.1-pro-preview, gemini-3-flash-preview, gemini-3.1-flash-lite-preview]
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
| `gemini-3.1-pro-preview`          | ✓                 | Combines with function calling                     |
| `gemini-3-flash-preview`          | ✓                 | Combines with function calling                     |
| `gemini-2.5-pro`                  | ✓                 |                                                    |
| `gemini-2.5-flash`                | ✓                 |                                                    |
| `gemini-2.5-flash-lite`           | ✓                 |                                                    |
| `gemini-2.0-flash`                | ✓                 | Requires explicit `propertyOrdering` array in schema |
| `gemini-2.0-flash-lite`           | ✓                 | Same `propertyOrdering` requirement                |

[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]

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
| `gemini-3-flash-preview`  | 1024                     |
| `gemini-3.1-pro-preview`  | 4096                     |
| `gemini-2.5-flash`        | 1024                     |
| `gemini-2.5-pro`          | 4096                     |

**Explicit caching** — manually created caches with predictable savings.

```python
cache = client.caches.create(
    model="gemini-3-flash-preview",
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

### Streaming

Server-Sent Events (SSE) streaming on `generateContentStream` (SDK) / `:streamGenerateContent` (REST endpoint suffix). Streams progressively arriving parts, including incremental thought summaries when `includeThoughts: true`. Function calls arrive as complete parts (not character-by-character).
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

### Batch

Batch API exists on the Gemini API and Vertex AI surfaces; detailed parameter shape is not covered in the retrieved excerpts. Consult `ai.google.dev/gemini-api/docs/batch-mode` before use.

## 8. Deployment Flags (closed-platform routing)

Closed-platform routing for Gemini is platform-level (there is no self-hosted path). Key distinctions:

- **Gemini API (ai.google.dev)** — global endpoint, consumer-facing pricing, free tier available.
- **Vertex AI** — region-bound; regional endpoints for data-residency, IAM integration, enterprise billing.
- **Model ID prefix differences** — Gemini API uses bare IDs (`gemini-3-flash-preview`); Vertex AI uses full resource paths (e.g. `projects/.../locations/.../publishers/google/models/gemini-3-flash`).

[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

There are no Gemini-specific beta headers; feature flags are surfaced via per-field config (e.g. `includeServerSideToolInvocations`) or via model ID (preview variants).

## 9. Deprecations and Breaking Changes

### Gemini 3 vs Gemini 2.5 thinking control

[applies-to: gemini-3.1-pro-preview, gemini-3-flash-preview, gemini-3.1-flash-lite-preview]
`thinkingLevel` replaces `thinkingBudget`. Callers migrating from 2.5 must rework thinking controls — `thinkingBudget` is still accepted but may not produce the intended control on 3.1 Pro.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

### Shutdown of the original Gemini 3 Pro Preview

**`gemini-3-pro-preview`** (the *original* 3 Pro, not `gemini-3.1-pro-preview`) was shut down **2026-03-09**. Callers must migrate to `gemini-3.1-pro-preview`.
[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

### Gemini 2.0 deprecations

- `gemini-2.0-flash` — shutdown pending.
- `gemini-2.0-flash-lite` — deprecated.

[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

### Function-call `id` field (Gemini 3 addition)

Gemini 3 responses carry a unique `id` on every `functionCall`. Code from the Gemini 2.5 era that ignored the `id` field will still work for single-call cases but must respect `id` for parallel-call result matching on Gemini 3.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

## 10. Gaps

- **Exact per-model context window sizes** for the Gemini 3 Preview family are not quoted in the retrieved models-page excerpt. Confirm at integration time.
- **Explicit-cache discount percentage** (often cited as 90% on Gemini 2.5+) is not replicated verbatim in the current caching-page primary excerpt; treat the percentage as community-reported until re-verified.
- **Vertex AI batch API parameter shape** is not covered here.
- **Live API (`gemini-3.1-flash-live-preview`)** bi-directional audio protocol — session setup, VAD, turn-detection signals, interruption handling — is out of scope.
- **Deep Research (`deep-research-pro-preview-12-2025`) parameter surface** and differences from standard Messages-style invocation are undocumented in the retrieved excerpts.
- **Safety settings** (`safetySettings` array, category / threshold enums) were not targeted in this retrieval pass.
- **OpenAI-compatibility layer behavioral deviations** from the Messages API (thinking, tool IDs, signatures) are not quoted.
- **Rate-limit and quota specifics** per model / per tier are not covered.
