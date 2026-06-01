---
family: openai
scope: prompt
versions:
  - gpt-5.5
  - gpt-5.5-pro
  - gpt-5.4
  - gpt-5.4-mini
  - gpt-5.4-nano
  - gpt-5.3-codex
retrieved: 2026-06-01
primary_sources:
  - https://developers.openai.com/api/docs/models
  - https://developers.openai.com/api/docs/models/gpt-5.5
  - https://developers.openai.com/api/docs/models/gpt-5.5-pro
  - https://developers.openai.com/api/docs/models/gpt-5.3-codex
  - https://developers.openai.com/api/docs/models/compare
  - https://developers.openai.com/api/docs/guides/reasoning
  - https://developers.openai.com/api/docs/guides/migrate-to-responses
  - https://developers.openai.com/api/docs/guides/prompt-caching
  - https://developers.openai.com/api/docs/guides/structured-outputs
  - https://developers.openai.com/api/docs/guides/deep-research
  - https://developers.openai.com/api/docs/guides/agents/orchestration
  - https://developers.openai.com/api/docs/deprecations
  - https://developers.openai.com/codex/changelog
maturity_note: |
  GPT-5.5 (snapshot gpt-5.5-2026-04-23) is the current flagship for complex
  reasoning and coding; gpt-5.5-pro is the highest-compute variant. GPT-5.4 is
  retained as the cheaper coding/professional tier, with -mini and -nano below
  it. The Responses API is the recommended primary endpoint; the Assistants API
  is removed 2026-08-26 and its replacement is the Responses API plus the
  Conversations API (state management). The o3-deep-research and
  o4-mini-deep-research model IDs, along with gpt-5.2-codex and earlier legacy
  codex IDs, shut down 2026-07-23. Reasoning defaults across the 5.x series have
  not been re-verified for 5.5 in this pass; do not rely on an implicit default,
  set `reasoning.effort` explicitly.
---

# OpenAI — Prompt-Layer Reference

Portable prompting guidance for the current GPT-5.x generation. API-layer detail (endpoints, parameter shapes, beta features) lives in `openai-prompt-api.md`.

## 1. Model Selection

Pick by task axis. "Reasoning" models are integrated into the GPT-5 family; the o-series is no longer a separate reasoning line for new work (o-series models remain available but the 5.x series supersedes them for most use cases).

| Target task                                               | Preferred model                 | Notes                                                                                     |
|-----------------------------------------------------------|---------------------------------|-------------------------------------------------------------------------------------------|
| Complex reasoning and coding, general flagship            | `gpt-5.5` (`gpt-5.5-2026-04-23`) | 1,050,000 context; 128K max output; $5 in / $30 out per MTok ($0.50 cached in); knowledge cutoff Dec 1 2025 |
| Highest-compute, hardest problems                         | `gpt-5.5-pro` (`gpt-5.5-pro-2026-04-23`) | "Uses more compute to think harder"; $30 in / $180 out, no cached discount; 1,050,000 context; Responses API incl. Batch; streaming NOT supported |
| Cheaper coding / professional tier                        | `gpt-5.4`                       | $2.50 in / $15 out; 1,050,000 context; knowledge cutoff Aug 31 2025                       |
| Balanced quality + cost                                   | `gpt-5.4-mini`                  | $0.75 in / $4.50 out; 400K context                                                       |
| Speed and cost optimization                               | `gpt-5.4-nano`                  | Low-latency variant                                                                      |
| Autonomous coding agents                                  | `gpt-5.3-codex`                 | Top dedicated agentic-coding model; $1.75 in / $14 out; 400K context; cutoff Aug 31 2025; reasoning efforts low/medium/high/xhigh. GPT-5.5 is also available in Codex |

[source: developers.openai.com/api/docs/models, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.5-pro, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.3-codex, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/compare, retrieved 2026-06-01]
[source: developers.openai.com/codex/changelog, retrieved 2026-06-01]

### Reasoning effort per model

This is the single most important capability to check when migrating prompts:

| Model               | Supported effort levels                     | Default     |
|---------------------|---------------------------------------------|-------------|
| `gpt-5.5`           | `none`, `low`, `medium`, `high`, `xhigh`    | `medium`    |
| `gpt-5.3-codex`     | `low`, `medium`, `high`, `xhigh`            | model-specific |

[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/models/gpt-5.3-codex, retrieved 2026-06-01]

`gpt-5.5` supports effort levels `none`, `low`, `medium` (default), `high`, `xhigh`.
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]

Default-change history note: across the GPT-5 series the implicit `reasoning.effort` default shifted between versions (GPT-5 defaulted to `medium`; the 5.1–5.4 line moved to `none`). Per-model defaults for 5.4 / 5.4-mini / 5.4-nano were not re-verified in this pass. Set `reasoning.effort` explicitly on migration rather than relying on an implicit default; an unexpected default is the most common cause of quality regressions on migration.

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

Setting `reasoning.summary: "auto"` returns a summary of the model's internal reasoning alongside the final output — useful for UX that wants to show the user what the model is considering. `"concise"` (supported by some computer-use models) and `"detailed"` (supported on o4-mini) are the other values. Summaries are opt-in.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

### Precedence within `input`

When both a `system`/`developer` message and a `user` message are present in `input`, the `system` / `developer` content takes precedence. Use `developer` role for constraints that must survive user override attempts; use `user` for the task itself.
[source: developers.openai.com/api/docs/api-reference/responses/create, retrieved 2026-04-18]

## 4. Context Window Practical Guidance

- **GPT-5.5**: 1,050,000 token context window; 128,000 token max output; knowledge cutoff 2025-12-01.
  [source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
- **GPT-5.5-pro**: 1,050,000 token context window; knowledge cutoff 2025-12-01.
  [source: developers.openai.com/api/docs/models/gpt-5.5-pro, retrieved 2026-06-01]
- **GPT-5.4**: 1,050,000 token context window; knowledge cutoff 2025-08-31. `gpt-5.4-mini`: 400K context.
  [source: developers.openai.com/api/docs/models, retrieved 2026-06-01]
- **Long-input surcharge**: prompts exceeding **272K tokens** incur **2×** input pricing and **1.5×** output pricing. Below 272K, standard pricing applies.
  [source: developers.openai.com/api/docs/models/gpt-5.4, retrieved 2026-04-18]

Practical implications:

- Stay under 272K tokens when possible to avoid the surcharge. Between 272K and 1M, factor the premium into cost estimates.
- Prompt caching covers up to 90% of input-token cost on repeated prefixes (see `openai-prompt-api.md` §7). Structure prompts to put stable content first.
- Native compaction support (GPT-5.4) and server-side `context_management.compact_threshold` can offload compaction from the caller.

[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]

## 5. Multimodal Conventions

Current flagship input modalities:

- **Text** — always.
- **Images** — via `input_image` content parts with `image_url` or uploaded `file_id`. A `detail` parameter tunes quality vs token cost ("low" / "high" / "auto"); caching requires the detail parameter to match across requests.
- **Files / documents** — via `input_file` content parts (uploaded document via Files API).
- **Audio** — supported on the Realtime / audio line, not on the GPT-5.x text flagships directly.

GPT-5.5 multimodal input conventions were not re-verified against the model card in this pass; the modality structure above reflects the GPT-5.4 retrieval and may not enumerate GPT-5.5 specifics.

[source: developers.openai.com/api/docs/models/gpt-5.4, retrieved 2026-04-18]
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

## 6. Behavioral Quirks

- **Reasoning effort defaults drifted between model versions.** GPT-5 defaulted to `medium`; the 5.1–5.4 line moved to `none`; `gpt-5.5` documents `medium` as default. Quality regressions on migration almost always trace to an unexpected effort default. Set `reasoning.effort` explicitly.
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]

- **Reasoning tokens are billed as output tokens** and are discarded from context after generating the visible response (but counted in `output_tokens_details.reasoning_tokens`).
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

- **Reasoning items must be preserved across tool-call turns.** If a turn includes a function call, you must include the reasoning items (via `previous_response_id` or explicit inclusion in `input`) on the follow-up request. Dropping them degrades multi-step reasoning quality.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]
[testable: id=openai.reasoning-items-preserved.v1, expected=multi-step reasoning quality degrades when reasoning items are dropped from the input on a function-call follow-up]

- **Structured-output refusal handling differs from plain-text.** When a request would be refused, the response is a `refusal` content block rather than schema-conforming JSON. Callers parsing `output_parsed` must handle the refusal case explicitly — a missing `output_parsed` is not necessarily a schema violation.
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

- **Tool calling with `reasoning: none` is not supported on Chat Completions for GPT-5.4.** This combination requires the Responses API — a breaking change compared to GPT-5.
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

- **Prompt cache routing is prefix-hash-based.** Requests with the same initial ~256-token prefix route to the same cache. Using `prompt_cache_key` gives explicit control over routing when the natural prefix hash would split cache usage.
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

## 7. Anti-Patterns

- **Do not start new work on Chat Completions.** It remains supported but is not recommended for new work. Use the Responses API.
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

- **Do not rely on an implicit reasoning default across versions.** The default differs between models (`gpt-5.5` documents `medium`; the 5.1–5.4 line uses `none`). Set `reasoning.effort` explicitly on migration.
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[source: developers.openai.com/api/docs/changelog, retrieved 2026-04-18]

- **Do not use "think step by step" / "reason carefully"** on reasoning-enabled effort levels. Use `reasoning.effort`.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

- **Do not combine tool use with `reasoning: none`** on Chat Completions for GPT-5.4. Either migrate to the Responses API or raise effort.
[source: developers.openai.com/api/docs/guides/migrate-to-responses, retrieved 2026-04-18]

- **Do not drop reasoning items between turns** when tool use is involved. Use `previous_response_id` or include them explicitly.
[source: developers.openai.com/api/docs/guides/reasoning, retrieved 2026-04-18]

- **Do not use `type: "json_object"` (legacy JSON mode) on new work.** It provides no schema adherence. Use `text.format` with `type: "json_schema"` and `strict: true`.
[source: developers.openai.com/api/docs/guides/structured-outputs, retrieved 2026-04-18]

- **Do not build on the Assistants API** — it was deprecated 2025-08-26 and is removed **2026-08-26**. The migration target is the **Responses API plus the Conversations API** (the named state-management replacement for Assistants threads), not the Responses API alone.
[source: developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]

- **Do not assume prefix hashing is content-insensitive.** Image `detail` parameter mismatches, tool-definition changes, or schema drift reset the cache silently.
[source: developers.openai.com/api/docs/guides/prompt-caching, retrieved 2026-04-18]

## 8. Gaps

- **Deep Research and the hosted agent surface.** OpenAI Deep Research and the hosted agent surface are documented in `resources/deep-research-agents.md` and `resources/agent-orchestration-surfaces.md`. Near-term doc/model mismatch to watch: Deep Research is still invoked with model IDs `o3-deep-research` and `o4-mini-deep-research` as of 2026-06-01, but those IDs shut down 2026-07-23 with `gpt-5.5-pro` as the replacement.
  [source: developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]
  [source: developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]
- **Multi-agent handoffs are Agents-SDK-only.** Handoff/orchestration is provided by the Python `agents` / JS `@openai/agents` SDKs; there is no hosted multi-agent handoff API endpoint. See `resources/agent-orchestration-surfaces.md`.
  [source: developers.openai.com/api/docs/guides/agents/orchestration, retrieved 2026-06-01]
- **Realtime API prompting patterns** (bidirectional audio, VAD, interruption handling) are not covered here. See the Realtime API docs.
- **Codex-specific prompting conventions** for `gpt-5.3-codex` beyond reasoning effort are not covered.
- **Per-model reasoning defaults for `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.4-nano`** were not re-verified in the 2026-06-01 pass.
- **Skills tool and Hosted Shell tool prompt-surface semantics** are new additions; detailed prompting conventions were not targeted in this retrieval pass.
- **Cross-language prompting differences** across OpenAI's multilingual support are not covered.
