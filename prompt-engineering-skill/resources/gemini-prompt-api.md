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
retrieved: 2026-07-19
primary_sources:
  - https://ai.google.dev/gemini-api/docs/models
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.5-flash
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.1-flash-lite
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview
  - https://ai.google.dev/gemini-api/docs/interactions-overview
  - https://ai.google.dev/gemini-api/docs/migrate-to-interactions
  - https://ai.google.dev/api/interactions-api
  - https://ai.google.dev/api/agents
  - https://ai.google.dev/gemini-api/docs/streaming
  - https://ai.google.dev/gemini-api/docs/thinking
  - https://ai.google.dev/gemini-api/docs/function-calling
  - https://ai.google.dev/gemini-api/docs/caching
  - https://ai.google.dev/gemini-api/docs/structured-output
  - https://ai.google.dev/gemini-api/docs/openai
  - https://ai.google.dev/gemini-api/docs/thought-signatures
  - https://ai.google.dev/gemini-api/docs/deprecations
  - https://blog.google/innovation-and-ai/technology/developers-tools/interactions-api-general-availability/
maturity_note: |
  The Interactions API reached GA on 2026-06-22 and is now the vendor-recommended
  primary surface for all new projects; `generateContent` is relabeled legacy but
  remains fully supported and continues to receive new mainline Gemini models.
  This file leads each section with the Interactions (snake_case) shape and retains
  the legacy `generateContent` (camelCase) shape clearly labeled. Gemini 3.5 Flash
  reached GA on 2026-05-19 and Gemini 3.1 Flash-Lite reached GA on 2026-05-07;
  Gemini 3.1 Pro remains Preview. The Gemini 3 line uses `thinkingLevel`, replacing
  the 2.5-era `thinkingBudget`. Deprecation pressure is high: Gemini 2.0 Flash /
  Flash-Lite GA shut down 2026-06-01; Gemini 2.5 Pro / Flash / Flash-Lite GA
  sunset 2026-10-16. Cache discount percentages and several rate-limit specifics
  are partial in the retrieved sources and appear in Gaps. Interactions API facts
  carry a 2026-07-19 retrieval date; older legacy claims retain their 2026-04-18 /
  2026-06-01 / 2026-07-18 inline dates.
---

# Gemini — API-Layer Reference

API-call-level detail for the current Gemini 3 generation (and 2.5 where still relevant). Portable prompt-layer content lives in `gemini-prompt.md`.

## 1. API Surface

### Surface selection: Interactions (primary) vs `generateContent` (legacy)

Gemini exposes two request surfaces on the Gemini API. Choose the surface before writing integration code.

- **Interactions API — primary, GA.** Vendor-recommended for all new projects: "The Interactions API is now generally available. We recommend using this API for all new projects." GA'd 2026-06-22, public beta since December 2025.
[source: ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]
[source: blog.google/innovation-and-ai/technology/developers-tools/interactions-api-general-availability/, published 2026-06-22, retrieved 2026-07-19]
- **`generateContent` — legacy, fully supported.** "now considered legacy" yet "fully supported"; the migration page states "While generateContent remains fully supported, we recommend the Interactions API for all new development." The legacy surface "will continue to receive new mainline Gemini models for the foreseeable future," but Google expects "frontier capabilities for long-running models and agents to increasingly land exclusively on the Interactions API."
[source: ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]
[source: ai.google.dev/gemini-api/docs/migrate-to-interactions, retrieved 2026-07-19]
[source: blog.google/innovation-and-ai/technology/developers-tools/interactions-api-general-availability/, retrieved 2026-07-19]

Prompt craft (roles, system instruction, structured-output intent, thinking level) is portable across both surfaces; the request envelope and field paths differ. The Interactions surface is snake_case (`generation_config`, `thinking_level`, `tool_choice`, `response_format`); the legacy surface is camelCase (`generationConfig`, `thinkingConfig`, `responseSchema`). Each section below leads with the Interactions shape and retains the legacy `generateContent` shape clearly labeled.

### Endpoints

- **Gemini API** (ai.google.dev) — public developer API, consumer-facing billing.
- **Interactions API** — primary Gemini API surface: `POST https://generativelanguage.googleapis.com/v1beta/interactions` (SDK `client.interactions.create`). A stable `v1` version exists alongside `v1beta`; this file documents `v1beta`.
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]
- **`generateContent`** — legacy Gemini API surface: `POST .../v1beta/models/{model}:generateContent`. Fully supported (see surface selection above).
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

The Interactions API (GA) requires SDK floors `google-genai >= 2.3.0` and `@google/genai >= 2.3.0`.
[source: ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]

### Model IDs

Use dated model IDs when pinning. Base preview IDs (e.g. `gemini-3.1-pro-preview`) rotate to newer snapshots without notice on the same string. Google has shut down superseded preview IDs (see §9). `gemini-3.1-pro-preview-customtools` is a separate endpoint tuned to prioritize custom tools over built-in tools.
[source: ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview, retrieved 2026-06-01]

## 2. Chat Template / Message Structure

Gemini does not use a special-token chat template (no `<|im_start|>` equivalent). Both surfaces are JSON. The Interactions API uses an `input` list of typed `Step` objects; the legacy `generateContent` surface uses a `contents` list of parts.

### Interactions API envelope (primary surface)

The Interactions API replaces the `contents`/`parts` array with an `input` field carrying an ordered list of typed `Step` objects; the model reply comes back as a `steps` array.

```json
{
  "model": "gemini-3.5-flash",
  "input": [
    { "type": "user_input", "content": [{ "type": "text", "text": "..." }] }
  ],
  "generation_config": { "thinking_level": "medium" }
}
```

Documented step types: `user_input`, `model_output`, `function_call`, `function_result`, plus `thought` (which appears in thinking contexts).
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

Function-call pairing uses matched IDs, not positional order: a `function_call` step carries an `id` (example `"gth23981"`); the corresponding `function_result` step carries a matching `call_id`.
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

Terminal text is read from the last step: `interaction.steps[-1].content[0].text`.
[source: ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-07-19]

Server-side conversation state: pass `previous_interaction_id` to continue from a prior completed interaction (see §7).

### `generateContent` message shape (legacy surface)

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

- **Interactions API (primary):** `generation_config.thinking_level` (snake_case), a field on `generation_config`.
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]
- **Legacy `generateContent`:** `generationConfig.thinkingConfig` (REST) / `config.thinkingConfig` (SDK).

### `thinking_level` (Interactions API, primary surface)

```json
"generation_config": {
  "thinking_level": "minimal" | "low" | "medium" | "high"
}
```

Lowercase enum, the same four values as the legacy `thinkingLevel` on a different path. Per-model defaults match the legacy table below.
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

### `thinkingLevel` (Gemini 3, legacy `generateContent`)

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
Verified 2026-07-19: on gemini-3.1-flash-lite the default is effectively zero, not merely low. With no `thinkingConfig` the model emitted **zero** reasoning tokens (5/5); `reasoning:{effort:high}` emitted 295-463 on the same route, so the field is surfaced and the zero default is real, not a reporting gap (via Google AI Studio). "Minimal thinking by default" holds at the floor: dynamic thinking is off by default here.
[testable: id=gemini.flash-lite-thinking-default-minimal.v1, expected=request to gemini-3.1-flash-lite with no thinkingConfig produces zero reasoning tokens; effort:high produces non-zero (295-463 observed 2026-07-19)]

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

### Thought signatures on the Interactions API (primary surface)

The Interactions API simplifies signature handling relative to `generateContent`. In **stateful mode** (`store: true` plus `previous_interaction_id` on later turns) the server manages all thought blocks and signatures automatically — "They are handled entirely on the server side." In **stateless mode** (you resend full history each request) you MUST resend every `thought` block verbatim, including built-in-tool result signatures; do not remove or modify them. On this surface signatures live only on `thought` steps and built-in-tool steps (for example `google_search_call` / `google_search_result`), never on user inputs, model outputs, or standard function calls. A `thought` step's `signature` field is always present (even under minimal reasoning); its `summary` is present only when `thinking_summaries` is enabled and the model reasoned enough to produce one.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-07-19]

### `thoughtSignature` handling (legacy `generateContent`)

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

### Interactions API tool declaration (primary surface)

Tools are a flat array of typed objects. A function tool is `{type: "function", name, description, parameters}` — `name` / `description` / `parameters` sit directly on the tool object alongside `type`, not nested under a `function` key.

```json
{
  "tools": [
    {
      "type": "function",
      "name": "get_weather",
      "description": "Get the current weather in a given location",
      "parameters": {
        "type": "object",
        "properties": { "location": { "type": "string" } },
        "required": ["location"]
      }
    }
  ],
  "generation_config": { "tool_choice": "auto" }
}
```

[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]
[source: ai.google.dev/api/agents, retrieved 2026-07-19]

**`tool_choice` sits under `generation_config`, not at the top level.** Enum: `auto` | `any` | `none` | `validated`. A top-level `tool_choice` belongs to the separate Vertex enterprise-agent-platform product, not the Gemini API Interactions surface; do not place it at the top level here.
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

Built-in and MCP tool types share the polymorphic-on-`type` shape: `{type: "code_execution"}`; `{type: "google_search"}` (with a `search_types` array of `web_search` / `image_search` / `enterprise_web_search`); `{type: "url_context"}`; and `{type: "mcp_server"}` (`name`, `url`, `allowed_tools.{mode, tools}`, `headers`).
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

Function-call results pair by ID: the `function_call` step's `id` matches the `function_result` step's `call_id` (see §2).
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

### Request shape (legacy `generateContent`)

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

### `response_format` (Interactions API, primary surface)

The Interactions API uses `response_format` (a `ResponseFormat` object or an array of them), polymorphic on `type`, to enforce output shape — it "enforces that the generated response is a JSON object that complies with the JSON schema specified."
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

Four documented sub-shapes:

- **`TextResponseFormat`** (`type: "text"`): `mime_type` (`application/json` | `text/plain`), `schema` (JSON schema; only when `mime_type` is `application/json`).
- **`AudioResponseFormat`** (`type: "audio"`): `mime_type` (`audio/mp3` | `audio/ogg_opus` | `audio/l16` | `audio/wav` | `audio/alaw` | `audio/mulaw`), `bit_rate`, `sample_rate`, `delivery` (`inline` | `uri`).
- **`ImageResponseFormat`** (`type: "image"`): `mime_type` (`image/jpeg` documented), `aspect_ratio` (`1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9`, `1:8`, `8:1`, `1:4`, `4:1`), `image_size` (`512` | `1K` | `2K` | `4K`), `delivery` (`inline` | `uri`).
- **`VideoResponseFormat`** (`type: "video"`): `aspect_ratio` (`16:9` | `9:16`), `duration`, `delivery` (`inline` | `uri`), `gcs_uri` (documented as Vertex-only; required for Vertex when delivery is URI).

```json
{ "type": "text", "mime_type": "application/json", "schema": { "type": "object", "properties": { "recipe_name": { "type": "string" } }, "required": ["recipe_name"] } }
```

[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

A separate top-level `response_modalities` field (`text` / `image` / `audio` / `video` / `document`) selects output modalities and is distinct from `response_format`.
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]

### Request shape (legacy `generateContent`)

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

**Some constructs outside the documented subset are silently ignored** (not rejected). Examples: complex `pattern` regex, `multipleOf`, recursive schemas, external `$ref` URLs.
[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]

Do not generalize "silently ignored" to every out-of-subset keyword. Verified 2026-07-19: `minLength` is **enforced**, not silently ignored, on gemini-3-flash-preview. A `response_format` json_schema string field constrained `minLength:50` returned values >=50 chars on 3/3 (the model padded/expanded a bare `"ok"` to satisfy it), while a control schema without `minLength` returned bare `"ok"` (2 chars); a `require_parameters:true` re-probe was unchanged, so this is vendor-stack enforcement, not OpenRouter-side stripping (via Google AI Studio). Treat length constraints as potentially enforced on current Gemini 3 models; validate conformance yourself only for the genuinely-unsupported keywords listed above.
[testable: id=gemini.unsupported-schema-silent.v1, expected=on gemini-3-flash-preview a schema with minLength:50 returns strings >=50 chars (constraint enforced); a genuinely-unsupported keyword such as multipleOf is ignored with no 400 error]

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

### Interactions API server-side state and retention (primary surface)

The Interactions API keeps conversation state server-side. `store` defaults to `true`; pass `previous_interaction_id` on a later request to continue from a prior completed interaction — the Interactions equivalent of implicit context reuse. Setting `store: false` disables background execution AND `previous_interaction_id`. Stored interactions are retained **55 days on paid tiers, 1 day on the free tier**.
[source: ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]

### Interactions API streaming (primary surface)

Set `stream: true`. The SSE event flow is: `interaction.created` (metadata: ID, model, status), then per step a `step.start` (carrying the step type) followed by one or more `step.delta` (incremental data) and a `step.stop`, then `interaction.completed` (final `usage`). An `interaction.status_update` event carries status transitions, and an `error` event carries `{message, code}`. Every transcript ends with the SSE sentinel `event: done` / `data: [DONE]`.
[source: ai.google.dev/gemini-api/docs/streaming, retrieved 2026-07-19]

Delta types by step: `model_output` produces `text` / `image` / `audio`; `thought` produces `thought_signature` / `thought_summary`; `function_call` produces `arguments_delta` (a partial JSON string that must be accumulated across deltas); server-side tool steps produce tool-specific deltas (`google_search_call`, `google_search_result`, `code_execution_call`, `code_execution_result`).
[source: ai.google.dev/gemini-api/docs/streaming, retrieved 2026-07-19]

**Stream resume is distinct from `previous_interaction_id`.** Each SSE event carries an `event_id`; on disconnect, call `GET .../v1beta/interactions/{id}?stream=true&last_event_id=<event_id>` to replay from the next chunk. `previous_interaction_id` chains a new interaction onto a prior completed one — it is not a stream-resume mechanism. Do not conflate the two.
[source: ai.google.dev/api/interactions-api, retrieved 2026-07-19]
[source: ai.google.dev/gemini-api/docs/streaming, retrieved 2026-07-19]

With `stream: false` the API returns one `interaction` object with a `steps` array; each element is the fully assembled `step.start` -> `step.delta`(s) -> `step.stop` cycle.
[source: ai.google.dev/gemini-api/docs/streaming, retrieved 2026-07-19]

### Capabilities not yet supported on the Interactions API

As of retrieval, the Interactions API does NOT support: (1) `video_metadata` (custom frame-rate / clipping); (2) the Batch API; (3) automatic function calling in the Python SDK; (4) explicit caching (implicit caching via `previous_interaction_id` is available); (5) custom safety settings. For any of these, use the legacy `generateContent` surface.
[source: ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]

### Context caching (legacy `generateContent`)

Two tiers:

**Implicit caching** — automatic, no cost guarantee. Enabled by default on supported models.

| Model                     | Minimum cacheable tokens |
|---------------------------|--------------------------|
| `gemini-3.5-flash`        | 4096                     |
| `gemini-3.1-pro-preview`  | 4096                     |
| `gemini-2.5-flash`        | 2048                     |
| `gemini-2.5-pro`          | 2048                     |

The floors are per-model, not a clean Flash-vs-Pro split: `gemini-3.5-flash` rose 1024→4096, `gemini-2.5-flash` rose 1024→2048, and `gemini-2.5-pro` dropped 4096→2048 from the prior figures.
[source: ai.google.dev/gemini-api/docs/caching, retrieved 2026-07-18]

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

### Interactions API is now primary; `generateContent` is legacy

The Interactions API GA'd 2026-06-22 and is the recommended default for new development. `generateContent` is now labeled legacy but remains fully supported and continues to receive new mainline Gemini models; frontier long-running-model and agent capabilities are expected to land on the Interactions API first. Migration is not forced, but new agent-shaped work should start on Interactions.
[source: ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]
[source: ai.google.dev/gemini-api/docs/migrate-to-interactions, retrieved 2026-07-19]
[source: blog.google/innovation-and-ai/technology/developers-tools/interactions-api-general-availability/, retrieved 2026-07-19]

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

- **Per-model context window sizes** — `gemini-3.5-flash` is now published at **1,048,576 input tokens / 65,536 output tokens**. [source: ai.google.dev/gemini-api/docs/models/gemini-3.5-flash, retrieved 2026-07-18] The `gemini-3.1-pro-preview` and `gemini-3.1-flash-lite` windows are still not quoted in the retrieved model-page excerpts; confirm those at integration time.
- **Explicit-cache discount percentage** (often cited as 90% on Gemini 2.5+) is not replicated verbatim in the current caching-page primary excerpt; treat the percentage as community-reported until re-verified.
- **Vertex AI batch API parameter shape** is not covered here.
- **Live API** bi-directional audio protocol — session setup, VAD, turn-detection signals, interruption handling — is out of scope.
- **Gemini Deep Research, the Antigravity managed agent, and Computer Use** are documented in `resources/deep-research-agents.md` and `resources/agent-orchestration-surfaces.md`.
- **Safety settings** (`safetySettings` array, category / threshold enums) were not targeted in this retrieval pass.
- **OpenAI-compat thought-summary retrieval shape** — `extra_body.google.thinking_config.include_thoughts=true` enables summaries, but the response field/shape carrying them is not specified in the docs.
- **Rate-limit and quota specifics** per model / per tier are not covered.
- **Interactions API `v1` (stable) vs `v1beta` divergence** — the reference page notes a stable `v1` exists alongside the `v1beta` documented here; per-version field or behavior differences are not enumerated at `ai.google.dev/api/interactions-api`, checked 2026-07-19.
- **Interactions usage-object schema divergence** — `ai.google.dev/gemini-api/docs/tokens` documents a 6-key core usage schema (`total_input_tokens`, `total_output_tokens`, `total_thought_tokens`, `total_cached_tokens`, `total_tool_use_tokens`, `total_tokens`), while the `ai.google.dev/api/interactions-api` `Usage` schema additionally lists `cached_tokens_by_modality` and `grounding_tool_count`. Whether those two are populated in practice is unconfirmed; parse the usage object defensively.
