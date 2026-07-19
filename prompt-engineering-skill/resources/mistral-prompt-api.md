---
family: mistral
scope: api
versions:
  - mistral-medium-3-5
  - mistral-small-2603
retrieved: 2026-06-01
primary_sources:
  - https://docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04
  - https://docs.mistral.ai/models/model-cards/mistral-small-4-0-26-03
  - https://docs.mistral.ai/models/overview
  - https://docs.mistral.ai/resources/changelogs
  - https://docs.mistral.ai/api/endpoint/chat
  - https://docs.mistral.ai/studio-api/conversations/reasoning
  - https://docs.mistral.ai/studio-api/conversations/reasoning/adjustable
  - https://docs.mistral.ai/resources/migration-guides
  - https://docs.mistral.ai/resources/known-limitations
  - https://docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates
maturity_note: |
  Mistral has both a hosted API (La Plateforme) and open weights. The wire
  API is OpenAI-compatible for messages, tools, and tool_choice, so OpenAI
  SDK code is portable with a base_url swap to `https://api.mistral.ai/v1`.
  Mistral Medium 3.5 (`mistral-medium-3-5`, released 2026-04-27, Modified MIT,
  256K) is the current frontier model; Small 4 (`mistral-small-2603`, 256K) is
  the current open hybrid model. Adjustable reasoning is unified across both
  via `reasoning_effort`, which on the chat endpoint accepts only `"high"` or
  `"none"`. The chat schema documents `max_tokens` (not OpenAI's
  `max_completion_tokens`). The Magistral 1.0-1.2 family is Legacy/Deprecated.
  The open-weights side has a tokenizer-level control-token protocol
  (`[INST]`, `[/INST]`, plus function-calling tokens); `mistral-common` is the
  canonical reference implementation.
---

# Mistral — API-Layer Reference

API-call-level detail for current Mistral models. Portable prompt-layer content (model selection, reasoning semantics, anti-patterns) lives in `mistral-prompt.md`.

## 1. API Surface

### Endpoints

- **La Plateforme** — Mistral's native API at `https://api.mistral.ai/v1/...`. Chat completions, embeddings, FIM (fill-in-the-middle) for Codestral, OCR, moderation, audio.
- **OpenAI-compatible** — same Chat Completions request structure; point `base_url` at `https://api.mistral.ai/v1`, swap the API key, and change the model name. OpenAI SDK code is then portable.
[source: docs.mistral.ai/resources/migration-guides, retrieved 2026-06-01]
- **Third-party hosts**: Amazon Bedrock, Azure Foundry, Hugging Face Inference, Modal, IBM WatsonX, OpenRouter, Fireworks, Together AI, Unsloth AI. Coming soon: NVIDIA NIM, AWS SageMaker.
[source: mistral.ai/news/mistral-3, retrieved 2026-04-19]

For the cross-family OpenAI-compatibility matrix, see `resources/openai-compatibility-surface.md` rather than duplicating it here.

### SDKs

- **`mistralai` Python client** — first-party SDK.
- **`mistral-common`** — canonical tokenization / chat-template implementation. Use this when interacting with open weights directly.

[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

### Model-ID versioning

Mistral uses `YYMM` date suffixes (e.g. `mistral-small-2603` = March 2026). The catalog also exposes incrementing aliases such as `mistral-medium-3-5` (`mistral-medium-3-5+1`) and `mistral-small-2603` (`mistral-small-2603+1`). Strip the suffix for alias-latest behavior on La Plateforme; pin the dated form for stable snapshots.
[source: docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04, retrieved 2026-06-01]
[source: docs.mistral.ai/models/model-cards/mistral-small-4-0-26-03, retrieved 2026-06-01]

## 2. Chat Template / Message Structure

Two distinct layers:

- **API layer**: OpenAI-compatible `messages` array with `user` / `assistant` / `system` / `tool` roles.
- **Tokenizer layer**: Mistral-specific control tokens (`[INST]`, `[/INST]`, plus function-calling tokens), applied by `mistral-common` or HuggingFace chat templates.

### Tokenizer versions

| Version  | Used by                                            | Notes                                              |
|----------|----------------------------------------------------|----------------------------------------------------|
| V1       | Mistral 7B V1/V2, Mixtral 8x7B V1                  | SentencePiece; leading spaces around tokens        |
| V2       | Mistral Small 2402, Mistral Large 2402             | Introduces control tokens; adds function calling   |
| V3       | Mixtral 8x22B, Codestral, Mathstral, Small 2409, Large 2407 | Enhanced tool use over V2                 |
| V3-Tekken | Mistral Nemo 12B, Pixtral 12B                     | TikToken-based; no spaces around tokens            |

[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

V7 is referenced in community discussion but not documented in the retrieved primary chat-template pages. Treat as unverified.
[unverified] A V7 tokenizer with further function-calling / reasoning protocol changes exists.

Current-generation models (Mistral 3 family, Small 4, Magistral 1.2) use `mistral-common >= 1.10.0`; the models_overview and Small 4 card require that version without labeling it explicitly as "V7". Refer to `mistral-common` as the source of truth.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

### Core control tokens

| Token        | Purpose                                              | V2/V3 token ID |
|--------------|------------------------------------------------------|----------------|
| `<s>`        | Beginning-of-sequence (BOS)                          | 1              |
| `</s>`       | End-of-sequence (EOS)                                | 2              |
| `[INST]`     | Open user-message block                              | 3              |
| `[/INST]`    | Close user-message block                             | 4              |

Additional tool-calling control tokens (`[AVAILABLE_TOOLS]`, `[/AVAILABLE_TOOLS]`, `[TOOL_CALLS]`, `[TOOL_RESULTS]`, `[/TOOL_RESULTS]`) are referenced in community and HF discussions but not quoted verbatim in the retrieved primary chat-template page. Treat exact token IDs for these as partial.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

### Basic chat template (V2 / V3)

```
<s>[INST] user message[/INST] assistant message</s>[INST] new user message[/INST]
```

Spaces follow `[INST]` and `[/INST]`; no space after `<s>` or `</s>`.

### Basic chat template (V3-Tekken — Nemo, Pixtral 12B)

```
<s>[INST]user message[/INST]assistant message</s>[INST]new user message[/INST]
```

No spaces around content or control tokens.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

### System-prompt placement

On V2 and V3, the system prompt is prepended to the **last** user message by `mistral-common` default (customizable):

```
<s>[INST] user message[/INST] assistant message</s>[INST] system prompt

new user message[/INST]
```

V1 placed it at the first user message instead. This is a breaking difference between generations.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]
[testable: id=mistral.system-prompt-last-user.v1, expected=mistral-common-rendered prompt places system-prompt content immediately before the last user turn's [INST] content, not at the start of the conversation]

### API-layer message shape (OpenAI-compatible)

```json
{
  "model": "mistral-medium-3-5",
  "messages": [
    { "role": "system",    "content": "..." },
    { "role": "user",      "content": "..." },
    { "role": "assistant", "content": "..." },
    { "role": "tool",      "content": "...", "tool_call_id": "..." }
  ],
  "reasoning_effort": "high",
  "max_tokens": 1024
}
```

Use `max_tokens` for the completion-length cap. The chat schema documents `max_tokens` ("the maximum number of tokens to generate in the completion") and has **no** `max_completion_tokens` field — Mistral has not adopted OpenAI's rename; send `max_tokens`.
[source: docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]

The `tool` role at the API layer maps to the tokenizer's function-calling control tokens at rendering time.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

## 3. Sampling Parameters

- **`max_tokens`** caps completion length. The chat schema documents `max_tokens`; there is no `max_completion_tokens` field. Send `max_tokens`.
[source: docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]
- When `reasoning_effort` is `"high"`, leave headroom in `max_tokens` for the thinking chunk that precedes the answer; with `"none"` the thinking chunk is omitted.
[source: docs.mistral.ai/studio-api/conversations/reasoning/adjustable, retrieved 2026-06-01]

Per-mode and per-model temperature defaults are not uniformly published in the retrieved primary sources. Validate on workload.

## 4. Reasoning / Thinking Control

### Adjustable reasoning (`reasoning_effort`)

[applies-to: mistral-medium-3-5, mistral-small-2603]

```json
{
  "model": "mistral-medium-3-5",
  "messages": [...],
  "reasoning_effort": "high" | "none"
}
```

Adjustable reasoning is available on `mistral-medium-3-5` and `mistral-small-latest`. On the chat endpoint, `reasoning_effort` accepts **only `"high"` or `"none"`** — not the OpenAI four-level (`low`/`medium`/`high`/`none`) scale. `"high"` emits a full thinking chunk before the answer; `"none"` omits the thinking chunk and applies minimal thinking.
[source: docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]
[source: docs.mistral.ai/studio-api/conversations/reasoning/adjustable, retrieved 2026-06-01]
[testable: id=mistral.reasoning-effort-high-emits-thinking.v1, expected=request with reasoning_effort="high" returns a thinking chunk before the answer; reasoning_effort="none" returns an answer with the thinking chunk omitted]
Verified 2026-07-19: mistral-medium-3-5 with `reasoning_effort`/`reasoning.effort` `"high"` returned a reasoning trace (3/3, ~3.4k reasoning tokens); `"none"` returned no reasoning trace (3/3, 0 tokens). Via OpenRouter to Mistral first-party. The binary high/none reasoning toggle is confirmed.

### Magistral reasoning (Legacy)

[applies-to: magistral-medium-2509]
Native reasoning models `magistral-small-latest` / `magistral-medium-latest` always generate reasoning traces — no `reasoning_effort` toggle. The Magistral 1.0-1.2 family is in the Legacy/Deprecated table; the documented alternative for `magistral-medium-2509` is Mistral Medium 3.5.
[source: docs.mistral.ai/studio-api/conversations/reasoning, retrieved 2026-06-01]
[source: docs.mistral.ai/models/overview, retrieved 2026-06-01]

### vLLM reasoning-parser flag

For open-weights deployments that want the thinking trace separated from the final answer on the wire:

```
--reasoning-parser mistral
```

Use with the open-weights models; the parser extracts the reasoning segment.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

Exact reasoning-output tag format (whether wrapped in `<think>...</think>`, placed in a separate field, or otherwise) is not quoted in the retrieved primary sources; see §10 (Gaps).

## 5. Tool Use / Function Calling

### API layer (OpenAI-compatible)

Expected wire format:

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get current weather",
        "parameters": { "type": "object", "properties": {...}, "required": [...] }
      }
    }
  ],
  "tool_choice": "auto"
}
```

Response includes `tool_calls`:

```json
{
  "tool_calls": [
    {
      "id": "call_abc",
      "type": "function",
      "function": { "name": "get_weather", "arguments": "{\"location\":\"Paris\"}" }
    }
  ]
}
```

Tool results come back as a `tool` role message with `tool_call_id` referencing the call. This is the standard OpenAI-compatible shape and is covered in the HF Small 4 card's agentic examples.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

### Tokenizer-layer control tokens

At the open-weights rendering layer, tool definitions and calls are wrapped in dedicated control tokens. Known / community-referenced tokens:

- `[AVAILABLE_TOOLS]` / `[/AVAILABLE_TOOLS]` — tool-definition block.
- `[TOOL_CALLS]` — emitted model tool call.
- `[TOOL_RESULTS]` / `[/TOOL_RESULTS]` — tool-response injection.

Exact IDs and rendering examples were not quoted in the retrieved chat-template primary page (the cookbook defers to function-calling docs and `mistral-common`). Rely on `mistral-common` for correct emission.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

### vLLM / SGLang parser flags

For open-weights deployments:

```
--tool-call-parser mistral
--enable-auto-tool-choice
```

These enable automatic tool-call extraction from the tokenizer-layer emissions.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

### Parallel tool calls

Supported on current Mistral models via multiple `tool_calls` entries in a single assistant response. Specific disable-flag (e.g. `parallel_tool_calls: false`) is not quoted in the retrieved primary sources.

## 6. Structured Outputs

Supported at the API level via OpenAI-compatible `response_format` (e.g. `{"type": "json_object"}` or `{"type": "json_schema", ...}`). Exact schema-constraint semantics, strict-mode behavior, and model-support matrix were not captured in detail in this retrieval pass.
[unverified] `response_format: {"type": "json_schema", "json_schema": {...}}` with strict schema enforcement is supported on Mistral Small 4 and Mistral Large 3.

For constrained JSON output without using the schema field directly, JSON-mode (`{"type": "json_object"}`) is a safer bet on models where strict schema support is undocumented.

## 7. Caching, Batch, Streaming

- **Prompt caching**: supported via the `prompt_cache_key` parameter on the chat endpoint. Cached tokens are billed at 10% of the standard input price.
[source: docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]
- **Batch API**: not covered in retrieved sources.
- **Streaming**: SSE streaming is supported via the OpenAI-compatible chat completions endpoint, standard event types (`chunk`, tool-call events).

## 8. Deployment Flags

### Mistral Small 4 on vLLM

Recommended serve command from the HF model card:

```
vllm serve mistralai/Mistral-Small-4-119B-2603 \
    --max-model-len 262144 \
    --tensor-parallel-size 2 \
    --attention-backend FLASH_ATTN_MLA \
    --tool-call-parser mistral \
    --enable-auto-tool-choice \
    --reasoning-parser mistral \
    --max_num_batched_tokens 16384 \
    --max_num_seqs 128 \
    --gpu_memory_utilization 0.8
```

Docker: `mistralllm/vllm-ms4:latest`. As of the model release (2026-03-16), vLLM support was awaiting PR merge; use the Docker image for immediate deployment.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

### Quantization variants

- `mistralai/Mistral-Small-4-119B-2603-NVFP4` — NVFP4 4-bit quantization.
- `mistralai/Mistral-Small-4-119B-2603-eagle` — Eagle head for speculative decoding.

[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

### transformers

```bash
uv pip install git+https://github.com/huggingface/transformers.git
uv pip install "mistral_common>=1.10.0"
```

[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

### Guardrails parameter (La Plateforme, March 2026)

```json
{
  "model": "...",
  "messages": [...],
  "guardrails": { /* custom rules */ }
}
```

The `guardrails` parameter was introduced on conversations and chat completions endpoints on 2026-03-12. Custom guardrails can also be configured directly on Agents. Exact rule-schema shape was not captured in this retrieval pass.
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

## 9. Deprecations and Breaking Changes

### Tokenizer changes across versions

[applies-to: tokenizer V2, V3, V3-Tekken]
- V1 → V2+: system-prompt placement moved from first user message to last user message.
- V2 → V3: enhanced tool-calling control tokens (exact new tokens not fully quoted in retrieved primary sources).
- V3 → V3-Tekken: TikToken base; whitespace handling changes (no spaces around content).

Mixing a V1-era chat template rendering against a V2+ tokenizer silently degrades output quality without errors.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

### Magistral family → Legacy/Deprecated

[applies-to: magistral-medium-2509]
The Magistral 1.0-1.2 family is in the Legacy/Deprecated table. Native reasoning still works (the models always emit traces), but the documented alternative for `magistral-medium-2509` is Mistral Medium 3.5. Adjustable reasoning via `reasoning_effort` on `mistral-medium-3-5` / `mistral-small-latest` is the current path.
[source: docs.mistral.ai/models/overview, retrieved 2026-06-01]
[source: docs.mistral.ai/studio-api/conversations/reasoning, retrieved 2026-06-01]

### Pixtral 12B (v1) deprecation

Pixtral 12B is marked deprecated on the Mistral news site. Migrate to a current model for multimodal workloads.
[source: mistral.ai/news/mistral-3, retrieved 2026-04-19]

### License

- **Mistral Medium 3.5** (`mistral-medium-3-5`): released as open weights under a **Modified MIT** license.
[source: docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04, retrieved 2026-06-01]

Re-verify the license per model at integration time.
[source: docs.mistral.ai/models/overview, retrieved 2026-06-01]

## 10. Gaps

- **`max_completion_tokens` handling** — the chat schema omits the field; whether the endpoint rejects, ignores, or silently maps it is undocumented. Send `max_tokens`.
- **`reasoning_effort` out-of-schema values** — the chat schema lists only `"high"` and `"none"`; whether `"low"` / `"medium"` are rejected or coerced is not stated.
- **Reasoning output tag format** for the `"high"` mode is not quoted in retrieved primary sources. vLLM `--reasoning-parser mistral` implies a parseable structure; the exact emission wrapping (tag form, field name) needs a targeted retrieval.
- **Tool-calling control tokens at the tokenizer level** (`[AVAILABLE_TOOLS]`, `[TOOL_CALLS]`, `[TOOL_RESULTS]`, `[/TOOL_RESULTS]`) are community-known but not quoted verbatim with IDs in the retrieved primary chat-template page.
- **Structured outputs `response_format` semantics** (strict-mode support, schema subset, model-support matrix) were not retrieved in this pass.
- **Batch API** parameter shape is not covered.
- **Mistral Medium 3.5 parameter counts** (total / active) are not quoted in the retrieved model card.
- **Image content-part shape** (detail parameter, per-request count limits beyond the 20 MB per-image cap) is not fully documented in retrieved sources.
- **Parallel tool-call disable flag** — whether `parallel_tool_calls: false` is accepted — is not explicitly documented.
