---
family: cross-family
scope: openai-compatibility
families: [grok, deepseek, gemini, qwen, mistral, vllm, llamacpp]
retrieved: 2026-06-01
primary_sources:
  - https://docs.x.ai/llms.txt
  - https://docs.x.ai/developers/rest-api-reference/inference/chat
  - https://docs.x.ai/developers/model-capabilities/text/reasoning
  - https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing
  - https://api-docs.deepseek.com/quick_start/pricing
  - https://api-docs.deepseek.com/api/create-chat-completion
  - https://api-docs.deepseek.com/guides/thinking_mode
  - https://api-docs.deepseek.com/guides/reasoning_model
  - https://ai.google.dev/gemini-api/docs/openai
  - https://ai.google.dev/gemini-api/docs/thought-signatures
  - https://www.alibabacloud.com/help/en/model-studio/qwen-api-via-openai-chat-completions
  - https://www.alibabacloud.com/help/en/model-studio/deep-thinking
  - https://www.alibabacloud.com/help/en/model-studio/compatibility-with-openai-responses-api
  - https://docs.mistral.ai/api/endpoint/chat
  - https://docs.mistral.ai/resources/migration-guides
  - https://docs.vllm.ai/en/latest/features/reasoning_outputs/
  - https://docs.vllm.ai/en/latest/features/automatic_prefix_caching/
  - https://docs.vllm.ai/en/latest/features/tool_calling/
  - https://docs.vllm.ai/en/latest/features/structured_outputs/
  - https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md
maturity_note: |
  "OpenAI-compatible" at the wire level is not OpenAI-equivalent at the
  semantic level. An OpenAI-shaped request body deserializes against every
  provider here, but the reasoning field name, the multi-turn reasoning
  round-trip rule, the set of rejected parameters, and the cache-token field
  path all diverge per provider. Some divergences return HTTP 400; others
  misbehave silently. Field paths and version-gated behaviors below are
  volatile — re-verify against the cited primary sources before relying on
  any single field path.
---

# OpenAI Compatibility Surface — Cross-Family Reference

This is the single source for the per-provider OpenAI-compatibility matrix. The
family `-api.md` files cross-reference this file rather than duplicating the
matrix. Each section is self-contained; no other reference file is required to
use it.

## Thesis

An OpenAI-shaped request is portable at the wire level: the same JSON body
(`model`, `messages`, `tools`, `stream`, `temperature`, etc.) is accepted by
every provider tabulated here when pointed at that provider's compatibility base
URL. Portability stops at four axes:

1. **Reasoning field name.** Where the model's chain-of-thought surfaces in the
   response is not standardized. It is `reasoning_content` on some providers,
   `reasoning` on vLLM, a `summary[]` reasoning item on Responses-style layers,
   and absent or differently-shaped on others.
2. **Multi-turn reasoning round-trip rule.** Whether you must (or must not) feed
   the prior turn's reasoning back into the next request differs per provider,
   and on DeepSeek it differs by whether the turn performed a tool call. Getting
   this wrong returns HTTP 400 on some paths and is silently ignored on others.
3. **Rejected / no-op parameters.** Penalty parameters, `stop`, `logprobs`, and
   the `max_tokens` vs `max_completion_tokens` rename are each handled
   differently — sometimes a hard error, sometimes silently dropped.
4. **Caching field paths.** The field reporting cached/prefix tokens (and the
   field enabling explicit caching) is provider-specific.

The per-provider sections below tabulate each axis. The closing "Gaps" section
lists behaviors that are commonly assumed but are NOT documented by the cited
primary sources; do not assert those.

## Quick matrix

The authoritative per-axis detail is in each provider section. This table is a
routing index only.

| Provider | Base URL (compat) | Reasoning field | Penalty / stop handling | Cache field |
|----------|-------------------|-----------------|-------------------------|-------------|
| xAI Grok | `https://api.x.ai/v1` | `message.reasoning_content` | penalties/`stop` error on reasoning models; `logprobs` silently ignored | `usage.prompt_tokens_details.cached_tokens` |
| DeepSeek | `https://api.deepseek.com` | `reasoning_content` | `frequency_penalty`/`presence_penalty` hard-deprecated (no effect) | `prompt_cache_hit_tokens` |
| Gemini (compat) | `https://generativelanguage.googleapis.com/v1beta/openai/` | none documented on this layer | not documented | `extra_body.cached_content` |
| Qwen / DashScope | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` | `reasoning_content` (Chat); `summary[]` item (Responses) | not documented | `prompt_tokens_details.cached_tokens` |
| Mistral | `https://api.mistral.ai/v1` | not documented here | n/a (see field-rename note) | `prompt_cache_key` |
| vLLM (self-host) | server-defined | `reasoning` (renamed from `reasoning_content`) | n/a | prefix cache (`enable_prefix_caching`) |
| llama.cpp (self-host) | server-defined | `message.reasoning_content` (mode-dependent) | n/a | `cache_prompt` |

---

## xAI Grok

[source: https://docs.x.ai/llms.txt, retrieved 2026-06-01]
[source: https://docs.x.ai/developers/rest-api-reference/inference/chat, retrieved 2026-06-01]
[source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-06-01]
[source: https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing, retrieved 2026-06-01]

- **Base URL:** `https://api.x.ai/v1`.
- **Reasoning field (response):** non-streaming Chat Completions →
  `message.reasoning_content`. Reasoning token count →
  `usage.completion_tokens_details.reasoning_tokens`. Streaming →
  `chunk.reasoning_content` (Chat Completions / xAI SDK) or the
  `response.reasoning_summary_text.delta` SSE event (Responses API). Encrypted
  full reasoning is opt-in via `include: ["reasoning.encrypted_content"]`
  (Responses API only).
- **Reasoning control:** `reasoning_effort` ∈ {none, low, medium, high},
  default `low`.
- **Rejected / no-op params:** `presencePenalty` / `frequencyPenalty` / `stop`
  cannot be used with reasoning models and return an error. `logprobs` /
  `top_logprobs` are silently ignored on grok-4.20 and newer.
- **Cache field:** `usage.prompt_tokens_details.cached_tokens` (Chat
  Completions) / `usage.input_tokens_details.cached_tokens` (Responses API).
- **Multi-turn reasoning round-trip:** not separately documented as a hard rule
  in the retrieved sources; see Gaps.

---

## DeepSeek

[source: https://api-docs.deepseek.com/quick_start/pricing, retrieved 2026-06-01]
[source: https://api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-06-01]
[source: https://api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-06-01]
[source: https://api-docs.deepseek.com/guides/reasoning_model, retrieved 2026-06-01]

- **Base URLs:** OpenAI format `https://api.deepseek.com`; Anthropic format
  `https://api.deepseek.com/anthropic`.
- **Reasoning field (response):** `reasoning_content`, returned at the same
  level as `content` (thinking mode only).
- **Multi-turn reasoning round-trip rule (V4):** on turns that do NOT perform
  tool calls, passing `reasoning_content` back is ignored (no error). On turns
  that DO perform tool calls, `reasoning_content` MUST be passed back or the API
  returns HTTP 400. (Legacy `deepseek-reasoner` behavior: ANY `reasoning_content`
  present in the input returned 400.) This is the sharpest divergence in the
  matrix — the same request that succeeds on a non-tool turn returns 400 on a
  tool turn if reasoning is dropped.
- **Rejected / no-op params:** `frequency_penalty` / `presence_penalty` are
  hard-deprecated (no effect).
- **Cache field:** `prompt_cache_hit_tokens`.
- **`max_tokens` note:** `max_tokens` includes the chain-of-thought portion, so
  budgeting must account for reasoning tokens against the same ceiling.

---

## Gemini (OpenAI-compat layer)

[source: https://ai.google.dev/gemini-api/docs/openai, retrieved 2026-06-01]
[source: https://ai.google.dev/gemini-api/docs/thought-signatures, retrieved 2026-06-01]

- **Base URL:** `https://generativelanguage.googleapis.com/v1beta/openai/`.
  Documented as beta ("still in beta while we extend feature support").
- **Reasoning field (response):** there is NO documented `reasoning_content`
  response field on this layer. Thought summaries are enabled via
  `extra_body.google.thinking_config.include_thoughts=true`; the response
  field/shape that carries those summaries is not specified in the source — see
  Gaps.
- **Reasoning control:** `reasoning_effort` ∈ {minimal, low, medium, high} plus
  `none` (the latter on 2.5 models only). Reasoning cannot be turned off for
  Gemini 2.5 Pro or any Gemini 3 model. `reasoning_effort` maps to
  thinking_level / thinking_budget.
- **Tool-call thought signatures:** travel in
  `extra_content.google.thought_signature` (request and response) and MUST be
  echoed back for Gemini 3 function calls.
- **Caching:** `extra_body.cached_content`.
- **Other compat fields:** `service_tier` ∈ {flex, priority, standard} (default
  standard). Structured output via `client.beta.chat.completions.parse`.

---

## Qwen / DashScope (OpenAI-compat)

[source: https://www.alibabacloud.com/help/en/model-studio/qwen-api-via-openai-chat-completions, retrieved 2026-06-01]
[source: https://www.alibabacloud.com/help/en/model-studio/deep-thinking, retrieved 2026-06-01]
[source: https://www.alibabacloud.com/help/en/model-studio/compatibility-with-openai-responses-api, retrieved 2026-06-01]

- **Base URL (regional):** `https://dashscope-intl.aliyuncs.com/compatible-mode/v1`.
  A US endpoint and a Beijing `dashscope` endpoint also exist.
- **Reasoning field — Chat Completions:** reasoning in `reasoning_content`
  (non-streaming and streaming); the answer is in `content`. `enable_thinking`
  is non-standard and must be passed via `extra_body`.
- **Reasoning field — OpenAI Responses-compatible layer:** `enable_thinking` is
  deprecating; use `reasoning.effort` (which takes precedence). Reasoning output
  on that layer is a reasoning item exposing `summary[]` — NOT
  `reasoning_content`. The reasoning field name therefore differs between the
  two Qwen-compatible layers.
- **Cache field:** OpenAI-standard `prompt_tokens_details.cached_tokens`;
  explicit context caching is supported.
- **Streaming:** `tool_stream` streams tool-call argument deltas; it takes
  effect only when `stream=true`.

---

## Mistral

[source: https://docs.mistral.ai/api/endpoint/chat, retrieved 2026-06-01]
[source: https://docs.mistral.ai/resources/migration-guides, retrieved 2026-06-01]

- **Base URL:** `https://api.mistral.ai/v1`.
- **Field-rename divergence:** the chat schema documents `max_tokens` only.
  There is NO `max_completion_tokens` field — Mistral did not adopt OpenAI's
  rename. Send `max_tokens`. (Whether `max_completion_tokens` is actively
  rejected or silently ignored is not documented; see Gaps.)
- **Reasoning control:** `reasoning_effort` accepts `"high"` or `"none"` only on
  the chat endpoint — NOT the OpenAI four-level scale. (Whether `low`/`medium`
  are accepted is not documented; see Gaps.)
- **Cache field:** `prompt_cache_key`; cached tokens are billed at 10% of the
  standard input price.

---

## vLLM (self-hosted OpenAI-compatible server)

[source: https://docs.vllm.ai/en/latest/features/reasoning_outputs/, retrieved 2026-06-01]
[source: https://docs.vllm.ai/en/latest/features/automatic_prefix_caching/, retrieved 2026-06-01]
[source: https://docs.vllm.ai/en/latest/features/tool_calling/, retrieved 2026-06-01]
[source: https://docs.vllm.ai/en/latest/features/structured_outputs/, retrieved 2026-06-01]

- **Reasoning field (response):** RENAMED from `reasoning_content` to
  `reasoning` ("directly replace reasoning_content with reasoning"). A reasoning
  parser is selected with `--reasoning-parser <name>`. Reasoning is exposed only
  on `/v1/chat/completions`, `/v1/messages`, and `/v1/responses`. Tool calls are
  parsed only from the `content` field, not from `reasoning`.
- **Reasoning control:** `reasoning_effort` low/medium/high → enable_thinking=true;
  `none` → false. The `thinking_token_budget` sampling param forces
  `reasoning_end_str` at the budget.
- **Prefix caching:** `enable_prefix_caching=True` (reduces prefill only, not
  decode).
- **Tool calling:** `--enable-auto-tool-choice` is mandatory, plus
  `--tool-call-parser <name>`. `tool_choice` ∈ {auto, required (vllm ≥ 0.8.3),
  none, named}. In `auto` mode, arguments are NOT schema-constrained. The
  `strict` field is accepted but is a no-op.
- **Structured outputs params:** `choice`, `regex`, `json`, `grammar`,
  `structural_tag`. BREAKING (v0.12.0): `guided_json` / `guided_regex` / etc.
  were REMOVED — use the nested `structured_outputs` field instead, and remove
  the `guided_decoding_backend` field. Backend flag is
  `--structured-outputs-config.backend` (default auto). Reasoning + structured
  outputs may need `--structured-outputs-config.enable_in_reasoning=True`
  (Qwen3 Coder, v0.11.2+).

---

## llama.cpp (self-hosted server)

[source: https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md, retrieved 2026-06-01]

- **Reasoning field (response):** mode-dependent via `--reasoning-format`:
  - `none` — thoughts left unparsed in `message.content`; raw generated text.
  - `deepseek-legacy` — keeps `<think>` tags in `message.content` while also
    populating `message.reasoning_content`.
  - `deepseek` — thoughts cleanly in `message.reasoning_content`.
  - default `auto`.
  Per-request override: `reasoning_format`. The server supports
  `reasoning_content` similar to the DeepSeek API.
- **Think-tag leak hazard:** with `--reasoning-format none` or
  `deepseek-legacy`, reasoning lands in `message.content` and will surface to
  end users unless stripped. Mitigation: `--reasoning-format deepseek` +
  `--jinja`.
- **Reasoning control:** `-rea` / `--reasoning` ∈ {on, off, auto} (default auto).
  `--reasoning-budget N` where -1 = unrestricted, 0 = immediate end, N>0 = budget
  (default -1).
- **Function calling:** `--jinja` is default-enabled and is required for
  OpenAI/Anthropic-style function calling.
- **Caching:** `cache_prompt` reuses KV cache from a previous request's common
  prefix; enabling it CAN cause nondeterministic results.
- **Multimodal:** projector via `-mm` / `--mmproj`.

---

## Gaps / do-not-assert

The following are NOT documented by the cited primary sources. Do not assert
them as fact; treat them as open until verified.

- **Gemini OpenAI-compat:** a `refusal:null`-causing 400, or a
  content-returned-as-array divergence — NOT documented. The image `detail`
  parameter — NOT documented on the OpenAI-compat page. The response field/shape
  that carries thought summaries when `include_thoughts=true` — NOT specified.
- **Qwen:** a `reasoning_details` field — NOT documented. A "tools forbidden
  while streaming" restriction — NOT documented.
- **Mistral:** whether `max_completion_tokens` is actively rejected versus
  silently ignored — NOT documented. Whether `reasoning_effort` accepts
  `low`/`medium` — NOT documented (the schema lists only `high`/`none`).
- **xAI Grok:** a hard multi-turn reasoning round-trip rule (must/must-not echo
  prior `reasoning_content`) — NOT documented in the retrieved sources.
- **llama.cpp:** a "reasoning tags split across SSE chunks" leak as a distinct
  named bug — NOT documented in the server README. The only documented leak
  vector is the `--reasoning-format` mode choice.
