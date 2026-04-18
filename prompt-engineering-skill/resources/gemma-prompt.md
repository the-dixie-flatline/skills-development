---
family: gemma
scope: prompt
versions:
  - google/gemma-4-E2B
  - google/gemma-4-E2B-it
  - google/gemma-4-E4B
  - google/gemma-4-E4B-it
  - google/gemma-4-26B-A4B
  - google/gemma-4-26B-A4B-it
  - google/gemma-4-31B
  - google/gemma-4-31B-it
retrieved: 2026-04-19
primary_sources:
  - https://ai.google.dev/gemma/docs/core/model_card_4
  - https://ai.google.dev/gemma/docs/core/prompt-formatting-gemma4
  - https://ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4
  - https://huggingface.co/google/gemma-4-E4B-it
maturity_note: |
  Gemma 4 is Google's current open-weights family (knowledge cutoff January
  2025; model card page updated 2026-04-17). It is a breaking change from
  Gemma 3 in two respects: a new chat template (`<|turn>` / `<turn|>`
  replaces `<start_of_turn>` / `<end_of_turn>`), and native function calling
  with its own token protocol (Gemma 3 had no native tool-calling surface).
  Apache 2.0 license, no regional or MAU restrictions.
---

# Gemma — Prompt-Layer Reference

Portable prompting guidance for Gemma 4. API-layer detail (chat-template tokens, tool-call wire format, inference-stack deployment) lives in `gemma-prompt-api.md`.

## 1. Model Selection

Four sizes, each in base and instruction-tuned variants. The "E" in E2B / E4B stands for **effective** parameters — Per-Layer Embeddings (PLE) reduce the active memory footprint so a 5.1B-parameter model runs at ~2.3B effective cost.

| Model                             | Total / Effective / Active | Context | Modalities            | Deployment target                        |
|-----------------------------------|----------------------------|---------|-----------------------|------------------------------------------|
| `google/gemma-4-E2B-it`           | 5.1B / 2.3B / —            | 128K    | text, image, video, audio | Mobile, edge, on-device              |
| `google/gemma-4-E4B-it`           | 8B / 4.5B / —              | 128K    | text, image, video, audio | Laptops, small consumer GPUs         |
| `google/gemma-4-26B-A4B-it`       | 25.2B / — / 3.8B (MoE)     | 256K    | text, image, video        | Consumer GPU, workstation            |
| `google/gemma-4-31B-it`           | 30.7B / — / — (dense)      | 256K    | text, image, video        | Workstation, single-node data center |

[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

Notes:

- Only the E-series (E2B, E4B) supports **audio** input. The 26B MoE and 31B dense variants handle text, image, and video but not audio.
- 26B A4B is a mixture-of-experts with **8 active / 128 total + 1 shared** expert routing.
- All models are multimodal at training; base (non-`it`) and instruct (`-it`) variants are published.
- Apache 2.0 license with no 700M MAU clause and no regional restrictions — materially more permissive than Llama 4.

[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

## 2. Prompt Structure Conventions

Gemma 4 uses a new chat template that is not backward-compatible with Gemma 3.

### Roles

Three primary roles: `system`, `user`, `model`. **Gemma 4 uses `model`, not `assistant`.** Tool-interaction roles are handled via specialized tokens rather than additional role names.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

### Template tokens

The opening and closing tokens are asymmetric by design:

- **Open turn**: `<|turn>` (pipe after `<`)
- **Close turn**: `<turn|>` (pipe before `>`)

This is not a typo. Multiple primary sources confirm the asymmetry. (Exact token strings and full special-token table in `gemma-prompt-api.md`.)
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

### System prompt

Gemma 4 supports a `system` turn at the start of the conversation (Gemma 1 did not support a system role — this is a carry-forward from Gemma 3 onward). The system prompt is where tool declarations, thinking-mode enablement, and role/persona instructions go.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

### Idiomatic entry via `apply_chat_template`

In practice, do not build the raw token string by hand. Use `AutoProcessor.apply_chat_template(messages, ...)` with a message-dict list — the processor emits the correct tokens. Pass `enable_thinking=True|False` as a kwarg to control thinking mode.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

## 3. Instruction Patterns

### Thinking mode

Gemma 4 has an opt-in thinking mode (new relative to Gemma 3). Enablement is via a `<|think|>` token in the system prompt, or via `enable_thinking=True` when calling `apply_chat_template`. When enabled, reasoning is wrapped in a separate `<|channel>thought\n...<channel|>` block before the answer. When disabled, an empty channel block may still appear as a stability mechanism for larger models.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

**Critical multi-turn rule:** strip thoughts from prior-turn model messages before re-sending. Only tool-call sequences within a single turn retain thoughts inline; across user turns, historical thoughts should be removed. Leaving them in degrades behavior.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Multimodal placement

Place **image and audio content before text** in the same user message for optimal performance. Interleaving is supported — images and text can appear in any order — but performance is best with modality content front-loaded.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Sampling defaults

Gemma 4's standardized sampling configuration across all use cases:

- `temperature = 1.0`
- `top_p = 0.95`
- `top_k = 64`

This is a uniform recommendation — unlike Qwen (task-scoped tables) or Claude/OpenAI (parameter-free on flagships), Gemma gives one setting for everything.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Function calling as a native feature

Gemma 4 emits tool calls using dedicated `<|tool_call>...<tool_call|>` tokens and expects tool responses wrapped in `<|tool_response>...<tool_response|>`. Declarations accept either JSON Schema or Python function signatures with type hints (the processor auto-generates JSON Schema from Google-style docstrings). Wire format details in `gemma-prompt-api.md`.
[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

## 4. Context Window Practical Guidance

- **E2B / E4B**: 128K tokens native.
- **26B A4B / 31B Dense**: 256K tokens native.
- **Sliding window** sizes: 512 for E-series, 1024 for 26B/31B. Full global attention interleaves with local sliding-window layers; the final layer is always global.

[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

Implications:

- The hybrid attention design gives long-context capability with lightweight memory on the E-series — 128K on a laptop-class GPU is realistic for E2B/E4B with standard quantization.
- Multimodal token budgets count toward context; image resolution choices (see §5) materially shift how much text fits.

## 5. Multimodal Conventions

### Supported per variant

| Modality | E2B / E4B | 26B A4B / 31B |
|----------|-----------|---------------|
| Text     | ✓         | ✓             |
| Image    | ✓         | ✓             |
| Video    | ✓         | ✓             |
| Audio    | ✓         | ✗             |

[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]

### Input limits

- **Audio**: max **30 seconds** per segment.
- **Video**: max **60 seconds** at 1 fps assumed.
- **Image**: variable aspect ratio and resolution supported.

[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Image resolution token budgets

Gemma 4 supports a set of image-token budgets: **70, 140, 280, 560, 1120**. Lower budgets reduce context consumption and speed up inference; higher budgets preserve fine-grained detail for OCR, document parsing, or small-element recognition. Pick the smallest budget that captures the needed detail.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

### Placement and interleaving

Image / audio content **before** text in the same user message is the performance-optimal convention. Interleaving multiple modalities with text in between is supported but costs some quality.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

## 6. Behavioral Quirks

- **New chat template tokens break Gemma 3 compatibility.** Code paths that emit `<start_of_turn>...<end_of_turn>` will produce malformed prompts on Gemma 4. Upgrade by using the processor's `apply_chat_template` rather than string-building.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

- **Role name is `model`, not `assistant`.** Prompts copied from OpenAI / Anthropic wrappers will misparse.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

- **Thoughts leak across turns if not stripped.** Keeping thinking-mode output from prior turns in subsequent message history degrades performance. This differs from Claude (where thinking blocks *must* be preserved in tool-call turns) — Gemma's rule is the opposite for cross-turn history.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

- **Audio is E-series only.** Calling a 26B or 31B variant with audio input is undocumented territory.
[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]

- **String escaping inside tool calls uses `<|"|>`.** All string values in tool-call parameters and tool-response fields are wrapped with this delimiter. It is idiosyncratic; parsers that assume standard JSON escaping will fail.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

- **Native function calling is new in Gemma 4.** Code built around community-defined Gemma 3 tool templates will misbehave — Gemma 4 has its own first-party wire format.
[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

- **Apache 2.0 license with no 700M MAU clause.** Unlike Llama 4, there is no scale-based licensing threshold and no regional exclusion. Gemma usage policy covers prohibited-use categories but does not impose user-count ceilings.
[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]

## 7. Anti-Patterns

- **Do not build raw prompt strings from scratch** unless you also maintain a tokenizer-aware pipeline. Use `AutoProcessor.apply_chat_template(messages, add_generation_prompt=True, enable_thinking=...)`.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

- **Do not copy Gemma 3 chat-template tokens** into Gemma 4 prompts. `<start_of_turn>` / `<end_of_turn>` do not work; use `<|turn>` / `<turn|>`.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

- **Do not use `assistant` as a role name.** Use `model`.
[source: ai.google.dev/gemma/docs/core/prompt-formatting-gemma4, retrieved 2026-04-19]

- **Do not retain thinking-mode output from prior turns in multi-turn history.** Strip thoughts before re-sending.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

- **Do not assume a community Gemma 3 tool-calling template works.** Gemma 4 has a native protocol with dedicated tokens; use it.
[source: ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4, retrieved 2026-04-19]

- **Do not ship audio with the 26B or 31B variants.** Only E2B and E4B are audio-capable.
[source: ai.google.dev/gemma/docs/core/model_card_4, retrieved 2026-04-19]

- **Do not maximize image resolution by default.** Start at the smallest token budget that captures needed detail (70 / 140 / 280); escalate only when OCR or fine-detail tasks require it.
[source: huggingface.co/google/gemma-4-E4B-it, retrieved 2026-04-19]

## 8. Gaps

- **Exact release date for Gemma 4** is not quoted in the retrieved model-card excerpt (page update date 2026-04-17 was captured, but the actual initial release timing is not documented here).
- **Explicit vLLM / SGLang minimum versions** for Gemma 4 chat-template support were not captured.
- **FunctionGemma 270M variant** (`google/functiongemma-270m-it`) is mentioned as a dedicated function-calling-specialized model; its relationship to Gemma 4 proper and specialized use cases were not deeply fetched.
- **Vertex AI / Google Cloud hosted-API pricing** for Gemma 4 (if any) is not covered here — this reference focuses on open-weights deployment.
- **Empirical comparisons with Qwen3.6 and Llama 4** on the same benchmark set are not in retrieved primary sources.
- **Per-modality language scope** — the "35+ officially supported languages" claim applies to text; image and audio understanding language coverage is not separately quoted.
