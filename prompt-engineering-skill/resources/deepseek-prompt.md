---
family: deepseek
scope: prompt
versions:
  - deepseek-chat
  - deepseek-reasoner
  - deepseek-ai/DeepSeek-V3.2
  - deepseek-ai/DeepSeek-V3.2-Speciale
retrieved: 2026-04-19
primary_sources:
  - https://api-docs.deepseek.com/news/news251201
  - https://api-docs.deepseek.com/guides/reasoning_model
  - https://api-docs.deepseek.com/guides/thinking_mode
  - https://api-docs.deepseek.com/guides/function_calling
  - https://api-docs.deepseek.com/guides/kv_cache
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.2
maturity_note: |
  DeepSeek-V3.2 (released 2025-12-01) is the current generation. The API
  exposes it as two model IDs at the same underlying weights: `deepseek-chat`
  (non-thinking) and `deepseek-reasoner` (thinking). A higher-compute
  variant `DeepSeek-V3.2-Speciale` is API-only and does not support tool
  calls. DeepSeek V4 is unreleased as of this retrieval (leaks suggest April
  2026). The open weights are MIT-licensed — the most permissive of any
  frontier-scale model family covered in this library. Models are text-only
  (no multimodal support).
---

# DeepSeek — Prompt-Layer Reference

Portable prompting guidance for the current DeepSeek generation. API-layer detail (chat template tokens, API shapes, caching mechanics) lives in `deepseek-prompt-api.md`.

## 1. Model Selection

Current generation is organized around DeepSeek-V3.2. Same underlying weights, two API personas.

| API Model ID          | Mode                  | Notes                                                           |
|-----------------------|-----------------------|-----------------------------------------------------------------|
| `deepseek-chat`       | Non-thinking          | Fast path; no CoT output                                        |
| `deepseek-reasoner`   | Thinking              | Chain-of-thought reasoning; returns `reasoning_content` field   |
| `deepseek-v3.2-speciale` | High-compute reasoning | API-only; **no tool calls**; olympiad-level reasoning        |

[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]

### Open weights

- **`deepseek-ai/DeepSeek-V3.2`** — HF, MIT license, 685B total parameters, DeepSeek Sparse Attention (DSA) architecture.

[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### Legacy / predecessors

- DeepSeek-V3.1 (`deepseek-ai/DeepSeek-V3.1`) — previous generation.
- DeepSeek-R1 family — pure reasoning model from January 2025, superseded by V3.2's integrated thinking.

### Selection rules

- **General use** — `deepseek-chat` on API, or V3.2 open weights with the non-thinking template.
- **Reasoning workloads** — `deepseek-reasoner`. Thinking is internal; caller receives separated `reasoning_content` + `content` fields.
- **Hardest reasoning / olympiad-level** — `deepseek-v3.2-speciale`. Give up tool calling; get maximum reasoning depth.
- **Self-hosted** — the 685B open weights (MIT) — most permissive license of any frontier-scale model in this reference library.
- **Multimodal use** — DeepSeek is **text-only** on current versions. Not the right family.

## 2. Prompt Structure Conventions

The API layer is OpenAI-compatible. Prompts portable from OpenAI Chat Completions generally work on `deepseek-chat` with a `base_url` swap.

At the open-weights level, DeepSeek uses a distinctive special-token chat template — visually different from most other open-weights families. Full-width pipe characters (`｜`) and underscore characters (`▁`) appear in control tokens. Exact tokens in `deepseek-prompt-api.md`.

### Roles

- `user` / `<｜User｜>` at tokenizer level.
- `assistant` / `<｜Assistant｜>` at tokenizer level.
- `developer` / `<｜Developer｜>` — **used exclusively for search agent scenarios** on V3.2. Not a general-purpose role (unlike OpenAI's `developer` role).

[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### No Jinja template in the tokenizer config

DeepSeek-V3.2 deliberately ships **no Jinja-format chat template** in its tokenizer config. The HF model card explicitly directs callers to a Python encoding script (`encoding/encoding_dsv32.py`) with `encode_messages()` and `parse_message_from_completion_text()` functions. This is unusual and forces integrators using HuggingFace's `apply_chat_template` to supply their own template or use DeepSeek's Python helper.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

## 3. Instruction Patterns

### Thinking via model ID or parameter toggle

Two equivalent ways to enable reasoning on the API:

- Select the model: `"model": "deepseek-reasoner"`.
- Stay on `deepseek-chat` but pass `thinking: {"type": "enabled"}` via the SDK's `extra_body`.

The second form mirrors Anthropic's `thinking` parameter — familiar shape, different semantics (DeepSeek is binary enabled/disabled, no `budget_tokens` or `effort` ladder).
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-04-19]

### Thinking with tools — new in V3.2

V3.2 is the first DeepSeek model to integrate thinking directly into tool-use. The model can reason, call a tool, reason again on the result, and continue — all within a single thinking-enabled turn. This is a material capability change from V3.1 and R1 (which did not support tool calling inside thinking).
[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]

**Exception**: `deepseek-v3.2-speciale` does **not support tool calls** at all.
[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]

### Reasoning content must not return to input

`reasoning_content` in the API response is **not** a re-sendable artifact. Including it in the next request's message history returns a **400 error**. Strip it before continuing the conversation. Only the final `content` goes back.

This is the opposite of Claude's signature-authenticated `thinking` blocks (which must be preserved across tool-call turns) and the opposite of OpenAI's `previous_response_id` / explicit reasoning-item pattern. Portable-prompt pipelines must special-case DeepSeek.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]
[testable: id=deepseek.reasoning-content-rejected-on-input.v1, expected=sending a prior turn's reasoning_content in messages[].content or alongside content returns HTTP 400]

### Function calling as OpenAI-compatible

`tools` array with OpenAI-shape function definitions. `strict: true` enforces schema compliance (via the beta endpoint). DeepSeek supports richer schema features than OpenAI's subset — `$def`, `$ref`, recursive definitions — useful for structured data extraction with shared sub-schemas.
[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

## 4. Context Window Practical Guidance

Exact context-window size for DeepSeek-V3.2 is not quoted in the retrieved primary excerpts. What is documented:

- `max_tokens` on `deepseek-reasoner` defaults to 32K and allows up to **64K**. The CoT content counts toward `max_tokens`.
- Prompt caching is **automatic on disk** — minimum cacheable 64 tokens; up to ~90% cost reduction on cache hit.

[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]
[source: api-docs.deepseek.com/guides/kv_cache, retrieved 2026-04-19]

The automatic-caching design means long stable system prompts and few-shot scaffolds cost very little to re-send on repeat turns — an architectural win for multi-turn agentic use.

## 5. Multimodal Conventions

**DeepSeek-V3.2 is text-only.** The model card lists no image, video, or audio input support. For multimodal workloads, choose a different family.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

## 6. Behavioral Quirks

- **Reasoning content is send-back-forbidden.** Stripping it from prior-turn history before re-sending is mandatory. This is unique among reasoning models in this library — every other family either requires (Claude) or tolerates (OpenAI, Gemini) reasoning artifacts on input.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]

- **Sampling parameters are rejected on `deepseek-reasoner`.** `temperature`, `top_p`, `presence_penalty`, `frequency_penalty`, `logprobs`, `top_logprobs` are all unsupported. Set them only on `deepseek-chat`.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-04-19]
[testable: id=deepseek.reasoner-rejects-sampling.v1, expected=setting temperature=0.5 on a deepseek-reasoner request is rejected or silently ignored]

- **Special tokens use full-width characters.** `<｜` (full-width pipe) and `▁` (underscore character) — **not** ASCII `<|` and `_`. Hand-built chat strings that substitute ASCII look right in printed output but produce wrong tokenization.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **No Jinja chat template shipped.** The tokenizer config does not contain a `chat_template` field. Use DeepSeek's Python encoding helper; do not rely on HuggingFace `apply_chat_template` defaults.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **V3.2-Speciale has no tool calls.** The high-compute reasoning variant is pure reasoning. Any caller with a `tools` array will not get tool calls back from Speciale.
[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]

- **The `developer` role is narrow.** On V3.2 it is scoped to search agent scenarios, not a general precedence-raising role as on OpenAI. Misusing it in other contexts is undocumented territory.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **MIT license.** No 700M MAU clause, no regional restriction, no attribution requirement beyond MIT's standard clause. Materially more permissive than Llama 4's Community License.

## 7. Anti-Patterns

- **Do not include `reasoning_content` in the message history on a follow-up request.** It returns 400. Strip it; keep only `content`.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-04-19]

- **Do not set `temperature`, `top_p`, etc. on `deepseek-reasoner`.** They are rejected. Only set them on `deepseek-chat`.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-04-19]

- **Do not pass `tools` expecting function calls on `deepseek-v3.2-speciale`.** Speciale is reasoning-only.
[source: api-docs.deepseek.com/news/news251201, retrieved 2026-04-19]

- **Do not substitute ASCII `|` for the full-width `｜` in hand-built chat strings.** The tokens tokenize differently. Use DeepSeek's provided encoding script.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **Do not assume a Jinja template is present.** `deepseek-ai/DeepSeek-V3.2`'s tokenizer config deliberately omits it; callers using HF's default apply_chat_template will not work.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **Do not use `developer` role as a general precedence-raiser.** It is narrowly scoped to search-agent scenarios.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **Do not build for multimodal inputs.** DeepSeek-V3.2 is text-only.

- **Do not start new work on R1 or V3.1.** V3.2 subsumes them for reasoning (via `deepseek-reasoner`) and general use (via `deepseek-chat`).

## 8. Gaps

- **Exact context-window size** for V3.2 is not quoted in retrieved primary excerpts (the reasoner's 64K `max_tokens` is documented but total context is not).
- **Active parameter count** for the MoE architecture is not in the retrieved HF card excerpt (685B total confirmed).
- **V3.2-Speciale availability** was flagged with a December 15, 2025 temporary-endpoint expiration in the release notes; current availability status requires re-verification.
- **V4 release timing** — unreleased at this retrieval; leaks suggested April 2026, but no confirmed date in primary sources.
- **vLLM / SGLang minimum versions** for V3.2 with the new thinking-with-tools chat template are not explicitly quoted in retrieved sources.
- **`Developer` role full behavior** (what exactly a search-agent scenario expects) is not covered in depth.
- **Streaming event types** for `reasoning_content` deltas vs `content` deltas on `deepseek-reasoner` are not quoted verbatim.
- **Parallel function-call defaults** (is `parallel_tool_calls: false` supported?) are not quoted.
