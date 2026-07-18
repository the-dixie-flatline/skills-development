---
family: minimax
scope: api
versions:
  - MiniMax-M3
  - MiniMax-M2.7
  - MiniMax-M2.7-highspeed
  - MiniMax-M2.5
  - MiniMax-M2.5-highspeed
  - MiniMax-M2.1
  - MiniMax-M2.1-highspeed
  - MiniMax-M2
  - M2-her
  - MiniMaxAI/MiniMax-M3
retrieved: 2026-07-19
primary_sources:
  - https://platform.minimax.io/docs/guides/text-generation
  - https://platform.minimax.io/docs/guides/models-intro
  - https://platform.minimax.io/docs/api-reference/text-anthropic-api
  - https://platform.minimax.io/docs/api-reference/text-prompt-caching
  - https://platform.minimax.io/docs/guides/pricing-paygo
  - https://platform.minimax.io/docs/guides/pricing-token-plan
  - https://platform.minimax.io/docs/token-plan/claude-code
  - https://huggingface.co/MiniMaxAI/MiniMax-M3
  - https://huggingface.co/MiniMaxAI/MiniMax-M3/blob/main/LICENSE
  - https://huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja
  - https://www.minimax.io/models/text/m3
maturity_note: |
  MiniMax-M3 is the current flagship: ~428B-total / ~23B-active MoE, 1M context,
  Multi-Scale Attention (MSA). The vendor exposes three API surfaces and recommends
  the Anthropic-compatible `/anthropic/v1/messages` path; an OpenAI-compatible
  `/v1/chat/completions` surface and a native legacy `chatcompletion_v2` endpoint
  also exist. Open weights are published on HuggingFace under the MiniMax Community
  License (non-commercial by default). Thinking is off-by-default on M3 (opt-in) and
  un-disable-able on M2.x. Caching is automatic/passive on M3; the explicit
  `cache_control` mechanism applies to M2.x only. Re-verify by 2026-10-19 (90 days).
---

# MiniMax — API-Layer Reference

API-call-level detail for the current MiniMax generation. Portable prompt-layer content (selection, thinking semantics, anti-patterns) lives in `minimax-prompt.md`.

## 1. API Surface

The vendor documents three surfaces and foregrounds the Anthropic-compatible path as the recommended/default one.

| Surface                              | Base URL                                                     | Endpoint                              |
|--------------------------------------|-------------------------------------------------------------|---------------------------------------|
| Anthropic-compatible (recommended)   | `https://api.minimax.io/anthropic` (CN: `https://api.minimaxi.com/anthropic`) | `POST /anthropic/v1/messages`         |
| OpenAI-compatible                    | `https://api.minimax.io/v1`                                 | `POST /v1/chat/completions`           |
| Native (legacy)                      | `https://api.minimax.io/v1`                                 | `POST /v1/text/chatcompletion_v2`     |

[source: platform.minimax.io/docs/guides/text-generation, retrieved 2026-07-19]
[source: www.minimax.io/models/text/m3, retrieved 2026-07-19]

Center integration work on the Anthropic-compatible path. The native `chatcompletion_v2` endpoint remains but is a legacy third surface, not the primary interface.

Cross-family OpenAI-compatibility divergences (which params are silently ignored, cache-token field names) are tabulated in `resources/openai-compatibility-surface.md`. MiniMax's own statements are kept here.

### Model IDs

`MiniMax-M3` (1M context), `MiniMax-M2.7` / `-highspeed`, `MiniMax-M2.5` / `-highspeed`, `MiniMax-M2.1` / `-highspeed`, `MiniMax-M2` (all 204,800 context), and the `M2-her` chat model (64K, standard API only, not on the Anthropic-compatible set). The Anthropic-compatible path is documented for M3 and the M2.x line.
[source: platform.minimax.io/docs/guides/text-generation, retrieved 2026-07-19]
[source: platform.minimax.io/docs/guides/models-intro, retrieved 2026-07-19]

### Open weights

`MiniMaxAI/MiniMax-M3` — HuggingFace, MiniMax Community License, BF16, HF widget reports 427B params (~428B total / ~23B active). Ships weights, tokenizer, and `chat_template.jinja`, with a companion GitHub repo and the MSA kernel repo. MSA design paper: arXiv 2606.13392.
[source: huggingface.co/MiniMaxAI/MiniMax-M3, retrieved 2026-07-19]

## 2. Chat Template / Message Structure

### Open-weights template

`MiniMaxAI/MiniMax-M3` ships `chat_template.jinja`. The template uses non-standard special-bracket control tokens rather than the ASCII `<|...|>` form used by most open-weights families. Observed bracket-token forms include `]~b]`, `[e~[`, and `]<]image[>[`. Use `apply_chat_template`; do not hand-assemble prompt strings.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]

The template default system message identifies the model as "MiniMax-M3, developed by MiniMax" with a knowledge cutoff of January 2026; the default developer message is "You are a helpful assistant."
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]

When thinking mode is enabled, the template injects: "Current thinking mode: enabled. You MUST think step by step before every response, including after receiving function/tool results."
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]

### Hosted message shape (Anthropic-compatible)

Standard Anthropic Messages shape: top-level `system`, then `messages` with `user` / `assistant` roles and content blocks (`text`, `thinking`, `image`, `tool_use`, `tool_result`).
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

## 3. Sampling Parameters

Anthropic-compatible path (canonical: Anthropic API reference "Supported Parameters"):

| Param                          | Value / status                                                        |
|--------------------------------|-----------------------------------------------------------------------|
| `temperature`                  | Range [0, 2]; recommended 1; out-of-range values error               |
| `top_p`                        | Default 0.95 (M3), 0.9 (M2.x); range [0, 1]                            |
| `top_k`                        | **Ignored by the API** (local-inference vLLM suggestion only)         |
| `max_tokens`                   | **Fully supported** on the Anthropic path                             |
| `service_tier`                 | `standard` (default) or `priority` (1.5x price, priority admission)   |
| `stop_sequences`               | **Ignored**                                                           |
| `mcp_servers` / `context_management` / `container` | **Ignored**                                       |

[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

For the open weights, the HF card recommends local sampling of temperature=1.0, top_p=0.95.
[source: huggingface.co/MiniMaxAI/MiniMax-M3, retrieved 2026-07-19]

## 4. Reasoning / Thinking Control

### The `thinking` parameter

Control reasoning via `thinking: {"type": ...}`:

- `{"type": "adaptive"}` — enables thinking (adaptive == on for M3).
- `{"type": "disabled"}` — keeps thinking off.
- **M3 default**: thinking is **OFF** when `thinking` is omitted.
- **M2.x**: thinking **cannot be disabled**; `disabled` is accepted but ignored, and thinking stays on.

```python
# M3: opt in to reasoning (off by default)
resp = client.messages.create(
    model="MiniMax-M3",
    max_tokens=4096,
    thinking={"type": "adaptive"},
    messages=[...],
)
```

[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

Note the layering: the raw `chat_template.jinja` default (when `thinking_mode` is undefined) is "adaptive"; the hosted API overrides M3 to off-by-default; the vendor's Claude Code integration turns extended thinking on by default in that client. Report all three; they differ.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]
[source: platform.minimax.io/docs/token-plan/claude-code, retrieved 2026-07-19]

### Reasoning surfacing per path

- **Anthropic path** — structured `thinking` content blocks (`block.type == "thinking"`; streaming `thinking_delta`).
- **OpenAI path** — reasoning inline in `content` by default; set `extra_body={"reasoning_split": True}` to split thinking into a `reasoning_details` field. `reasoning_split` and `reasoning_details` are first-party OpenAI-compat fields (confirmed in the caching doc), not merely router concepts.

[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]
[source: platform.minimax.io/docs/api-reference/text-prompt-caching, retrieved 2026-07-19]

### Multi-turn reasoning-preservation contract

The vendor states, for multi-turn (especially tool-use) conversations:

- Append the **complete model response** (the full `response.content` list: thinking / text / tool_use) to conversation history to "maintain the continuity of the reasoning chain."
- When a response includes `thinking` blocks, **preserve them unchanged** in later turns.

The first-party consequence of omission is **reasoning-chain degradation**, not a hard HTTP 400. A hard 400 on missing reasoning content is a router-surface behavior (OpenRouter), Tier-2, and is not a first-party MiniMax contract.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

## 5. Tool Use / Function Calling

- Anthropic path tool-use content types: `tool_use` (assistant) and `tool_result` (user).
- The open-weights template renders multiple `tool_calls` inside a single `toolcall` block, so **parallel tool calls are supported**.

[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]

When thinking is enabled, the template requires the model to think after receiving function/tool results (see §2). Combined with the §4 preservation contract, tool-use turns must re-append prior `thinking` blocks unchanged.

## 6. Structured Outputs

A dedicated JSON-schema / grammar-constrained structured-output parameter is not documented at `platform.minimax.io/docs/api-reference/text-anthropic-api`, checked 2026-07-19. Use function calling (`tool_use`) as the structured-extraction path. See §10.

## 7. Caching, Batch, Streaming

### Automatic (passive) caching

- Applies to calls with **≥ 512 input tokens**.
- Prefix matching in the order **tool list → system prompts → user messages**.
- Expiration **auto-adjusts with system load** (not a fixed 5-minute TTL, not a fixed backward-block scan).
- Supported on M3, M2.7, M2.5, M2.1 series.

[source: platform.minimax.io/docs/api-reference/text-prompt-caching, retrieved 2026-07-19]

### Explicit caching (`cache_control`, Anthropic path)

- 5-minute expiration, auto-renewed on hit.
- First-time cache **writes incur a charge**.
- Supported on M2.7 / M2.5 / M2.1 / M2 series. **Not M3** — M3 is passive-only.

[source: platform.minimax.io/docs/api-reference/text-prompt-caching, retrieved 2026-07-19]

### Cache-token reporting fields

- **OpenAI path**: `usage.prompt_tokens_details.cached_tokens`.
- **Anthropic path**: `usage.cache_read_input_tokens` and `usage.cache_creation_input_tokens`.

[source: platform.minimax.io/docs/api-reference/text-prompt-caching, retrieved 2026-07-19]

### Streaming

Anthropic-path streaming emits `thinking_delta` events for reasoning blocks alongside text deltas. Full SSE event-type enumeration was not quoted from the surveyed pages.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

### Batch

Not documented at the surveyed pages, checked 2026-07-19.

## 8. Deployment Flags

### Pricing — PAYG (standard tier)

Prices below reflect the vendor's "permanent 50% off" promotional PAYG rates, per million tokens.

| Model                       | Input   | Output  | Cache read | Cache write |
|-----------------------------|---------|---------|------------|-------------|
| M3, ≤512k input             | $0.30/M | $1.20/M | $0.06/M    | none        |
| M3, >512k input             | $0.60/M | $2.40/M | $0.12/M    | none        |
| M3, ≤512k input, `priority` | $0.45/M | $1.80/M | $0.09/M    | none        |
| M3, >512k input, `priority` | $0.90/M | $3.60/M | $0.18/M    | none        |
| M2.7                        | $0.30/M | $1.20/M | $0.06/M    | $0.375/M    |
| M2.7-highspeed              | $0.60/M | $2.40/M | $0.06/M    | $0.375/M    |
| M2.5 / M2.1 / M2            | $0.30/M | $1.20/M | $0.03/M    | $0.375/M    |
| M2.5-hs / M2.1-hs           | $0.60/M | $2.40/M | $0.03/M    | $0.375/M    |

- The long-context pricing tier triggers when **input tokens exceed 512k** (including cache-hit tokens), not on total serialized payload.
- `-highspeed` variants are priced at 2x the base string's input/output.
- M3 has no cache-write charge (passive-only caching); M2.x carry a cache-write charge (explicit caching).

[source: platform.minimax.io/docs/guides/pricing-paygo, retrieved 2026-07-19]

### Pricing — Token Plan (subscription)

| Tier  | Price   | Agents | Windows                    |
|-------|---------|--------|----------------------------|
| Plus  | $20/mo  | 3–4    | 5-hour rolling + weekly    |
| Max   | $50/mo  | 4–5    | 5-hour rolling + weekly    |
| Ultra | $120/mo | 6–7    | 5-hour rolling + weekly    |

Quota is shared across text/image/speech/music via a Subscription Key. Credits (1000 credits = $1; $5/$25/$100 packs, 365-day validity) cover PAYG-priced overflow after quota.
[source: platform.minimax.io/docs/guides/pricing-token-plan, retrieved 2026-07-19]

### Claude Code deployment

The vendor documents Claude Code against the Anthropic-compatible base URL. Verbatim configuration:

```bash
unset ANTHROPIC_AUTH_TOKEN
unset ANTHROPIC_BASE_URL
```
```json
{ "env": {
  "ANTHROPIC_BASE_URL": "https://api.minimax.io/anthropic",
  "ANTHROPIC_AUTH_TOKEN": "<MINIMAX_API_KEY>",
  "CLAUDE_CODE_AUTO_COMPACT_WINDOW": "1000000",
  "ANTHROPIC_MODEL": "MiniMax-M3[1m]",
  "ANTHROPIC_DEFAULT_SONNET_MODEL": "MiniMax-M3[1m]",
  "ANTHROPIC_DEFAULT_OPUS_MODEL": "MiniMax-M3[1m]",
  "ANTHROPIC_DEFAULT_HAIKU_MODEL": "MiniMax-M3[1m]" }}
```

CN region uses `https://api.minimaxi.com/anthropic`. Verify with `/status` (base URL) and `/model` (shows `MiniMax-M3`). Extended thinking is on by default in Claude Code; toggle via `/config` Thinking mode or Option+T / Alt+T. Other coding-agent integrations are documented under `/docs/token-plan/` (Roo Code, Kilo Code, Cline, Codex CLI, OpenCode, Droid, TRAE, Grok CLI, Cursor, OpenClaw, Hermes Agent).
[source: platform.minimax.io/docs/token-plan/claude-code, retrieved 2026-07-19]

### Open-weights self-hosting

`MiniMaxAI/MiniMax-M3` provides weights + tokenizer + `chat_template.jinja`; the HF card lists deployment recipes (SGLang / vLLM / KTransformers / unsloth). MSA requires the `kernels/MiniMaxAI/msa` op. Whether stock vLLM / SGLang builds include the MSA kernel (versus a dense text-only fallback) is community-reported, not vendor-confirmed. [community-reported]
[source: huggingface.co/MiniMaxAI/MiniMax-M3, retrieved 2026-07-19]

License: MiniMax Community License — non-commercial by default; commercial use requires "Built with MiniMax M3" attribution and license-notice retention; written authorization required above US$20M yearly revenue.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/blob/main/LICENSE, retrieved 2026-07-19]

## 9. Deprecations and Breaking Changes

- **Current vs Legacy split.** The vendor marks M3, M2.7, and M2.7-highspeed as Current; M2.5/-hs, M2.1/-hs, and M2 as Legacy. Start new work on Current IDs.
[source: platform.minimax.io/docs/guides/models-intro, retrieved 2026-07-19]

- **`chatcompletion_v2` is a legacy surface.** Prefer the Anthropic-compatible `/anthropic` path. The native endpoint remains available but is no longer the primary interface.
[source: platform.minimax.io/docs/guides/text-generation, retrieved 2026-07-19]

- **Stale marketing "coming soon" for M3 open weights.** The marketing page understates availability; weights are published. Trust the HuggingFace artifact repository.
[source: huggingface.co/MiniMaxAI/MiniMax-M3, retrieved 2026-07-19]

## 10. Gaps

- **M3 max-output token cap** is not documented at `platform.minimax.io/docs/guides/text-generation`, checked 2026-07-19. The Anthropic path takes a user-set `max_tokens`; the ceiling is unstated. (M2 documents 128k output including CoT.)
- **Native `chatcompletion_v2` request/response fields** (for example `mask_sensitive_info`, `input_sensitive`, `base_resp`) are not documented at the Anthropic/OpenAI compat references, checked 2026-07-19. Verifying them requires the native-endpoint reference, not fetched this pass.
- **Dedicated structured-output / JSON-schema parameter** is not documented at the surveyed pages, checked 2026-07-19. Function calling is the documented extraction path.
- **Batch API** shape / availability is not documented at the surveyed pages, checked 2026-07-19.
- **Token Plan rolling-quota percentages** (reported as 30%/60% elsewhere) are not on `platform.minimax.io/docs/guides/pricing-token-plan`, checked 2026-07-19; only the window structure (5-hour rolling + weekly) is confirmed.
- **CN/RMB Token Plan pricing** is not reconciled against the EN surface this pass.
- **Literal thinking wire-token** (`<think>` vs a special-bracket token) is unresolved; the markdown-rendered template stripped the `think_begin`/`think_end` token values.
- **Full SSE streaming event-type enumeration** was not quoted from the surveyed pages.
