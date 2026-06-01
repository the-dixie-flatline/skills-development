---
family: deepseek
scope: prompt
versions:
  - deepseek-v4-flash
  - deepseek-v4-pro
  - deepseek-ai/DeepSeek-V3.2
  - deepseek-ai/DeepSeek-V3.2-Speciale
retrieved: 2026-06-01
primary_sources:
  - https://api-docs.deepseek.com/news/news260424
  - https://api-docs.deepseek.com/quick_start/pricing
  - https://api-docs.deepseek.com/updates
  - https://api-docs.deepseek.com/guides/thinking_mode
  - https://api-docs.deepseek.com/guides/reasoning_model
  - https://api-docs.deepseek.com/guides/function_calling
  - https://api-docs.deepseek.com/guides/kv_cache
  - https://api-docs.deepseek.com/api/create-chat-completion
  - https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.2
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.2-Speciale
maturity_note: |
  DeepSeek V4 is the current production generation, exposed as two model IDs:
  `deepseek-v4-flash` and `deepseek-v4-pro`. Open weights are MIT-licensed,
  context length is 1M tokens (default across all DeepSeek services), and the
  API is available over both OpenAI ChatCompletions and Anthropic interfaces.
  Legacy API model names `deepseek-chat` and `deepseek-reasoner` remain as
  mappings onto V4-Flash (non-thinking and thinking respectively) but hard-retire
  and become inaccessible after Jul 24th, 2026, 15:59 UTC. The prior generation
  DeepSeek-V3.2 remains available as MIT open weights. Models are text-only
  (no multimodal support).
---

# DeepSeek — Prompt-Layer Reference

Portable prompting guidance for the current DeepSeek generation. API-layer detail (chat template tokens, API shapes, caching mechanics) lives in `deepseek-prompt-api.md`.

## 1. Model Selection

Current generation is DeepSeek V4, exposed as two model IDs.

| API Model ID        | Mode                            | Notes                                                              |
|---------------------|---------------------------------|--------------------------------------------------------------------|
| `deepseek-v4-flash` | Tri-state reasoning             | 284B total / 13B active params; legacy `deepseek-chat`/`deepseek-reasoner` map here |
| `deepseek-v4-pro`   | Tri-state reasoning             | 1.6T total / 49B active params; flagship                            |

Both expose a tri-state reasoning ladder (Non-think / Think (High) / Think Max). Context length is 1M tokens; max output is 384K.
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-06-01]
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

### Legacy API model IDs (hard-retire 2026-07-24)

`deepseek-chat` and `deepseek-reasoner` still resolve but are hard-retired and inaccessible after **Jul 24th, 2026, 15:59 UTC**. Until then they map to the non-thinking and thinking modes of `deepseek-v4-flash` respectively. Migrate to the explicit `deepseek-v4-flash` / `deepseek-v4-pro` IDs plus a reasoning toggle.
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-06-01]
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]
[source: api-docs.deepseek.com/updates, retrieved 2026-06-01]

### Open weights

- **`deepseek-ai/DeepSeek-V4-Pro`** — HF, MIT license, 1.6T total / 49B active params.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]
- **`deepseek-ai/DeepSeek-V3.2`** — HF, MIT license, 685B total parameters, DeepSeek Sparse Attention (DSA) architecture. Prior generation.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### Legacy / predecessors

- DeepSeek-V3.2-Speciale (`deepseek-ai/DeepSeek-V3.2-Speciale`) — weights-only deep-reasoning variant, no tool-calling. Its temporary API endpoint expired 2025-12-15, 15:59 UTC; weights remain available.
[source: api-docs.deepseek.com/updates, retrieved 2026-06-01]
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2-Speciale, retrieved 2026-06-01]
- DeepSeek-V3.1 (`deepseek-ai/DeepSeek-V3.1`) — earlier generation.
- DeepSeek-R1 family — pure reasoning model from January 2025, superseded by integrated thinking.

### Selection rules

- **General use** — `deepseek-v4-flash` with reasoning disabled.
- **Reasoning workloads** — `deepseek-v4-flash` or `deepseek-v4-pro` with thinking enabled. Thinking is internal; caller receives separated `reasoning_content` + `content` fields.
- **Hardest reasoning** — `deepseek-v4-pro` at Think Max effort.
- **Self-hosted** — V4-Pro open weights (MIT) — most permissive license of any frontier-scale model in this reference library.
- **Multimodal use** — DeepSeek is **text-only** on current versions. Not the right family.

## 2. Prompt Structure Conventions

The API layer is exposed over both OpenAI ChatCompletions and Anthropic interfaces. Prompts portable from OpenAI Chat Completions generally work on `deepseek-v4-flash` / `deepseek-v4-pro` with a `base_url` swap.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]

At the open-weights level, DeepSeek uses a distinctive special-token chat template — visually different from most other open-weights families. Full-width pipe characters (`｜`) and underscore characters (`▁`) appear in control tokens. Exact tokens in `deepseek-prompt-api.md`.

### Roles

- `user` / `<｜User｜>` at tokenizer level.
- `assistant` / `<｜Assistant｜>` at tokenizer level.
- `developer` / `<｜Developer｜>` — [applies-to: deepseek-ai/DeepSeek-V3.2] **used exclusively for search agent scenarios** on V3.2. Not a general-purpose role (unlike OpenAI's `developer` role).

[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### No Jinja template in the tokenizer config

The DeepSeek V4 release ships **no Jinja-format chat template**. It provides an `encoding` folder with Python `encode_messages(...)` and `parse_message_from_completion_text(...)` helpers; use those rather than relying on HuggingFace's `apply_chat_template` defaults. Recommended local sampling for the open weights is temperature=1.0, top_p=1.0.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

[applies-to: deepseek-ai/DeepSeek-V3.2] DeepSeek-V3.2 likewise ships no Jinja template; its model card directs callers to `encoding/encoding_dsv32.py` with `encode_messages()` and `parse_message_from_completion_text()`.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

## 3. Instruction Patterns

### Tri-state reasoning toggle

V4 exposes three reasoning levels: Non-think, Think (High), and Think Max. Toggle via either:

- `thinking: {"type": "enabled" | "disabled"}`, or
- `reasoning_effort: "high" | "max"`.

`reasoning_effort` values `low`/`medium` map to `high`; `xhigh` maps to `max`. Some complex agent harnesses (for example Claude Code and OpenCode) automatically set effort to `max`.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-06-01]

Think Max is the maximum reasoning-effort mode. For it, DeepSeek recommends setting the context window to at least 384K tokens (a recommendation, not a hard gate); it uses a special system prompt.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

### Thinking with tools

The model can reason, call a tool, reason again on the result, and continue — all within a single thinking-enabled turn.
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-06-01]

**Exception**: `deepseek-ai/DeepSeek-V3.2-Speciale` does **not support tool calls** at all (weights-only variant).
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2-Speciale, retrieved 2026-06-01]

### Reasoning content round-trip rule (V4)

`reasoning_content` is returned at the same level as `content`, in thinking mode only. On V4 the multi-turn round-trip rule depends on whether the turn performs tool calls:

- On turns that do **not** perform tool calls, passing `reasoning_content` back in message history is **ignored** (no error).
- On turns that **do** perform tool calls, `reasoning_content` **must** be passed back or the API returns **HTTP 400**.

This inverts the legacy `deepseek-reasoner` rule, where **any** `reasoning_content` in input returned 400. It is also unlike Claude's signature-authenticated `thinking` blocks and OpenAI's `previous_response_id` / explicit reasoning-item pattern. Portable-prompt pipelines must special-case DeepSeek.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-06-01]
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]
[testable: id=deepseek.reasoning-content-roundtrip-tool-turn.v2, expected=on V4 a tool-call turn omitting prior reasoning_content returns HTTP 400; a non-tool-call turn including it is accepted]

### Function calling as OpenAI-compatible

`tools` array with OpenAI-shape function definitions. `strict: true` enforces schema compliance (via the beta endpoint). DeepSeek supports richer schema features than OpenAI's subset — `$def`, `$ref`, recursive definitions — useful for structured data extraction with shared sub-schemas.
[source: api-docs.deepseek.com/guides/function_calling, retrieved 2026-04-19]

## 4. Context Window Practical Guidance

- Context length is **1M tokens** (the default across all DeepSeek services); max output is **384K**.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]
- `max_tokens` includes the chain-of-thought (CoT) portion. Budget output accordingly when thinking is enabled.
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]
- For Think Max, DeepSeek recommends a context window of at least 384K tokens.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]
- Prompt caching is **automatic on disk** — minimum cacheable 64 tokens; up to ~90% cost reduction on cache hit.
[source: api-docs.deepseek.com/guides/kv_cache, retrieved 2026-04-19]

The automatic-caching design means long stable system prompts and few-shot scaffolds cost very little to re-send on repeat turns — an architectural win for multi-turn agentic use.

## 5. Multimodal Conventions

**DeepSeek-V3.2 is text-only.** The model card lists no image, video, or audio input support. For multimodal workloads, choose a different family.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

## 6. Behavioral Quirks

- **Reasoning-content round-trip is conditional on V4.** On non-tool-call turns, passing `reasoning_content` back is ignored; on tool-call turns it must be passed back or the request returns 400. This inverts the legacy `deepseek-reasoner` rule (any input `reasoning_content` returned 400).
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]
[source: api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]

- **`frequency_penalty` and `presence_penalty` are hard-deprecated** (no effect).
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]

- **Special tokens use full-width characters.** `<｜` (full-width pipe) and `▁` (underscore character) — **not** ASCII `<|` and `_`. Hand-built chat strings that substitute ASCII look right in printed output but produce wrong tokenization.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **No Jinja chat template shipped.** The V4 release provides an `encoding` folder with Python `encode_messages(...)` / `parse_message_from_completion_text(...)` instead. Use those; do not rely on HuggingFace `apply_chat_template` defaults.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Pro, retrieved 2026-06-01]

- **V3.2-Speciale has no tool calls.** The weights-only deep-reasoning variant is pure reasoning. Any caller with a `tools` array will not get tool calls back from Speciale. Its temporary API endpoint expired 2025-12-15; weights remain available.
[source: api-docs.deepseek.com/updates, retrieved 2026-06-01]
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2-Speciale, retrieved 2026-06-01]

- **The `developer` role is narrow.** [applies-to: deepseek-ai/DeepSeek-V3.2] On V3.2 it is scoped to search agent scenarios, not a general precedence-raising role as on OpenAI. Misusing it in other contexts is undocumented territory.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **MIT license.** No 700M MAU clause, no regional restriction, no attribution requirement beyond MIT's standard clause. Materially more permissive than Llama 4's Community License.

## 7. Anti-Patterns

- **Do not drop `reasoning_content` on a tool-call turn (V4).** On turns that perform tool calls, omitting prior `reasoning_content` returns 400; on non-tool-call turns it is ignored. Do not assume the legacy "always strip" rule.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]

- **Do not rely on `frequency_penalty` / `presence_penalty`.** They are hard-deprecated and have no effect.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]

- **Do not start new work on the legacy `deepseek-chat` / `deepseek-reasoner` IDs.** They hard-retire after Jul 24th, 2026, 15:59 UTC. Use `deepseek-v4-flash` / `deepseek-v4-pro` with a reasoning toggle.
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-06-01]

- **Do not pass `tools` expecting function calls on `deepseek-ai/DeepSeek-V3.2-Speciale`.** Speciale is reasoning-only.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2-Speciale, retrieved 2026-06-01]

- **Do not substitute ASCII `|` for the full-width `｜` in hand-built chat strings.** The tokens tokenize differently. Use DeepSeek's provided encoding script.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **Do not assume a Jinja template is present.** `deepseek-ai/DeepSeek-V3.2`'s tokenizer config deliberately omits it; callers using HF's default apply_chat_template will not work.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **Do not use `developer` role as a general precedence-raiser.** It is narrowly scoped to search-agent scenarios.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **Do not build for multimodal inputs.** DeepSeek is text-only on current versions.

- **Do not start new work on R1 or V3.1.** The current generation subsumes them for both reasoning and general use.

## 8. Gaps

- **Knowledge cutoff dates** for any V3.x or V4 model are not documented in primary sources.
- **vLLM / SGLang minimum versions** for the V4 chat-encoding helper are not explicitly quoted in retrieved sources.
- **`Developer` role full behavior** (what exactly a search-agent scenario expects) is not covered in depth.
- **Streaming event types** for `reasoning_content` deltas vs `content` deltas in thinking mode are not quoted verbatim.
- **Parallel function-call defaults** (is `parallel_tool_calls: false` supported?) are not quoted.
