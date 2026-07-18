---
family: glm
scope: api
versions:
  - glm-5.2
  - glm-5.1
  - glm-5
  - glm-5-turbo
  - glm-4.7
  - glm-4.6
  - glm-4.5
  - zai-org/GLM-5.2
retrieved: 2026-07-19
primary_sources:
  - https://huggingface.co/zai-org/GLM-5.2
  - https://huggingface.co/zai-org/GLM-5.2/blob/main/chat_template.jinja
  - https://docs.z.ai/guides/llm/glm-5.2
  - https://docs.z.ai/guides/llm/glm-5.1
  - https://docs.z.ai/guides/llm/glm-5-turbo
  - https://docs.z.ai/guides/llm/glm-4.7
  - https://docs.z.ai/guides/capabilities/thinking
  - https://docs.z.ai/guides/capabilities/thinking-mode
  - https://docs.z.ai/guides/overview/migrate-to-glm-new
  - https://docs.z.ai/guides/overview/pricing
  - https://docs.z.ai/devpack/quick-start
maturity_note: |
  API-call-level detail for the current GLM generation (Z.ai / zai-org). GLM-5.2
  is a 753B MoE with IndexShare sparse attention, 1M context, MIT open weights.
  Three call surfaces: native OpenAI-shaped endpoint, an Anthropic-compatible
  endpoint (api.z.ai/api/anthropic) usable as a Claude Code drop-in, and open
  weights. The reasoning-control surface has two entry points (`thinking.type`
  and a seven-value `reasoning_effort` ladder that shims onto two tiers). Tool
  calls use an XML wire format and require a dual stream/tool_stream flag for
  real-time argument emission. Prompt caching is automatic. Parallel tool calls
  are a confirmed documentation absence. Several Claude-Code-via-GLM env-var
  ergonomics are unconfirmed and declared as gaps rather than asserted.
---

# GLM — API-Layer Reference

API-call-level detail for the current GLM generation from Z.ai (`zai-org`). Portable prompt-layer content (model selection, thinking semantics, anti-patterns) lives in `glm-prompt.md`.

## 1. API Surface

### Base URLs

- **Anthropic-compatible**: `https://api.z.ai/api/anthropic` — a drop-in for Claude Code and other Anthropic-Messages clients via `ANTHROPIC_BASE_URL` + `ANTHROPIC_AUTH_TOKEN`.
[source: docs.z.ai/devpack/quick-start, retrieved 2026-07-19]
- **OpenAI-shaped native surface**: an OpenAI-compatible chat-completions surface exists (the migration and streaming docs describe `stream`/`tool_stream`, `reasoning_content`/`content`/`tool_calls` deltas, and `reasoning_effort`), but the exact base-URL path is not quoted in the retrieved primary sources for this pass. See Gaps.
[source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

Cross-family OpenAI-compatibility divergences (reasoning field name, round-trip rules, rejected params) are tabulated in `resources/openai-compatibility-surface.md`. GLM's own statements are kept here.

### Model IDs, context, and pricing

Pricing is USD per 1M tokens. Cache-write ("cached input storage") is documented as "Limited-time Free" across models.

| Model ID          | Context | Max output | Input | Output | Cached input (read) |
|-------------------|---------|------------|-------|--------|---------------------|
| `glm-5.2`         | 1,000,000 | 128K / 163,840 (disputed) | $1.40 | $4.40 | $0.26 |
| `glm-5.1`         | 200K    | 128K       | $1.40 | $4.40 | $0.26 |
| `glm-5`           | not documented at docs.z.ai/guides/llm, checked 2026-07-19 | — | $1.00 | $3.20 | $0.20 |
| `glm-5-turbo`     | 200K    | 128K       | $1.20 | $4.00 | $0.24 |
| `glm-4.7`         | 200K    | 128K       | $0.60 | $2.20 | $0.11 |
| `glm-4.7-flash`   | not documented, checked 2026-07-19 | — | Free | Free | Free |
| `glm-4.7-flashx`  | not documented, checked 2026-07-19 | — | $0.07 | $0.40 | $0.01 |
| `glm-4.6`         | not documented, checked 2026-07-19 | — | $0.60 | $2.20 | $0.11 |
| `glm-4.5`         | not documented, checked 2026-07-19 | — | $0.60 | $2.20 | $0.11 |
| `glm-4.5-x`       | not documented, checked 2026-07-19 | — | $2.20 | $8.90 | $0.45 |
| `glm-4.5-air`     | not documented, checked 2026-07-19 | — | $0.20 | $1.10 | $0.03 |
| `glm-4.5-airx`    | not documented, checked 2026-07-19 | — | $1.10 | $4.50 | $0.22 |
| `glm-4.5-flash`   | not documented, checked 2026-07-19 | — | Free | Free | Free |
| `glm-4-32b-0414-128k` | 128K | — | $0.10 | $0.10 | — |

[source: docs.z.ai/guides/overview/pricing, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-5.2, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-5.1, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-5-turbo, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-4.7, retrieved 2026-07-19]

**GLM-5.2 max output disputed:** docs.z.ai/guides/llm/glm-5.2 states 128K (128,000/131,072); the HuggingFace card states 163,840. [disputed: docs.z.ai = 128K; huggingface.co/zai-org/GLM-5.2 = 163,840]
[source: docs.z.ai/guides/llm/glm-5.2, retrieved 2026-07-19]
[source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

Note `glm-5.1` is priced identically to `glm-5.2` ($1.40 / $4.40 / $0.26) — confirmed on the canonical pricing page, not a scrape error.
[source: docs.z.ai/guides/overview/pricing, retrieved 2026-07-19]

### Open weights

`zai-org/GLM-5.2` — MIT license, no regional restriction, no revenue threshold. 753B-parameter MoE with IndexShare sparse attention (the indexer is reused across every four sparse-attention layers, ~2.9x FLOP reduction at 1M context).
[source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

## 2. Chat Template / Message Structure

### Role tokens (open weights)

| Token             | Purpose                         |
|-------------------|---------------------------------|
| `<|system|>`      | System-turn delimiter           |
| `<|user|>`        | User-turn delimiter             |
| `<|assistant|>`   | Assistant-turn delimiter        |
| `<|observation|>` | Tool / function-result delimiter |

[source: huggingface.co/zai-org/GLM-5.2/blob/main/chat_template.jinja, retrieved 2026-07-19]

Reasoning is wrapped in `<think>` … `</think>`.
[source: huggingface.co/zai-org/GLM-5.2/blob/main/chat_template.jinja, retrieved 2026-07-19]

### Tool-call wire format

Tool calls are emitted in an XML-shaped format at the template level, not as an OpenAI-style JSON `tool_calls` array inside the raw completion text:

```
<tool_call>{name}<arg_key>{k}</arg_key><arg_value>{v}</arg_value>…</tool_call>
```

Each argument is a `<arg_key>`/`<arg_value>` pair; multiple pairs stack inside one `<tool_call>` block.
[source: huggingface.co/zai-org/GLM-5.2/blob/main/chat_template.jinja, retrieved 2026-07-19]

## 3. Sampling Parameters

Vendor production recommendation:

- `temperature` default **1.0**.
- `top_p` default **0.95**.
- Tune only one at a time — "not recommended to adjust both simultaneously."

[source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

## 4. Reasoning / Thinking Control

### Two entry points

- **`thinking.type`** — `enabled` (default) or `disabled`. `enabled` lets the model auto-decide whether to think (on dynamic-thinking models).
- **`reasoning_effort`** — seven-value ladder, GLM-5.2-and-above only.

[source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

### `reasoning_effort` ladder and shim mapping

Accepted values: `max` (default), `xhigh`, `high`, `medium`, `low`, `minimal`, `none`. The vendor documents the mapping verbatim — the ladder collapses onto two thinking tiers plus a no-think path:

| Requested value      | Effective behavior |
|----------------------|--------------------|
| `none`, `minimal`    | Skip thinking      |
| `low`, `medium`      | Mapped to **high** |
| `high`               | High tier          |
| `xhigh`              | Mapped to **max**  |
| `max` (default)     | Max tier           |

[source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]
[testable: id=glm.reasoning-effort-shim.v1, expected=requests with reasoning_effort=low or medium produce the same thinking-tier behavior as reasoning_effort=high on glm-5.2]

This is the OpenAI-compat reconciliation point: an OpenAI-style caller's `low`/`medium`/`high`/`xhigh` are shimmed as above, so the value names do not carry OpenAI semantics. `reasoning_effort` is rejected/ignored on pre-5.2 models — those use `thinking.type`.

### Dynamic vs forced thinking

- **Dynamic** (model decides): GLM-5.2, GLM-5.1, GLM-5, GLM-5-Turbo, GLM-5V-Turbo, GLM-4.6, GLM-4.5.
- **Forced** (always thinks): GLM-4.7, GLM-4.5V.

[source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

### Turn-level effort guidance

Reduce reasoning on quick tool-execution turns; enable deeper thinking on decision turns. Dynamically balance across the trajectory.
[source: docs.z.ai/guides/capabilities/thinking-mode, retrieved 2026-07-19]

### Multi-turn reasoning contract (`clear_thinking`)

`clear_thinking: false` preserves prior-turn thinking in the multi-turn context. Its **default differs by endpoint**: enabled (thinking cleared) on the Coding Plan endpoint, disabled (thinking preserved) on the standard API.

When preserving thinking, the caller must return the complete, unmodified `reasoning_content`; blocks must exactly match the original order. Reordering or editing them is explicitly warned against — "performance may degrade and cache hit rates may be affected."
[source: docs.z.ai/guides/capabilities/thinking-mode, retrieved 2026-07-19]

## 5. Tool Use / Function Calling

### Streaming tool arguments requires two flags

Real-time tool-argument emission requires **both** `stream=true` **and** `tool_stream=true`. With `tool_stream` unset, tool-call arguments are not streamed incrementally.
[source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

### Streaming delta structure

Three separate delta streams are emitted; the client concatenates each independently:

- `delta.reasoning_content` — thinking text.
- `delta.content` — final answer text.
- `delta.tool_calls[*].function.arguments` — tool-call argument fragments (when `tool_stream=true`).

[source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

### Parallel tool calls — confirmed absence

Parallel tool calls are **not documented** on the first-party API. The streaming contract describes single-accumulation `tool_calls[*]`. Treat parallel tool calling as unsupported/undocumented, not merely unmentioned.
[source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

## 6. Structured Outputs

A dedicated JSON-schema / grammar-constrained structured-output parameter is not documented in the retrieved primary sources for this pass. Use function calling with an explicit schema as the structured-extraction path. See Gaps.

## 7. Caching, Batch, Streaming

### Prompt caching (automatic / implicit prefix caching)

- Caching is **automatic** — cached input is separately priced on the pricing page (e.g. `glm-5.2` cached-input read $0.26/1M vs $1.40/1M input), and cache-write storage is "Limited-time Free."
[source: docs.z.ai/guides/overview/pricing, retrieved 2026-07-19]
- **No minimum cacheable block size is documented at z.ai** (checked 2026-07-19). A 512-token minimum appears only on a third-party integration doc and is not a z.ai-documented figure. [community-reported] [unverified]
- Prefix-cache invalidation on prefix mutation is generic prefix-cache behavior; no z.ai-specific mechanism (radix-tree or otherwise) is confirmed on a canonical page.

### Streaming

OpenAI-shaped SSE streaming. In thinking mode the three deltas above (`reasoning_content`, `content`, `tool_calls[*].function.arguments`) are emitted in separate fields; the client concatenates each.
[source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

### Batch

Not covered in the retrieved primary sources for this pass. See Gaps.

## 8. Deployment Flags

### Anthropic-compatible surface (Claude Code drop-in)

Point an Anthropic-Messages client at the compat endpoint:

```
ANTHROPIC_BASE_URL=https://api.z.ai/api/anthropic
ANTHROPIC_AUTH_TOKEN=sk-REPLACE-ME   # a Z.ai API key
```

[source: docs.z.ai/devpack/quick-start, retrieved 2026-07-19]

The GLM-specific facts of this compat surface (reasoning field name, `reasoning_effort` shim) live in this file; the cross-family divergence matrix is in `resources/openai-compatibility-surface.md`.

### Open weights

`zai-org/GLM-5.2` — MIT, 753B MoE, IndexShare sparse attention, 1M context. Minimum vLLM / SGLang versions for the IndexShare attention path were not quoted in the retrieved primary sources; verify against release notes at deployment time. See Gaps.
[source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

## 9. Deprecations and Breaking Changes

- **`reasoning_effort` is GLM-5.2-and-above only.** Older models use `thinking.type`; passing `reasoning_effort` to them is not supported. [source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]
- **Forced-thinking models cannot suppress thinking.** GLM-4.7 and GLM-4.5V always think; `thinking.type: "disabled"` does not apply. [source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]
- **License.** MIT (weights), no regional restriction, no revenue threshold. [source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

## 10. Gaps

- **GLM-5.2 max output** — disputed between vendor surfaces (docs.z.ai 128K vs HF card 163,840); carried as a disagreement.
- **The `glm-5.2[1m]` Claude-Code model alias** is referenced only in aggregator summaries, not confirmed on a canonical docs.z.ai / devpack page as checked 2026-07-19. It ships only once verified; until then it is a declared gap, not an asserted fact.
- **`ANTHROPIC_DEFAULT_OPUS_MODEL` / `CLAUDE_CODE_AUTO_COMPACT_WINDOW` values for GLM** — the env vars are real Claude Code variables, but the specific GLM values are not confirmed on a canonical vendor page (checked 2026-07-19). Declared gap.
- **Maximum functions per `tools` array** (a 128-function cap was referenced but not re-verified against docs.z.ai/api-reference/llm/chat-completion) — unverified; not asserted.
- **Structured-output / JSON-schema parameter** — no dedicated mechanism documented in the retrieved sources.
- **Batch API** — shape / availability not covered.
- **Cache minimum block size** — no documented z.ai minimum (confirmed absence); a 512-token figure exists only third-party.
- **Vision variants (`glm-5v-turbo`, `glm-4.5v`)** — context window and pricing not confirmed on a canonical multimodal page.
- **vLLM / SGLang minimum versions** for the IndexShare attention path are not quoted in the retrieved sources.
- **Native OpenAI-shaped base-URL path** — the surface is documented behaviorally but the exact endpoint URL is not quoted in the retrieved sources.
- **Knowledge cutoff dates** for any GLM model are not documented in the retrieved primary sources.
