---
family: mistral
scope: prompt
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
  Mistral Medium 3.5 (`mistral-medium-3-5`, released 2026-04-27) is the
  current frontier model, released as open weights under a Modified MIT
  license with a 256K context window. Small 4 (`mistral-small-2603`) is the
  current open hybrid model unifying instruct, reasoning, and coding (256K
  context, 119B / 6.5B active). Adjustable reasoning is now unified across
  both via the `reasoning_effort` parameter, which on the chat endpoint
  accepts only `"high"` or `"none"` (not the OpenAI four-level scale). The
  Magistral 1.0-1.2 family has moved to Legacy/Deprecated; for native
  reasoning, the documented alternative is Mistral Medium 3.5 / Small 4.
  Version strings use a `YYMM` date suffix (e.g. `mistral-small-2603` =
  March 2026).
---

# Mistral — Prompt-Layer Reference

Portable prompting guidance for the current Mistral generation. API-layer detail (tokenizer versions, control tokens, vLLM flags, wire format) lives in `mistral-prompt-api.md`.

## 1. Model Selection

Mistral's lineup spans several tiers and family-specific variants. Pick by task and openness.

### Generalist text + vision

| Model                                          | Total / Active    | Context | License        | Notes                                          |
|------------------------------------------------|-------------------|---------|----------------|------------------------------------------------|
| Mistral Medium 3.5 (`mistral-medium-3-5`)      | —                 | 256K    | Modified MIT (open weights) | Current frontier model; released 2026-04-27       |
| Mistral Small 4 (`mistral-small-2603`)         | 119B / 6.5B (MoE) | 256K    | Open           | Hybrid instruct/reasoning/coding; March 2026   |

[source: docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04, retrieved 2026-06-01]
[source: docs.mistral.ai/models/model-cards/mistral-small-4-0-26-03, retrieved 2026-06-01]
[source: docs.mistral.ai/resources/changelogs, retrieved 2026-06-01]

The catalog also exposes earlier generation models (Mistral Large 3, Ministral 3 family, Mistral Small 3.2). Confirm current availability and licensing on the models overview before selecting one.
[source: docs.mistral.ai/models/overview, retrieved 2026-06-01]

### Reasoning-focused (Legacy)

The Magistral 1.0-1.2 family (`magistral-small-latest` / `magistral-medium-latest` and their dated snapshots) always generates reasoning traces, but the family is now in the Legacy/Deprecated table. For `magistral-medium-2509` the documented alternative is Mistral Medium 3.5.
[source: docs.mistral.ai/studio-api/conversations/reasoning, retrieved 2026-06-01]
[source: docs.mistral.ai/models/overview, retrieved 2026-06-01]

**Note**: Adjustable reasoning is now unified across `mistral-medium-3-5` and `mistral-small-latest` via `reasoning_effort`. For new reasoning work, use one of these with `reasoning_effort: "high"` rather than a Magistral snapshot.
[source: docs.mistral.ai/studio-api/conversations/reasoning/adjustable, retrieved 2026-06-01]

### Specialist

| Model                         | Purpose                                      |
|-------------------------------|----------------------------------------------|
| Devstral 2 (`v25.12`)         | Code agents; subsumed by Small 4 hybrid      |
| Codestral 25.08               | Fill-in-the-middle code completion           |
| OCR 3 (`v25.12`)              | OCR with table / header / hyperlink support  |
| Voxtral TTS (`voxtral-tts-2603`) | Text-to-speech, zero-shot voice cloning   |
| Voxtral Mini Transcribe 2 (`voxtral-mini-2602`) | Speech-to-text                |
| Leanstral (`labs-leanstral-2603`) | Lean 4 formal proof engineering          |
| Mistral Moderation (`mistral-moderation-2603`) | Content moderation              |

[source: docs.mistral.ai/getting-started/models/models_overview/, retrieved 2026-04-19]
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

### Selection rules

- **Frontier capability**: `mistral-medium-3-5` (Mistral Medium 3.5). Current frontier model, 256K context, open weights under a Modified MIT license.
[source: docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04, retrieved 2026-06-01]
- **New work, open hybrid model**: `mistral-small-2603` (Small 4). Hybrid instruct/reasoning/coding, 256K context, 119B / 6.5B active on MoE.
[source: docs.mistral.ai/models/model-cards/mistral-small-4-0-26-03, retrieved 2026-06-01]
- **Adjustable reasoning**: `mistral-medium-3-5` or `mistral-small-latest` with `reasoning_effort: "high"`. Native always-on reasoning (Magistral) is Legacy.
[source: docs.mistral.ai/studio-api/conversations/reasoning/adjustable, retrieved 2026-06-01]
- **Coding agents specifically**: Devstral 2 or Codestral 25.08.

## 2. Prompt Structure Conventions

Mistral has two layers to think about:

- **API-layer messages** are OpenAI-compatible (`{role, content}` arrays, `tools`, `tool_choice`, etc.). Prompts portable from OpenAI Chat Completions generally work. For the cross-family OpenAI-compatibility matrix, see `resources/openai-compatibility-surface.md` rather than duplicating it here.
- **Tokenizer-layer chat template** uses Mistral-specific control tokens (`[INST]`, `[/INST]`, plus `<s>`/`</s>`). Callers hitting open weights directly (vLLM, transformers) need to respect tokenizer version idiosyncrasies. Details in `mistral-prompt-api.md`.

The canonical implementation is [`mistral-common`](https://github.com/mistralai/mistral-common) — when in doubt, use it rather than building chat strings by hand.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

### System-prompt placement quirk

On tokenizer V2 and V3 (the current generations), **the system prompt is prepended to the LAST user message, not placed at the start of the conversation**. This is an important difference from OpenAI / Anthropic / Gemini conventions:

```
<s>[INST] user message[/INST] assistant message</s>[INST] system prompt

new user message[/INST]
```

V1 placed it at the first user message; V2+ reverses this. If you are building tokens by hand, get this right. If you are using `mistral-common`, it is handled automatically (and customizable).
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

### No `ipython`-style tool role at the tokenizer

Tool messages do not use a separate role like Llama's `ipython`. Mistral's function calling uses tokenizer-level control tokens (`[AVAILABLE_TOOLS]`, `[TOOL_CALLS]`, `[TOOL_RESULTS]`) inside the existing role structure. At the OpenAI-compatible API layer, the standard `tool` role with `tool_call_id` is used.

## 3. Instruction Patterns

### Adjustable reasoning via `reasoning_effort`

[applies-to: mistral-medium-3-5, mistral-small-2603]
Adjustable reasoning is available on `mistral-medium-3-5` and `mistral-small-latest` via `reasoning_effort`. On the chat endpoint, `reasoning_effort` accepts **only `"high"` or `"none"`** — not the OpenAI four-level (`low`/`medium`/`high`/`none`) scale. `"high"` emits a full thinking chunk before the answer; `"none"` omits the thinking chunk and applies minimal thinking. Treat the knob as binary on the cloud API.
[source: docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]
[source: docs.mistral.ai/studio-api/conversations/reasoning/adjustable, retrieved 2026-06-01]

Native reasoning models `magistral-small-latest` / `magistral-medium-latest` always generate reasoning traces and do not take a `reasoning_effort` toggle. The Magistral family is Legacy; prefer the adjustable-reasoning path above.
[source: docs.mistral.ai/studio-api/conversations/reasoning, retrieved 2026-06-01]

### Function calling as OpenAI-compatible

At the API layer, Mistral's function calling follows the OpenAI shape: `tools: [{"type": "function", "function": {...}}]`, responses include a `tool_calls` array with `{id, function: {name, arguments}}`, tool results come back in a `tool` role message with `tool_call_id`. Portable. Tokenizer-level control tokens are covered in `mistral-prompt-api.md`.

### Custom Guardrails (March 2026 addition)

Mistral exposes a `guardrails` parameter on chat completions and conversations endpoints (introduced 2026-03-12) that accepts custom moderation / safety rules configurable per-request. This is Mistral-specific and not portable from OpenAI.
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

## 4. Context Window Practical Guidance

- **Mistral Medium 3.5** (`mistral-medium-3-5`): 256K tokens.
[source: docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04, retrieved 2026-06-01]
- **Mistral Small 4** (`mistral-small-2603`): 256K tokens.
[source: docs.mistral.ai/models/model-cards/mistral-small-4-0-26-03, retrieved 2026-06-01]

The context-window table in the Known-Limitations page (Small/Medium 32,768; Large 131,072) is **outdated** — it contradicts the current Medium 3.5 and Small 4 model cards, both of which document 256K. Use the model cards, not the limitations table, for context budgeting.
[source: docs.mistral.ai/resources/known-limitations, retrieved 2026-06-01]

Prompt caching is available via the `prompt_cache_key` parameter on the chat endpoint; cached tokens are billed at 10% of the standard input price.
[source: docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]

## 5. Multimodal Conventions

- **Text + vision**: all Mistral 3 models + Small 4 + Magistral series.
- **Audio input / output**: via the **Voxtral** family (`voxtral-tts-2603`, `voxtral-mini-2602`) — separate endpoints, not inline parts on chat models.
- **OCR**: via the **OCR 3** specialist model, not inline on chat models.

[source: docs.mistral.ai/getting-started/models/models_overview/, retrieved 2026-04-19]
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

Vision input limits: maximum image size 20 MB; supported formats PNG, JPG, JPEG, GIF, WEBP.
[source: docs.mistral.ai/resources/known-limitations, retrieved 2026-06-01]

Exact image placement conventions inside the OpenAI-compatible `content` array (detail parameters, resolution budgets) are not fully documented in the retrieved primary sources. Community practice aligns with OpenAI's image content-part shape.

## 6. Behavioral Quirks

- **System prompt attaches to the LAST user message**, not the first, on V2+ tokenizers. Using `mistral-common` handles this; rolling your own chat-string assembly can place the system prompt incorrectly and degrade instruction following.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

- **`reasoning_effort` on the chat endpoint is binary**: only `"none"` and `"high"` are documented in the chat schema. OpenAI's four-level scale does not apply.
[source: docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]

- **Tokenizer whitespace rules differ across versions.** V1 has leading spaces after `<s>`; V2/V3 do not; Tekken (used by Mistral Nemo, Pixtral 12B) has no spaces around content at all. Mixing a V1-era chat template against a V2 tokenizer produces subtly worse outputs without errors.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

- **The Magistral 1.0-1.2 family is Legacy/Deprecated.** It still generates reasoning traces by design, but for new work the documented alternative is Mistral Medium 3.5 / Small 4 with `reasoning_effort: "high"`.
[source: docs.mistral.ai/models/overview, retrieved 2026-06-01]
[source: docs.mistral.ai/studio-api/conversations/reasoning, retrieved 2026-06-01]

- **License varies per model; the naming convention does not telegraph it.** Mistral Medium 3.5 is open weights under a Modified MIT license. Verify the license per model on the models overview before deployment planning.
[source: docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04, retrieved 2026-06-01]
[source: docs.mistral.ai/models/overview, retrieved 2026-06-01]

- **Pixtral 12B (v1) is deprecated.** Use a current model for multimodal on new work.

- **YYMM versioning** in model IDs: `-2603` = March 2026, `-2512` = December 2025. Strip the suffix (`mistral-small-2603` → `mistral-small`) to get alias-latest behavior on the La Plateforme API.

## 7. Anti-Patterns

- **Do not build raw chat strings by hand.** Use `mistral-common` (or the chat template shipped in each HF tokenizer). System-prompt placement and whitespace rules are version-dependent.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

- **Do not assume OpenAI's `reasoning_effort` scale on the chat endpoint.** The chat schema documents only `"high"` and `"none"`; do not send `"low"` or `"medium"`.
[source: docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]

- **Do not use `reasoning_effort` on native-reasoning or non-reasoning models.** It is meaningful on the adjustable-reasoning models (`mistral-medium-3-5`, `mistral-small-latest`). Magistral always reasons without it; non-reasoning models ignore it.
[source: docs.mistral.ai/studio-api/conversations/reasoning/adjustable, retrieved 2026-06-01]

- **Do not keep reasoning traces in multi-turn history** unless specifically required. Behavior on reasoning-retention across turns is not documented in the retrieved primary sources for Small 4; mirror OpenAI-reasoning-item discipline (preserve within tool loops, strip across user turns) until Mistral publishes explicit guidance.

- **Do not place the system prompt at the start of the conversation** expecting V1 semantics. On current tokenizers, `mistral-common` moves it to the last user message.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

- **Do not start new work on Pixtral 12B.** It is deprecated.

- **Do not assume `mistral-small` means "Small 4" forever.** The YYMM versioning rotates; pin `mistral-small-2603` explicitly if the hybrid reasoning behavior matters to your workload.
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

## 8. Gaps

- **Mistral Medium 3.5 parameter counts** (total / active) are not quoted in the retrieved model card.
- **Reasoning output tag format** for the `"high"` mode (whether wrapped in a specific tag or placed in a separate field) is not explicitly quoted in the retrieved primary sources.
- **Behavior when `reasoning_effort` is sent `"low"` or `"medium"`** on the chat endpoint is undocumented — the schema lists only `"high"` and `"none"`. Whether out-of-schema values are rejected or coerced is not stated.
- **Tool-calling control tokens at the tokenizer level** (`[AVAILABLE_TOOLS]`, `[TOOL_CALLS]`, `[TOOL_RESULTS]`) are referenced in community sources but not fully quoted in the retrieved primary chat-template page.
- **Structured outputs / JSON mode** parameter shape was not retrieved in this pass; expect OpenAI-compatible `response_format`-style semantics, unverified.
- **Per-image-count / total-payload vision limits** beyond the 20 MB per-image cap are not quoted.
