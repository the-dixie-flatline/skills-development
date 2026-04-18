---
family: mistral
scope: api
versions:
  - mistral-small-2603
  - mistralai/Mistral-Small-4-119B-2603
  - mistral-large-2512
  - mistral-medium-2508
  - mistral-small-2506
  - ministral-3-14b-2512
  - ministral-3-8b-2512
  - ministral-3-3b-2512
  - magistral-medium-1.2-2509
  - magistral-small-1.2-2509
  - mistralai/Magistral-Small-2506
retrieved: 2026-04-19
primary_sources:
  - https://docs.mistral.ai/getting-started/models/models_overview/
  - https://docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates
  - https://huggingface.co/mistralai/Mistral-Small-4-119B-2603
  - https://mistral.ai/news/mistral-3
  - https://docs.mistral.ai/getting-started/changelog
maturity_note: |
  Mistral has both a closed API (La Plateforme) and open weights (HuggingFace,
  Apache 2.0). The wire API is OpenAI-compatible for messages, tools, and
  tool_choice, which makes OpenAI SDK code portable with a base_url swap.
  The open-weights side has its own tokenizer-level control-token protocol
  (`[INST]`, `[/INST]`, plus function-calling tokens) that callers interacting
  with the model directly (vLLM, transformers) must respect. `mistral-common`
  is the canonical reference implementation and is the right first choice
  rather than hand-built chat strings.
---

# Mistral — API-Layer Reference

API-call-level detail for current Mistral models. Portable prompt-layer content (model selection, reasoning semantics, anti-patterns) lives in `mistral-prompt.md`.

## 1. API Surface

### Endpoints

- **La Plateforme** — Mistral's native API at `https://api.mistral.ai/v1/...`. Chat completions, embeddings, FIM (fill-in-the-middle) for Codestral, OCR, moderation, audio.
- **OpenAI-compatible** — the chat completions endpoint accepts OpenAI SDK code with a base URL change and API-key swap.
- **Third-party hosts**: Amazon Bedrock, Azure Foundry, Hugging Face Inference, Modal, IBM WatsonX, OpenRouter, Fireworks, Together AI, Unsloth AI. Coming soon: NVIDIA NIM, AWS SageMaker.

[source: mistral.ai/news/mistral-3, retrieved 2026-04-19]

### SDKs

- **`mistralai` Python client** — first-party SDK.
- **`mistral-common`** — canonical tokenization / chat-template implementation. Use this when interacting with open weights directly.

[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

### Model-ID versioning

Mistral uses `YYMM` date suffixes (e.g. `mistral-small-2603` = March 2026, `magistral-medium-1.2-2509` = September 2025). Strip the suffix for alias-latest behavior on La Plateforme; pin the dated form for stable snapshots.
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

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
  "model": "mistral-small-2603",
  "messages": [
    { "role": "system",    "content": "..." },
    { "role": "user",      "content": "..." },
    { "role": "assistant", "content": "..." },
    { "role": "tool",      "content": "...", "tool_call_id": "..." }
  ],
  "reasoning_effort": "high"
}
```

The `tool` role at the API layer maps to the tokenizer's function-calling control tokens at rendering time.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

## 3. Sampling Parameters

Recommended sampling on Mistral Small 4, per mode:

| `reasoning_effort` | Temperature          | `do_sample` | max_new_tokens |
|--------------------|----------------------|-------------|----------------|
| `"none"`           | 0.0–0.7 (task-dependent) | `True`   | ~1024          |
| `"high"`           | 0.7                  | `True`      | ~1024+ (reasoning traces need headroom) |

[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

Per-model sampling defaults for other Mistral models (Large 3, Medium 3.1, Ministral 3, Magistral) are not uniformly published in the retrieved primary sources. Validate on workload.

## 4. Reasoning / Thinking Control

### Mistral Small 4 hybrid reasoning

[applies-to: mistralai/Mistral-Small-4-119B-2603, mistral-small-2603]

```json
{
  "model": "mistral-small-2603",
  "messages": [...],
  "reasoning_effort": "none" | "high"
}
```

`reasoning_effort` is **binary** on Small 4 — only `"none"` (fast, approximates Small 3.2 behavior) and `"high"` (deep reasoning, approximates Magistral behavior) are officially documented. OpenAI's full ladder (`minimal`/`low`/`medium`/`high`/`xhigh`) is not supported as-is.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]
[testable: id=mistral.small4-reasoning-effort-binary.v1, expected=request to mistral-small-2603 with reasoning_effort="medium" returns an error or is silently coerced to "none" or "high"]

### Magistral reasoning

[applies-to: magistral-medium-1.2-2509, magistral-small-1.2-2509, mistralai/Magistral-Small-2506]
Magistral reasons **by design** — no `reasoning_effort` toggle documented. The model is specialized for chain-of-thought across 8 languages (EN, FR, ES, DE, IT, AR, RU, zh-Hans).
[source: mistral.ai/news/magistral, retrieved 2026-04-19]

### vLLM reasoning-parser flag

For open-weights deployments that want the thinking trace separated from the final answer on the wire:

```
--reasoning-parser mistral
```

Use with Mistral Small 4 or Magistral; the parser extracts the reasoning segment.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

Exact reasoning-output tag format (whether wrapped in `<think>...</think>`, placed in a separate field, or otherwise) is not quoted in the retrieved primary sources; see §11 (Gaps).

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

- **Prompt caching on La Plateforme**: not prominently documented in retrieved primary sources. If caching applies, it is not surfaced via a `cache_control` field or a cached-token discount in the retrieved excerpts. Hosted third parties (Bedrock, Together) may apply their own caching.
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

### Magistral → Mistral Small 4 unification

[applies-to: mistralai/Mistral-Small-4-119B-2603, mistral-small-2603]
Mistral Small 4 (March 2026) unifies Magistral (reasoning), Pixtral (multimodal), and Devstral (code) into one hybrid model. Magistral 1.2 variants remain available; for new work prefer Small 4 with `reasoning_effort: "high"`.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

### Pixtral 12B (v1) deprecation

Pixtral 12B is marked deprecated on the Mistral news site. Migrate to Mistral Small 4 or Ministral 3 for multimodal workloads.
[source: mistral.ai/news/mistral-3, retrieved 2026-04-19]

### License tier distinction

- **Open (Apache 2.0)**: Mistral Large 3, Mistral Small 4, Mistral Small 3.2, Ministral 3 family, Magistral Small, Codestral (check per-version).
- **Premier (closed / commercial)**: Mistral Medium 3.1, Magistral Medium.

Re-verify per model at integration time.
[source: docs.mistral.ai/getting-started/models/models_overview/, retrieved 2026-04-19]
[source: mistral.ai/news/mistral-3, retrieved 2026-04-19]

## 10. Gaps

- **Reasoning output tag format** for Small 4's `"high"` mode is not quoted in retrieved primary sources. vLLM `--reasoning-parser mistral` implies a parseable structure; the exact emission wrapping (tag form, field name) needs a targeted retrieval.
- **Tool-calling control tokens at the tokenizer level** (`[AVAILABLE_TOOLS]`, `[TOOL_CALLS]`, `[TOOL_RESULTS]`, `[/TOOL_RESULTS]`) are community-known but not quoted verbatim with IDs in the retrieved primary chat-template page.
- **V7 tokenizer status** — references exist in community discussion; primary docs in the retrieved pass cover V1 through V3-Tekken only.
- **Structured outputs `response_format` semantics** (strict-mode support, schema subset, model-support matrix) were not retrieved in this pass.
- **La Plateforme prompt caching** (if any) — not documented in retrieved sources.
- **Batch API** parameter shape is not covered.
- **Per-model context windows** beyond Small 4's 256K are not uniformly quoted.
- **Image content-part shape** (detail parameter, resolution budgets) is undocumented in retrieved sources.
- **`guardrails` parameter rule schema** — structure of custom rules is not quoted.
- **Magistral reasoning output format** — whether the model wraps thoughts in a specific tag or emits them inline — is not quoted in retrieved sources.
- **Parallel tool-call disable flag** — whether `parallel_tool_calls: false` is accepted — is not explicitly documented.
- **Devstral 2 and Codestral 25.08 deployment flags and FIM (fill-in-the-middle) specifics** are covered in separate docs not fetched here.
