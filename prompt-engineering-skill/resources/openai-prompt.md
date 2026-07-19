---
family: openai
scope: prompt
versions:
  - gpt-5.6-sol
  - gpt-5.6-terra
  - gpt-5.6-luna
  - gpt-5.5
  - gpt-5.5-pro
  - gpt-5.4
  - gpt-5.4-mini
  - gpt-5.4-nano
  - gpt-5.3-codex
retrieved: 2026-07-19
primary_sources:
  - https://developers.openai.com/api/docs/models/all
  - https://developers.openai.com/api/docs/models/gpt-5.6-sol
  - https://developers.openai.com/api/docs/models/gpt-5.6-terra
  - https://developers.openai.com/api/docs/models/gpt-5.6-luna
  - https://developers.openai.com/api/docs/models/gpt-5.5
  - https://developers.openai.com/api/docs/models/gpt-5.5-pro
  - https://developers.openai.com/api/docs/models/gpt-5.3-codex
  - https://developers.openai.com/api/docs/guides/latest-model
  - https://developers.openai.com/api/docs/guides/reasoning
  - https://developers.openai.com/api/docs/guides/migrate-to-responses
  - https://developers.openai.com/api/docs/guides/prompt-caching
  - https://developers.openai.com/api/docs/guides/structured-outputs
  - https://developers.openai.com/api/docs/guides/deep-research
  - https://developers.openai.com/api/docs/deprecations
  - https://developers.openai.com/api/docs/changelog
maturity_note: |
  GPT-5.6 (Sol / Terra / Luna) is the current flagship generation. The `gpt-5.6`
  alias routes to GPT-5.6 Sol; Terra and Luna have no generic alias and must be
  addressed by their full IDs. GPT-5.5 / gpt-5.5-pro and the GPT-5.4 line remain
  Active as prior-generation tiers. GPT-5.6 adds `max` reasoning effort, a per-request
  `reasoning.mode` (standard | pro), a `reasoning.context` control, an assistant-message
  `phase` field, Programmatic Tool Calling, explicit prompt-caching controls, and a
  Multi-agent orchestration beta on the Responses API. Lifecycle dates now in play:
  the o3-deep-research / o4-mini-deep-research / gpt-5.2-codex model IDs and several
  legacy snapshots shut down 2026-07-23 (imminent); a second snapshot wave (including
  o3-2025-04-16) shuts down 2026-12-11; the Assistants API is removed 2026-08-26. The
  effort-scale enum conflicts across two live vendor pages (see §1). OpenAI's
  open-weight line (gpt-oss, gpt-oss-safeguard) is covered separately in
  `resources/gpt-oss-prompt.md`.
---

# OpenAI — Prompt-Layer Reference

Portable prompting guidance for the current GPT-5.x generation. API-layer detail (endpoints, parameter shapes, beta features) lives in `openai-prompt-api.md`.

## 1. Model Selection

Pick by task axis. "Reasoning" models are integrated into the GPT-5 family; the o-series is no longer a separate reasoning line for new work (o-series snapshots remain available for now but are being retired — see §7 deprecations, and note `o3-2025-04-16` shuts down 2026-12-11).

The GPT-5.6 generation is exactly three tiers: `gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`.
[source: developers.openai.com/api/docs/models/all, retrieved 2026-07-19]

| Target task                                          | Preferred model                       | Notes                                                                                                                     |
|------------------------------------------------------|---------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| Current flagship, hardest reasoning + coding         | `gpt-5.6-sol` (alias `gpt-5.6`)       | $5 in / $30 out per MTok ($0.50 cached in); 1,050,000 context; 128K max output; knowledge cutoff Feb 16 2026            |
| Balanced quality + cost, current gen                 | `gpt-5.6-terra`                       | $2.50 in / $15 out ($0.25 cached in); 1,050,000 context; 128K max output; cutoff Feb 16 2026; no generic alias          |
| Cost/latency-optimized, current gen                  | `gpt-5.6-luna`                        | $1 in / $6 out ($0.10 cached in); 1,050,000 context; 128K max output; cutoff Feb 16 2026; no generic alias              |
| Prior-gen flagship (still Active)                    | `gpt-5.5` (`gpt-5.5-2026-04-23`)      | $5 in / $30 out ($0.50 cached in); 1,050,000 context; 128K max output; cutoff Dec 1 2025                                |
| Prior-gen highest-compute (still Active)             | `gpt-5.5-pro` (`gpt-5.5-pro-2026-04-23`) | "Uses more compute to think harder"; $30 in / $180 out, no cached discount; 1,050,000 context; streaming NOT supported |
| Prior-gen cheaper coding / professional tier         | `gpt-5.4`                             | $2.50 in / $15 out; 1,050,000 context; knowledge cutoff Aug 31 2025                                                      |
| Prior-gen balanced                                   | `gpt-5.4-mini`                        | $0.75 in / $4.50 out; 400K context                                                                                       |
| Prior-gen speed/cost                                 | `gpt-5.4-nano`                        | Low-latency variant                                                                                                      |
| Autonomous coding agents                             | `gpt-5.3-codex`                       | Dedicated agentic-coding model; $1.75 in / $14 out; 400K context; cutoff Aug 31 2025; reasoning efforts low/medium/high/xhigh |

[source: developers.openai.com/api/docs/models/all, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/models/gpt-5.6-sol, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/models/gpt-5.6-terra, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/models/gpt-5.6-luna, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.5-pro, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.3-codex, retrieved 2026-06-01]

Notes on selection:

- **`gpt-5.6` routes to Sol.** "The `gpt-5.6` alias routes requests to GPT-5.6 Sol." Terra and Luna have no generic alias — a confirmed absence; address them by full ID.
  [source: developers.openai.com/api/docs/models/gpt-5.6-sol, retrieved 2026-07-19]
- **Context / max output / cutoff are uniform across all three 5.6 tiers**: 1,050,000 context, 128,000 max output, knowledge cutoff Feb 16 2026. The tiers differ on price and capability, not on window.
  [source: developers.openai.com/api/docs/models/gpt-5.6-sol, retrieved 2026-07-19]
  [source: developers.openai.com/api/docs/models/gpt-5.6-terra, retrieved 2026-07-19]
  [source: developers.openai.com/api/docs/models/gpt-5.6-luna, retrieved 2026-07-19]
- **Pro capability is now available two ways on 5.6.** The prior generation exposed it only as a distinct model ID (`gpt-5.5-pro`). On the GPT-5.6 base models it is additionally available per-request via `reasoning.mode: "pro"`, billed at the base model's standard token rates (no multiplier). The existing `-pro` model IDs are NOT deprecated by this — they keep their current behavior and pricing. See `openai-prompt-api.md` §4.
  [source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]
- **A `GPT-5.4 Pro` tier is registry-listed** on `/models/all` but is not otherwise spec'd in this pass (price/context/effort not retrieved). Treat as existing-but-undetailed.
  [source: developers.openai.com/api/docs/models/all, retrieved 2026-07-19]

### Reasoning effort per model

This is the single most important capability to check when migrating prompts:

| Model               | Supported effort levels                            | Default        |
|---------------------|----------------------------------------------------|----------------|
| `gpt-5.6-*`         | `none`, `low`, `medium`, `high`, `xhigh`, `max`    | `medium`       |
| `gpt-5.5`           | `none`, `low`, `medium`, `high`, `xhigh`           | `medium`       |
| `gpt-5.3-codex`     | `low`, `medium`, `high`, `xhigh`                   | model-specific |

[source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.3-codex, retrieved 2026-06-01]

- **GPT-5.6 adds `max` above `xhigh`** and defaults to `medium` "in both standard and pro modes."
  [source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]

- **[disputed: the effort enum differs across two live vendor pages]** The model-specific `guides/latest-model` page lists `none, low, medium, high, xhigh, max` (has `max`, omits `minimal`). The general `guides/reasoning` page lists `none, minimal, low, medium, high, xhigh` (has `minimal`, omits `max`). For GPT-5.6 specifically, author against the model-specific scale (`none…xhigh, max`, default `medium`); `minimal` appears only on the general reasoning guide and is likely a legacy/older-model value; `max` appears only on the model-specific page. Do not assume a single enum covers every model.
  [source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]
  [source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

Default-change history note: across the GPT-5 series the implicit `reasoning.effort` default shifted between versions (GPT-5 defaulted to `medium`; the 5.1-5.4 line moved to `none`; 5.5 and 5.6 document `medium`). Per-model defaults for 5.4 / 5.4-mini / 5.4-nano were not re-verified. Set `reasoning.effort` explicitly on migration rather than relying on an implicit default; an unexpected default is the most common cause of quality regressions on migration.

## 2. Prompt Structure Conventions

The Responses API replaces `messages` (Chat Completions) with `input`, and `role: system` content with a top-level `instructions` field. Portable guidance:

- **Use `instructions` for persistent system-level context** (tone, role, constraints that persist across turns).
- **Use role-tagged items in `input`** for the turn sequence. Valid roles: `user`, `assistant`, `system`, `developer`.
- **The `developer` role** carries higher precedence than `user` role content. Use it for instructions that must survive user prompt-injection attempts ("ignore previous instructions" etc.).
[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]

Content types inside role items:

- `input_text` — text.
- `input_image` — image (`image_url` or `file_id`, with `detail` setting).
- `input_file` — uploaded document (`file_id` / `file_url` / inline `file_data`).

[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]

### Assistant-message `phase` field (GPT-5.4 / GPT-5.5, and 5.6)

Assistant messages in the Responses API carry a `phase` field taking `commentary` or `final_answer`. It separates interim narration from the answer proper in tool-heavy flows. It is **assistant-only** — "Don't add `phase` to user messages." Documentation enumerates support for `gpt-5.4` and `gpt-5.5`; support on codex snapshots and "subsequent mainline" models is not enumerated on the reasoning guide (treat as unverified beyond 5.4/5.5). When you replay conversation history manually, preserve `phase` on the assistant turns you carry forward.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

### Stateful vs stateless conversations

Two supported patterns:

- **Stateless**: pass the full turn history in `input` on every request. Callers manage state.
- **Stateful**: set `store: true` and/or `previous_response_id` / `conversation.id`. The server retains items between calls, and you reference a prior response when continuing.

[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

Stateful is the path of least resistance for reasoning models because reasoning items must be preserved across tool-call boundaries — the server does that automatically when `store: true` is in play.

## 3. Instruction Patterns

### Do not over-prompt reasoning models

Reasoning models already "think" internally. Adding "think step by step," "reason carefully," or elaborate planning scaffolding tends to be redundant on GPT-5.x reasoning-enabled effort levels and can hurt quality by confusing the model's own planning. Rely on `reasoning.effort` rather than prompt-based reasoning encouragement.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

### Structured outputs instead of "respond in JSON"

Use `text.format` with `strict: true` to constrain the output to a JSON schema. Instructing the model to "respond only in JSON" is less reliable and does not protect against refusals, truncation, or schema drift. Structured Outputs emits a `refusal` content block when the request is refused, making safety refusals programmatically detectable.
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

### Reasoning summaries

Setting `reasoning.summary: "auto"` returns a summary of the model's internal reasoning alongside the final output — useful for UX that wants to show the user what the model is considering. `"concise"` (supported by some computer-use models) and `"detailed"` (supported on o4-mini) are the other values. Summaries are opt-in. Note: `reasoning.summary` is not supported when the Multi-agent orchestration beta is enabled (see `openai-prompt-api.md` §8).
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

### Precedence within `input`

When both a `system`/`developer` message and a `user` message are present in `input`, the `system` / `developer` content takes precedence. Use `developer` role for constraints that must survive user override attempts; use `user` for the task itself.
[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]
[testable: id=openai.developer-role-precedence.v1, expected=a developer-role instruction overrides a directly conflicting user instruction] Verified 2026-07-19: this is a real role hierarchy, not best-effort. A `developer`-role instruction beat a directly conflicting `system`-role instruction 10/10 in BOTH directions — `developer` is distinct from `system` and outranks it (native OpenAI Chat Completions) — and beat a directly conflicting `user` instruction 20/20 (via OpenRouter→OpenAI, gpt-5.6-terra). Treat developer-role constraints as strong-precedence over both system and user content, suitable for instructions that must resist prompt injection.

## 4. Context Window Practical Guidance

- **GPT-5.6 (Sol / Terra / Luna)**: 1,050,000 token context window; 128,000 token max output; knowledge cutoff 2026-02-16 — uniform across all three tiers.
  [source: developers.openai.com/api/docs/models/gpt-5.6-sol, retrieved 2026-07-19]
  [source: developers.openai.com/api/docs/models/gpt-5.6-terra, retrieved 2026-07-19]
  [source: developers.openai.com/api/docs/models/gpt-5.6-luna, retrieved 2026-07-19]
- **GPT-5.5**: 1,050,000 token context window; 128,000 token max output; knowledge cutoff 2025-12-01.
  [source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
- **GPT-5.4**: 1,050,000 token context window; knowledge cutoff 2025-08-31. `gpt-5.4-mini`: 400K context.
  [source: developers.openai.com/api/docs/models/all, retrieved 2026-06-01]
- **Long-input surcharge**: prompts exceeding **272K tokens** incur **2×** input pricing and **1.5×** output pricing on GPT-5.4. Below 272K, standard pricing applies. Not re-verified for the 5.6 tiers in this pass.
  [source: developers.openai.com/api/docs/models/gpt-5.4, retrieved 2026-04-18]

Practical implications:

- Stay under 272K tokens where a surcharge applies. Between 272K and 1M, factor the premium into cost estimates.
- Prompt caching covers a large fraction of input-token cost on repeated prefixes (see `openai-prompt-api.md` §7). Structure prompts to put stable content first. Note: on GPT-5.6 and later, cache writes now cost 1.25× the uncached input rate and `prompt_cache_key` is required for the reliable matching path — a break from prior-gen behavior (detail in `openai-prompt-api.md` §7).
  [source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-07-19]
- Native compaction support and server-side `context_management.compact_threshold` can offload compaction from the caller.

[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

## 5. Multimodal Conventions

Current flagship input modalities:

- **Text** — always.
- **Images** — via `input_image` content parts with `image_url` or uploaded `file_id`. A `detail` parameter tunes quality vs token cost ("low" / "high" / "auto"); caching requires the detail parameter to match across requests. The GPT-5.6 changelog launch entry states 5.6 accepts images at their original resolution; the exact resolution/token semantics were not retrieved verbatim this pass.
  [source: developers.openai.com/api/docs/changelog, retrieved 2026-07-19]
- **Files / documents** — via `input_file` content parts (uploaded document via Files API).
- **Audio** — supported on the Realtime / audio line, not on the GPT-5.x text flagships directly.

GPT-5.6 multimodal specifics beyond the original-resolution note were not re-verified against the per-model pages in this pass.

[source: developers.openai.com/api/docs/models/gpt-5.4, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

## 6. Behavioral Quirks

- **Reasoning effort defaults drifted between model versions.** GPT-5 defaulted to `medium`; the 5.1-5.4 line moved to `none`; `gpt-5.5` and `gpt-5.6` document `medium` as default. Quality regressions on migration almost always trace to an unexpected effort default. Set `reasoning.effort` explicitly.
[source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]

- **The effort enum is not uniform and the vendor docs conflict** (see §1). `max` exists on 5.6 (model-specific page); `minimal` appears only on the general reasoning guide. Check the specific model page, not a generic enum.
[source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

- **Pro mode is additive on 5.6, not a replacement for `-pro` IDs.** `reasoning.mode: "pro"` on a 5.6 base model bills at standard token rates with no multiplier; the separate `-pro` model IDs (e.g. `gpt-5.5-pro`) are unchanged and not deprecated.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

- **Reasoning tokens are billed as output tokens** and are discarded from context after generating the visible response (but counted in `output_tokens_details.reasoning_tokens`).
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

- **Reasoning items must be preserved across tool-call turns.** If a turn includes a function call, you must include the reasoning items (via `previous_response_id` or explicit inclusion in `input`) on the follow-up request. Dropping them degrades multi-step reasoning quality.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]
[testable: id=openai.reasoning-items-preserved.v1, expected=multi-step reasoning quality degrades when reasoning items are dropped from the input on a function-call follow-up]

- **Structured-output refusal handling differs from plain-text.** When a request would be refused, the response is a `refusal` content block rather than schema-conforming JSON. Callers parsing `output_parsed` must handle the refusal case explicitly — a missing `output_parsed` is not necessarily a schema violation.
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

- **Tool calling with `reasoning: none` is not supported on Chat Completions for GPT-5.4.** This combination requires the Responses API — a breaking change compared to GPT-5. Not re-verified for 5.5/5.6.
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

- **Prompt cache routing is prefix-hash-based on prior-gen models.** Requests with the same initial ~256-token prefix route to the same cache. On GPT-5.6 and later, `prompt_cache_key` is required for the improved matching path and explicit breakpoints are available (see `openai-prompt-api.md` §7).
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-07-19]

## 7. Anti-Patterns

- **Do not start new work on Chat Completions.** It remains supported but is not recommended for new work. Use the Responses API.
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

- **Do not rely on an implicit reasoning default across versions.** The default differs between models. Set `reasoning.effort` explicitly on migration.
[source: developers.openai.com/api/docs/guides/latest-model, retrieved 2026-07-19]

- **Do not assume `reasoning.mode: "pro"` deprecates the `-pro` model IDs.** They coexist; `-pro` IDs keep their behavior and pricing.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

- **Do not use "think step by step" / "reason carefully"** on reasoning-enabled effort levels. Use `reasoning.effort`.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

- **Do not add `phase` to user messages.** It is assistant-only; a `phase` on a user message is a misuse.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]

- **Do not drop reasoning items between turns** when tool use is involved. Use `previous_response_id` or include them explicitly.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

- **Do not use `type: "json_object"` (legacy JSON mode) on new work.** It provides no schema adherence. Use `text.format` with `type: "json_schema"` and `strict: true`.
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

- **Do not build on the Assistants API** — it was deprecated 2025-08-26 and is removed **2026-08-26**. The migration target is the **Responses API plus the Conversations API** (the named state-management replacement for Assistants threads), not the Responses API alone.
[source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]

- **Do not invoke `o3-deep-research` / `o4-mini-deep-research` / `gpt-5.2-codex` for new work.** They shut down **2026-07-23** (imminent as of retrieval). See §8 and `openai-prompt-api.md` §8/§9.
[source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]

## 8. Gaps

- **Deep Research and the hosted agent surface.** OpenAI Deep Research and the hosted agent surface are documented in `resources/deep-research-agents.md` and `resources/agent-orchestration-surfaces.md`. Confirmed doc/model mismatch to watch: `guides/deep-research` still documents the retiring `o3-deep-research` / `o4-mini-deep-research` model IDs (shutdown 2026-07-23, replacement `gpt-5.5-pro`) and does not yet mention the successor invocation (`return_token_budget` on the Responses web-search tool, changelog-only). Address in `openai-prompt-api.md` §8.
  [source: developers.openai.com/api/docs/guides/deep-research, retrieved 2026-07-19]
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]
  [source: developers.openai.com/api/docs/changelog, retrieved 2026-07-19]
- **Multi-agent orchestration is now a hosted beta.** A hosted multi-agent/handoff surface exists in beta on the Responses API for GPT-5.6 models (the prior "no hosted handoff endpoint" statement is retracted). Detail in `openai-prompt-api.md` §8 and `resources/agent-orchestration-surfaces.md`.
  [source: developers.openai.com/api/docs/guides/responses-multi-agent, retrieved 2026-07-19]
- **`phase` model scope beyond 5.4/5.5** — codex snapshots and "subsequent mainline" support are not enumerated on the reasoning guide; author conservatively.
  [source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-07-19]
- **`reasoning.mode` / `reasoning.context` / caching detail** are API-layer; see `openai-prompt-api.md` §4 and §7.
- **Realtime API prompting patterns** (bidirectional audio, VAD, interruption handling) are not covered here. See the Realtime API docs.
- **Codex-specific prompting conventions** for `gpt-5.3-codex` beyond reasoning effort are not covered.
- **Per-model reasoning defaults for `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.4-nano`, and the `GPT-5.4 Pro` tier** were not re-verified in the 2026-07-19 pass.
- **GPT-5.6 multimodal, long-input surcharge, and per-tier effort behavior** were not re-verified against the per-model pages beyond the fields in §1 and §4.
- **OpenAI's open-weight line (gpt-oss, gpt-oss-safeguard)** is covered separately in `resources/gpt-oss-prompt.md` / `resources/gpt-oss-prompt-api.md`.
