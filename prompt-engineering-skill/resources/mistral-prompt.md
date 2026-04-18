---
family: mistral
scope: prompt
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
  - devstral-2-2512
  - codestral-2508
retrieved: 2026-04-19
primary_sources:
  - https://docs.mistral.ai/getting-started/models/models_overview/
  - https://mistral.ai/news/mistral-3
  - https://huggingface.co/mistralai/Mistral-Small-4-119B-2603
  - https://docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates
  - https://docs.mistral.ai/getting-started/changelog
  - https://mistral.ai/news/magistral
maturity_note: |
  Mistral's current generation is "Mistral 3" (December 2025), anchored by
  Mistral Large 3 (open-weight MoE flagship) plus the Ministral 3 small
  family. **Mistral Small 4** (March 2026) is a significant newer release
  that unifies instruct, reasoning (Magistral), and coding (Devstral) into
  one hybrid model with a binary `reasoning_effort` toggle. Version strings
  use `YYMM` date-suffix (e.g. `mistral-small-2603` = March 2026). Most open
  models are Apache 2.0; "Premier" tier (Medium 3.1, Magistral Medium) is
  closed / commercial.
---

# Mistral — Prompt-Layer Reference

Portable prompting guidance for the current Mistral generation. API-layer detail (tokenizer versions, control tokens, vLLM flags, wire format) lives in `mistral-prompt-api.md`.

## 1. Model Selection

Mistral's lineup spans several tiers and family-specific variants. Pick by task and openness.

### Generalist text + vision

| Model                                              | Total / Active         | Context | Tier     | Notes                                    |
|----------------------------------------------------|------------------------|---------|----------|------------------------------------------|
| `mistralai/Mistral-Small-4-119B-2603` / `mistral-small-2603` | 119B / 6.5B (MoE) | 256K   | Open (Apache 2.0) | Hybrid instruct/reasoning/coding; March 2026 |
| Mistral Large 3 (`mistral-large-2512`)             | 675B / 41B (MoE)       | —       | Open (Apache 2.0) | December 2025 flagship                   |
| Mistral Medium 3.1 (`mistral-medium-2508`)         | —                      | —       | Premier (closed)  | Frontier multimodal, closed              |
| Mistral Small 3.2 (`mistral-small-2506`)           | —                      | —       | Open     | June 2025                                |
| Ministral 3 14B / 8B / 3B (`-2512` suffix)         | 14B / 8B / 3B (dense)  | —       | Open     | Text + vision; small-footprint tier      |

[source: docs.mistral.ai/getting-started/models/models_overview/, retrieved 2026-04-19]
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]
[source: mistral.ai/news/mistral-3, retrieved 2026-04-19]

### Reasoning-focused (pre-unification)

| Model                                     | Licensing     | Notes                                    |
|-------------------------------------------|---------------|------------------------------------------|
| Magistral Medium 1.2 (`v25.09`)           | Premier       | Reasoning-specialized; multimodal        |
| Magistral Small 1.2 (`v25.09`)            | Open (Apache 2.0) | 24B base (Magistral-Small-2506)     |

[source: docs.mistral.ai/getting-started/models/models_overview/, retrieved 2026-04-19]
[source: mistral.ai/news/magistral, retrieved 2026-04-19]

**Note**: Mistral Small 4 (March 2026) explicitly unifies Magistral's reasoning capabilities into a general-purpose model. For new work, Small 4 with `reasoning_effort: "high"` supersedes using Magistral directly unless pinning to a specific Magistral snapshot is required.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

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

- **New work, open weights, broad capabilities**: `mistral-small-2603` (Small 4). Hybrid reasoning, multimodal, 256K context, 6.5B active on MoE — one of the best quality-per-active-parameter deployments.
- **Frontier open-weight reasoning**: Mistral Large 3. 41B active / 675B total, Apache 2.0.
- **Frontier closed via API**: Mistral Medium 3.1.
- **Small footprint**: Ministral 3 3B / 8B / 14B.
- **Coding agents specifically**: Devstral 2 or Codestral 25.08.
- **Reasoning on Magistral lineage (legacy path)**: Magistral Small / Medium 1.2 if pinning is required; otherwise prefer Small 4.

## 2. Prompt Structure Conventions

Mistral has two layers to think about:

- **API-layer messages** are OpenAI-compatible (`{role, content}` arrays, `tools`, `tool_choice`, etc.). Prompts portable from OpenAI Chat Completions generally work.
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

### Hybrid reasoning on Small 4

[applies-to: mistral-small-2603]
Mistral Small 4 exposes `reasoning_effort` with **two documented values**: `"none"` (fast, no reasoning) and `"high"` (deep reasoning via Magistral-style chain-of-thought). OpenAI's full ladder (`minimal`/`low`/`medium`/`high`/`xhigh`) does **not** apply — treat the knob as binary.

Recommended sampling per mode:

| `reasoning_effort` | Temperature          | Notes                                       |
|--------------------|----------------------|---------------------------------------------|
| `"none"`           | 0.0–0.7 (task-dependent) | Approximates Mistral Small 3.2 behavior  |
| `"high"`           | 0.7                  | Deep reasoning; leaves room for exploration |

[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]
[testable: id=mistral.small4-effort-binary.v1, expected=request to mistral-small-2603 with reasoning_effort="medium" returns an error or is silently coerced; only "none" and "high" are officially documented]

### Multilingual reasoning via Magistral / Small 4

Magistral (and by extension Small 4's reasoning mode) is tuned for reasoning in English, French, Spanish, German, Italian, Arabic, Russian, and Simplified Chinese. Prompts in these languages benefit measurably; other languages fall back to general reasoning quality.
[source: mistral.ai/news/magistral, retrieved 2026-04-19]

### Function calling as OpenAI-compatible

At the API layer, Mistral's function calling follows the OpenAI shape: `tools: [{"type": "function", "function": {...}}]`, responses include a `tool_calls` array with `{id, function: {name, arguments}}`, tool results come back in a `tool` role message with `tool_call_id`. Portable. Tokenizer-level control tokens are covered in `mistral-prompt-api.md`.

### Custom Guardrails (March 2026 addition)

Mistral exposes a `guardrails` parameter on chat completions and conversations endpoints (introduced 2026-03-12) that accepts custom moderation / safety rules configurable per-request. This is Mistral-specific and not portable from OpenAI.
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

## 4. Context Window Practical Guidance

- **Mistral Small 4**: 256K tokens.
- Other current Mistral 3 models: context windows not uniformly quoted in the retrieved models-overview excerpt; confirm per-model on the models page before budgeting.

[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

No provider-side prompt-caching pricing is prominently documented in the retrieved sources (in contrast to OpenAI's 90% cached-input discount or Anthropic's `cache_control`). Multi-platform hosts (Bedrock, Azure, Together) may apply their own caching.

## 5. Multimodal Conventions

- **Text + vision**: all Mistral 3 models + Small 4 + Magistral series.
- **Audio input / output**: via the **Voxtral** family (`voxtral-tts-2603`, `voxtral-mini-2602`) — separate endpoints, not inline parts on chat models.
- **OCR**: via the **OCR 3** specialist model, not inline on chat models.

[source: docs.mistral.ai/getting-started/models/models_overview/, retrieved 2026-04-19]
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

Exact image placement conventions inside the OpenAI-compatible `content` array (detail parameters, resolution budgets) are not fully documented in the retrieved primary sources. Community practice aligns with OpenAI's image content-part shape.

## 6. Behavioral Quirks

- **System prompt attaches to the LAST user message**, not the first, on V2+ tokenizers. Using `mistral-common` handles this; rolling your own chat-string assembly can place the system prompt incorrectly and degrade instruction following.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

- **`reasoning_effort` on Small 4 is binary-ish**: only `"none"` and `"high"` are officially documented. OpenAI's full ladder does not map cleanly.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

- **Tokenizer whitespace rules differ across versions.** V1 has leading spaces after `<s>`; V2/V3 do not; Tekken (used by Mistral Nemo, Pixtral 12B) has no spaces around content at all. Mixing a V1-era chat template against a V2 tokenizer produces subtly worse outputs without errors.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

- **Magistral is superseded for new work by Small 4's hybrid mode** but remains available. Version pinning to Magistral Small 1.2 / Medium 1.2 is supported; new workloads should prefer Small 4 unless there's a specific reason to lock to Magistral behavior.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

- **Premier vs Open license distinction matters for deployment planning.** Mistral Large 3, Small 4, Ministral 3, Magistral Small are Apache 2.0 open-weight. Medium 3.1, Magistral Medium, and some specialist models are "Premier" (closed / API-only or commercial license). The naming convention does not telegraph this — verify on the models page.
[source: docs.mistral.ai/getting-started/models/models_overview/, retrieved 2026-04-19]

- **Pixtral 12B (v1) is deprecated.** Use Mistral Small 4 or Ministral 3 for multimodal on new work. Pixtral Large (if still listed) predates Mistral Small 4's unification.

- **YYMM versioning** in model IDs: `-2603` = March 2026, `-2512` = December 2025, `-2508` = August 2025. Strip the suffix (`mistral-small-2603` → `mistral-small`) to get alias-latest behavior on the La Plateforme API.

## 7. Anti-Patterns

- **Do not build raw chat strings by hand.** Use `mistral-common` (or the chat template shipped in each HF tokenizer). System-prompt placement and whitespace rules are version-dependent.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

- **Do not assume OpenAI's `reasoning_effort` ladder on Small 4.** Only `"none"` and `"high"` are documented. Sending `"medium"` or `"low"` is not guaranteed.
[source: huggingface.co/mistralai/Mistral-Small-4-119B-2603, retrieved 2026-04-19]

- **Do not use `reasoning_effort` on non-hybrid models.** Magistral reasons without it; Ministral 3 family and Codestral don't reason. The parameter is meaningful only on Small 4's hybrid.

- **Do not keep reasoning traces in multi-turn history** unless specifically required. Behavior on reasoning-retention across turns is not documented in the retrieved primary sources for Small 4; mirror OpenAI-reasoning-item discipline (preserve within tool loops, strip across user turns) until Mistral publishes explicit guidance.

- **Do not place the system prompt at the start of the conversation** expecting V1 semantics. On current tokenizers, `mistral-common` moves it to the last user message.
[source: docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates, retrieved 2026-04-19]

- **Do not start new work on Pixtral 12B.** It is deprecated.

- **Do not assume `mistral-small` means "Small 4" forever.** The YYMM versioning rotates; pin `mistral-small-2603` explicitly if the hybrid reasoning behavior matters to your workload.
[source: docs.mistral.ai/getting-started/changelog, retrieved 2026-04-19]

## 8. Gaps

- **Exact context window for Mistral Large 3 and Medium 3.1** is not quoted in the retrieved models-overview excerpt.
- **Reasoning output tag format** for Small 4's `"high"` mode (is it wrapped in `<think>...</think>`? a separate field?) is not explicitly shown in the retrieved primary sources. The vLLM `--reasoning-parser mistral` flag implies a parseable structure; the exact emission format needs a targeted retrieval.
- **Tool-calling control tokens at the tokenizer level** (`[AVAILABLE_TOOLS]`, `[TOOL_CALLS]`, `[TOOL_RESULTS]`) are referenced in community sources but not fully quoted in the retrieved primary chat-template page (the cookbook explicitly defers to external references for function calling).
- **V7 tokenizer status** — community references exist; primary docs in the retrieved pass cover V1, V2, V3, V3-Tekken only.
- **Structured outputs / JSON mode** parameter shape was not retrieved in this pass; expect OpenAI-compatible `response_format`-style semantics, unverified.
- **Prompt caching on La Plateforme** (if any) is not documented in retrieved sources.
- **Pixtral Large current status** vs Mistral Small 4 is not fully clarified.
- **Vision input content-part shape** (detail parameters, resolution limits) is not quoted.
- **`guardrails` parameter structure** (shape of custom rules) is not covered in retrieved excerpts beyond its existence.
