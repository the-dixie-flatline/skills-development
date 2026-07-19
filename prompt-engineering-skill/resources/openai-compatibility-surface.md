---
family: cross-family
scope: openai-compatibility
families: [grok, deepseek, gemini, qwen, mistral, vllm, llamacpp, minimax, glm, kimi, gpt-oss]
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
  - https://developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide
  - https://developers.openai.com/cookbook/articles/openai-harmony
maturity_note: |
  "OpenAI-compatible" at the wire level is not OpenAI-equivalent at the
  semantic level. An OpenAI-shaped request body deserializes against every
  provider here, but the reasoning field name, the multi-turn reasoning
  round-trip rule, the set of rejected parameters, and the cache-token field
  path all diverge per provider. Some divergences return HTTP 400; others
  misbehave silently. Field paths and version-gated behaviors below are
  volatile — re-verify against the cited primary sources before relying on
  any single field path. A 2026-07-18 pass added a `[field-observed]` (N=15)
  grammar-decoding cross-reference to the vLLM/llama.cpp rows; no Tier-1 claim
  changed, so the retrieval date is unchanged. A 2026-07-19 pass added MiniMax,
  GLM, and Kimi provider sections and split the Grok reasoning-control line by
  generation (grok-4.5 vs grok-4.3); a 2026-07-19 pass also added a self-hosted
  gpt-oss / gpt-oss-safeguard section (Harmony reasoning-effort divergence).
  A later 2026-07-19 live-test pass added `[field-observed]` router notes
  (OpenRouter refusal normalization; an OR-hosted gpt-oss effort-steering
  carve-out to the Harmony no-op; the MiniMax reasoning-omission 400
  non-reproduction). All of these additions carry their own dated sources, so
  the file-level `retrieved:` (which represents the 2026-06-01 sweep of the
  original seven providers) is unchanged.
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
| MiniMax | `https://api.minimax.io/v1` | inline in `content` (split via `reasoning_split` → `reasoning_details`) | `top_k`/`stop_sequences` ignored; `max_tokens` accepted | `usage.prompt_tokens_details.cached_tokens` |
| GLM (Z.ai) | see Gaps (native path not quoted) | `reasoning_content` (separate delta stream) | not documented | automatic prefix caching |
| Kimi (Moonshot) | `https://api.moonshot.ai/v1` | `reasoning_content` | five sampling params fixed, ERROR on deviation | not documented (see section) |
| gpt-oss (self-host) | server-defined (vLLM / Ollama / LM Studio) | server-dependent Harmony channels; effort is a system-message directive, NOT a request field | server-defined | server-defined |

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
- **Reasoning control (per generation):** grok-4.5 (flagship) — `reasoning_effort`
  ∈ {low, medium, high}, default `high`; `none` removed, reasoning cannot be
  disabled. grok-4.3 (prior gen, still live) — `reasoning_effort` ∈ {none, low,
  medium, high}, default `low`. Any row asserting a portable `none` for Grok is
  stale for the flagship.
  [source: https://docs.x.ai/developers/model-capabilities/text/reasoning, retrieved 2026-07-19]
- **Rejected / no-op params:** `presencePenalty` / `frequencyPenalty` / `stop`
  cannot be used with reasoning models and return an error. Because grok-4.5 is a
  reasoning model on EVERY request (reasoning cannot be disabled), penalties and
  `stop` are rejected unconditionally on grok-4.5; on grok-4.3 they are rejected
  only when `reasoning_effort` ≠ `none`. `logprobs` / `top_logprobs` are silently
  ignored on grok-4.20 and newer.
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
- **Grammar-decoding behavioral traps** (applies to `choice`/`regex`/`json`/`grammar`
  on this server and to llama.cpp grammars below): a grammar constrains *emitted*
  tokens but is not *read* by the model — allowed enum values must also appear in
  the prompt text — and a closed enum with no fallback member cannot fail safe, so
  an out-of-vocabulary input is forced into a confident near-miss. Add an explicit
  fallback member plus an open free-text companion field when the value space is
  not genuinely finite. Detail and sample size in `resources/qwen-prompt-api.md` §6.
  [field-observed, N=15]

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

## MiniMax (OpenAI-compat surface)

[source: https://platform.minimax.io/docs/guides/text-generation, retrieved 2026-07-19]
[source: https://platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]
[source: https://platform.minimax.io/docs/api-reference/text-prompt-caching, retrieved 2026-07-19]

- **Base URL:** `https://api.minimax.io/v1` (CN: `https://api.minimaxi.com/v1`);
  endpoint `POST /v1/chat/completions`. The vendor recommends the
  Anthropic-compatible `/anthropic` path over this one.
- **Reasoning field (response):** inline in `content` by default;
  `extra_body={"reasoning_split": True}` splits thinking into a
  `reasoning_details` field (first-party field, not an OpenAI standard field).
- **Reasoning control (differs by model, not param):** M3 is OFF-by-default —
  opt in via `thinking:{"type":"adaptive"}`; M2.x cannot disable thinking.
- **Rejected / no-op params:** `top_k` (a local-inference vLLM suggestion only)
  and `stop_sequences` are ignored. `max_tokens` is accepted (not deprecated).
- **Cache field:** `usage.prompt_tokens_details.cached_tokens` (OpenAI path);
  the Anthropic path reports `usage.cache_read_input_tokens` /
  `usage.cache_creation_input_tokens` instead.
- **Router divergence (Tier 2 — not reproduced):** A `[community-reported]` claim
  held that OpenRouter returns a hard 400 when reasoning content is missing on a
  round-trip. This did NOT reproduce: MiniMax-M3 via OpenRouter (DeepInfra),
  replaying an assistant turn with reasoning stripped, returned HTTP 200 (3/3),
  not 400 [field-observed, 2026-07-19]. The first-party MiniMax consequence of
  omitting reasoning is reasoning-chain degradation, not a 400; that native
  contrast remains native-only (not exercised through OpenRouter this pass).

---

## GLM (Z.ai / zai-org)

[source: https://docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]
[source: https://docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]
[source: https://docs.z.ai/guides/capabilities/thinking-mode, retrieved 2026-07-19]
[source: https://docs.z.ai/devpack/quick-start, retrieved 2026-07-19]

- **Reasoning field (response):** chain-of-thought surfaces as
  `reasoning_content`, a separate delta stream alongside `content` and
  `tool_calls[*].function.arguments`. The client concatenates each stream
  independently.
- **Reasoning control (family-scoped, not OpenAI-equivalent):** seven values
  {none, minimal, low, medium, high, xhigh, max} (`max` default), GLM-5.2-and-
  above only, shimmed onto two real tiers — none/minimal → skip thinking,
  low/medium → high, xhigh → max. The same value names carry different semantics
  than on an OpenAI reasoning model.
- **Streaming tool arguments require a second flag:** incremental tool-argument
  emission needs BOTH `stream=true` AND `tool_stream=true`; `stream` alone does
  not stream tool-call arguments.
- **Parallel tool calls:** confirmed documentation absence — not documented on
  the first-party API; the streaming contract describes single-accumulation
  `tool_calls[*]`.
- **Multi-turn reasoning round-trip:** `clear_thinking: false` preserves
  prior-turn thinking; its default differs by endpoint (cleared on the Coding
  Plan endpoint, preserved on the standard API). Preserved `reasoning_content`
  must be returned complete, unmodified, and in original order or performance and
  cache-hit rates degrade.
- **Separate Anthropic-compatible endpoint:** `https://api.z.ai/api/anthropic`
  accepts Anthropic-Messages requests (Claude Code drop-in via
  `ANTHROPIC_BASE_URL` / `ANTHROPIC_AUTH_TOKEN`), distinct from the OpenAI-shaped
  surface.
- **Gap:** the exact native OpenAI base-URL path is not quoted in the retrieved
  primary sources (checked 2026-07-19).

---

## Kimi (Moonshot AI, OpenAI-compat layer)

[source: https://platform.kimi.ai/docs/api/models-overview, retrieved 2026-07-19]
[source: https://platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]
[source: https://platform.kimi.ai/docs/guide/use-dynamic-tool-loading, retrieved 2026-07-19]
[source: https://platform.kimi.ai/docs/guide/response_format, retrieved 2026-07-19]
[source: https://platform.kimi.ai/docs/guide/claude-code-kimi, retrieved 2026-07-19]

Base URL `https://api.moonshot.ai/v1`, model `kimi-k3`. Wire-compatible with
OpenAI Chat Completions but diverges on:

- **Five sampling params are fixed and error on deviation** [applies-to: kimi-k3]:
  `temperature`=1.0, `top_p`=0.95, `n`=1, `presence_penalty`=0,
  `frequency_penalty`=0, marked "cannot be modified". Passing a non-default value
  returns an error; omit them. (Contrast DeepSeek, where penalties silently
  no-op — Kimi errors.)
- **Reasoning is always on;** field name is `reasoning_content` (same as
  DeepSeek/Qwen), controlled by top-level `reasoning_effort`, whose default and
  only accepted value is `max` as of 2026-07-19.
- **Multi-turn replay:** the complete assistant message (incl. `reasoning_content`
  + `tool_calls`) must be passed back as-is; omitting it risks errors/degradation
  (unconditional, unlike DeepSeek's tool-call-conditional 400 rule).
- **Dynamic tool loading** [applies-to: kimi-k3]: a `role:"system"` message
  carrying a `tools` array must NOT carry `content` (400 "cannot be used with
  content"); it is not server-persisted, and fails "tokenization failed" on
  non-K3 models.
- **Structured output:** `response_format {type:"json_schema", strict:true}`
  under MFJS; parse only `choices[0].message.content`.
- **Anthropic surface** (`https://api.moonshot.ai/anthropic`) uses model string
  `kimi-k3[1m]` and does not support `ENABLE_TOOL_SEARCH`.
- **Token cap:** `max_completion_tokens` default 131072, up to 1048576.

---

## gpt-oss / gpt-oss-safeguard (self-hosted, OpenAI-shaped wrappers)

[source: https://developers.openai.com/cookbook/articles/gpt-oss-safeguard-guide, retrieved 2026-07-19]
[source: https://developers.openai.com/cookbook/articles/openai-harmony, retrieved 2026-07-19]

OpenAI's open-weight line (base gpt-oss 20b/120b + the gpt-oss-safeguard classifier
fine-tunes) is commonly served through OpenAI-shaped Chat-Completions wrappers
(vLLM, Ollama, LM Studio). It is wire-compatible at the request-body level but has
one divergence worth calling out here; the full family contract (Harmony format,
policy-as-prompt discipline) lives in `resources/gpt-oss-prompt-api.md`.

- **Base URL:** server-defined (whatever host you run — vLLM, Ollama, or LM Studio).
- **Reasoning control is NOT a top-level field (raw passthrough).** Reasoning effort is a
  Harmony system-message directive (`Reasoning: low|medium|high`, default medium), not a
  top-level `reasoning.effort` / `reasoning_effort` request parameter. On a raw passthrough
  server an OpenAI-style client that sets a top-level effort param will not steer the model —
  the setting has to be rendered into the Harmony system message, which the serving stack
  handles when it applies the model's chat template.
  - **OpenRouter-hosted carve-out:** top-level `reasoning.effort` DOES steer OR-hosted
    `gpt-oss-120b` — `effort=high` vs `effort=low` gave mean 811 vs 112 reasoning tokens
    (7.24×, non-overlapping 95% CIs, N=20/arm, provider-pinned DeepInfra). The OpenRouter
    provider wrapper translates the effort field into the Harmony directive server-side, so
    the no-op holds only for the raw-passthrough case, not this route. May not generalize
    across gpt-oss deployments (single pinned provider).
    [field-observed, N=20/arm, 2026-07-19, via OpenRouter (DeepInfra)]
- **Harmony rendering is mandatory.** The model requires Harmony-format rendering to
  work correctly; a raw non-Harmony prompt misbehaves. The OpenAI-shaped wrapper is a
  transport convenience over a model whose native contract is Harmony, not the GPT-5.x
  Responses API.
- **Reasoning field (response):** server-dependent — the three Harmony channels
  (analysis / commentary / final) are surfaced however the specific server exposes
  parsed reasoning; there is no single standardized field across vLLM / Ollama /
  LM Studio.

---

## OpenRouter response normalization (field-observed)

OpenRouter rewrites some upstream response fields into its OpenAI-shaped envelope, so the
finish/stop semantics a caller sees are the router's, not always the upstream provider's.
Observed for an Anthropic-family model routed through OpenRouter: an Anthropic
`stop_reason: "refusal"` is normalized to `finish_reason: "content_filter"` with
`native_finish_reason: "refusal"` and a populated `message.refusal` field; refusal
completions bill $0. Code that branches on `finish_reason` should treat `content_filter`
plus a `native_finish_reason: "refusal"` / `message.refusal` as the refusal signal on this
route.
[field-observed, N=105, 2026-07-19]

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
