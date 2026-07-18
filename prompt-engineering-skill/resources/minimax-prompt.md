---
family: minimax
scope: prompt
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
  - https://platform.minimax.io/docs/token-plan/claude-code
  - https://huggingface.co/MiniMaxAI/MiniMax-M3
  - https://huggingface.co/MiniMaxAI/MiniMax-M3/blob/main/LICENSE
  - https://huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja
maturity_note: |
  MiniMax-M3 is the current flagship: a ~428B-total / ~23B-active MoE with a 1M
  context window and Multi-Scale Attention (MSA). Open weights ARE published on
  HuggingFace (weights, tokenizer, chat_template) under the MiniMax Community
  License; the vendor marketing page still carries stale "coming soon" copy, but
  the artifacts are live. M3 is multimodal (text + image + video); the M2.x line
  is text + tool-call only. Thinking is OFF-by-default on M3 (opt-in) but cannot be
  disabled on M2.x. The vendor foregrounds an Anthropic-compatible (`/anthropic`)
  API path as the recommended surface. Content re-verify by 2026-10-19 (90 days).
---

# MiniMax — Prompt-Layer Reference

Portable prompting guidance for the current MiniMax generation. API-layer detail (endpoints, sampling parameters, caching mechanics, chat-template tokens) lives in `minimax-prompt-api.md`.

## 1. Model Selection

Current flagship is MiniMax-M3. The vendor splits the hosted lineup into a **Current** set and a **Legacy** set.

| Model ID                  | Context   | Status  | Notes                                                        |
|---------------------------|-----------|---------|--------------------------------------------------------------|
| `MiniMax-M3`              | 1,000,000 | Current | Flagship; multimodal (text + image + video); MoE ~428B/~23B active |
| `MiniMax-M2.7`           | 204,800   | Current | Text + tool-call; ~60 tps                                     |
| `MiniMax-M2.7-highspeed` | 204,800   | Current | Same model, higher throughput (~100 tps); priced ~2x base    |
| `MiniMax-M2.5`           | 204,800   | Legacy  | ~60 tps                                                      |
| `MiniMax-M2.5-highspeed` | 204,800   | Legacy  | ~100 tps                                                    |
| `MiniMax-M2.1`           | 204,800   | Legacy  | 230B total / 10B active                                      |
| `MiniMax-M2.1-highspeed` | 204,800   | Legacy  | ~100 tps                                                    |
| `MiniMax-M2`             | 204,800   | Legacy  | Max output 128k tokens including CoT                          |
| `M2-her`                 | 64K       | Chat    | Chat / role-play model; not in the Anthropic-compatible set  |

[source: platform.minimax.io/docs/guides/text-generation, retrieved 2026-07-19]
[source: platform.minimax.io/docs/guides/models-intro, retrieved 2026-07-19]

### Selection rules

- **General use, long context, or multimodal** — `MiniMax-M3`. It is the only model in the lineup with a 1M window and the only one accepting image/video input.
- **Cost-sensitive text-only work at 200K context** — `MiniMax-M2.7`. Use `MiniMax-M2.7-highspeed` when latency matters more than the ~2x token price.
- **Chat / role-play** — `M2-her`. It is a distinct chat model at 64K context, not exposed on the Anthropic-compatible path; call it over the standard API.
- **Self-hosted** — `MiniMaxAI/MiniMax-M3` open weights (MiniMax Community License; non-commercial by default). See §6 and the API-layer file for licensing constraints.
- **New work on M2.5 / M2.1 / M2** — avoid. These are Legacy; prefer M2.7 or M3.

### Open weights

`MiniMaxAI/MiniMax-M3` publishes weights, tokenizer, and `chat_template.jinja` on HuggingFace (BF16, HF widget reports 427B params), with a companion GitHub repo and the MSA kernel repo. The MSA design paper is arXiv 2606.13392.
[source: huggingface.co/MiniMaxAI/MiniMax-M3, retrieved 2026-07-19]

License is the **MiniMax Community License**: non-commercial by default; commercial use requires a "Built with MiniMax M3" attribution and retention of the license notice; written authorization is required above US$20M yearly revenue.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/blob/main/LICENSE, retrieved 2026-07-19]

The vendor marketing page `www.minimax.io/models/text/m3` still says M3 "will soon be fully open-sourced." That copy is stale; the artifacts are already published on HuggingFace. Trust the artifact repository.
[source: huggingface.co/MiniMaxAI/MiniMax-M3, retrieved 2026-07-19]

## 2. Prompt Structure Conventions

The hosted API foregrounds an **Anthropic-compatible** message surface (`/anthropic/v1/messages`) as the recommended path, with an OpenAI-compatible (`/v1/chat/completions`) surface as the alternative and a native `chatcompletion_v2` endpoint as a third/legacy surface. Prompts portable from the Anthropic Messages API generally work with a base-URL swap. Endpoint detail is in `minimax-prompt-api.md`.
[source: platform.minimax.io/docs/guides/text-generation, retrieved 2026-07-19]

### Identity and knowledge cutoff

The open-weights chat template's default system message identifies the model as "MiniMax-M3, developed by MiniMax" with a **knowledge cutoff of January 2026**. The default developer message is "You are a helpful assistant." These are template defaults, not a fixed vendor system prompt you must reproduce.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]

### Roles

The open-weights template distinguishes a system message (identity/cutoff) from a developer message (task instructions). On the hosted Anthropic-compatible path, use the standard Anthropic system + user/assistant message shape.

## 3. Instruction Patterns

### Thinking is opt-in on M3, mandatory on M2.x

There are three distinct layers, and they differ:

- **Hosted API, M3** — thinking is **OFF by default** when the `thinking` parameter is omitted. Enable it with `thinking: {"type": "adaptive"}`; `{"type": "disabled"}` keeps it off.
- **Hosted API, M2.x** — thinking **cannot be disabled**. `{"type": "disabled"}` is accepted but ignored; thinking stays on.
- **Raw open-weights template** — when the template's `thinking_mode` variable is undefined, the default is "adaptive" (on). The hosted API overrides M3 to off-by-default.
- **Claude Code integration** — the vendor's Claude Code deployment page states extended thinking is **on by default** in that client.

[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]
[source: platform.minimax.io/docs/token-plan/claude-code, retrieved 2026-07-19]

When thinking mode is enabled, the template's own instruction text reads: "You MUST think step by step before every response, including after receiving function/tool results."
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]

### Preserve reasoning across turns

This is a hard multi-turn contract, not a stylistic suggestion. When a model response includes `thinking` blocks, **append the full response content list unchanged** (thinking + text + tool_use) to conversation history before the next turn. The vendor states this maintains "the continuity of the reasoning chain," particularly in tool-use conversations. Dropping the thinking blocks degrades the reasoning chain.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

The first-party consequence of omission is **degradation**, not a hard error. A hard HTTP 400 on missing reasoning content is a router-surface (OpenRouter) behavior, Tier-2, not a first-party MiniMax contract. Treat reasoning preservation as required regardless.

### Reasoning surfacing differs by path

- **Anthropic path** — reasoning is returned as structured `thinking` content blocks.
- **OpenAI path** — reasoning is inline in `content` by default; set `extra_body={"reasoning_split": True}` to split it into a separate `reasoning_details` field.

[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]
[source: platform.minimax.io/docs/api-reference/text-prompt-caching, retrieved 2026-07-19]

## 4. Context Window Practical Guidance

- **M3 context is 1M tokens.** The M2.x line is 204,800; `M2-her` is 64K.
[source: platform.minimax.io/docs/guides/text-generation, retrieved 2026-07-19]

- **The 512K boundary is a pricing tier, not an availability cliff.** M3 input priced in two tiers: at or below 512K input tokens vs above 512K input tokens (the >512K tier is 2x the input/output rate). The trigger is **input tokens** (including cache-hit tokens), not the total serialized payload.
[source: platform.minimax.io/docs/guides/pricing-paygo, retrieved 2026-07-19]

- **"Guaranteed minimum 512K" is a capacity floor**, separate from the pricing tier. The 1M window is the ceiling; 512K is the guaranteed-available floor. Do not conflate the two.
[source: platform.minimax.io/docs/guides/pricing-paygo, retrieved 2026-07-19]

- Token ratio heuristic: ~750 English words is roughly 1000 tokens.
[source: platform.minimax.io/docs/guides/pricing-paygo, retrieved 2026-07-19]

## 5. Multimodal Conventions

**Only M3 is multimodal.** The M2.x line accepts text and tool-call content blocks only; no image or video.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

M3 image and video limits (Anthropic-compatible path):

| Aspect        | Value                                                                          |
|---------------|--------------------------------------------------------------------------------|
| Image         | URL or base64; JPEG / PNG / GIF / WEBP; up to 10 MB                             |
| Video         | URL, base64, or `mm_file://{file_id}`; MP4 / AVI / MOV / MKV; up to 50 MB inline, up to 512 MB via Files API |
| Request body  | 64 MB cap                                                                      |
| OpenAI path   | Uses `image_url` and `video_url` content parts                                 |

[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

Image token cost is governed by a `detail`-based heuristic: `low` up to ~600 tokens; `default` ~1k-3k (up to ~5k); `high` several-thousand (up to ~15k+). For an exact count, call `POST /anthropic/v1/messages/count_tokens` or read the response `usage`.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

## 6. Behavioral Quirks

- **Thinking OFF-by-default on M3, un-disable-able on M2.x.** The default differs by model within the same family. Omitting `thinking` gives no reasoning on M3 but full reasoning on M2.x.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

- **`top_k` is silently ignored** by the hosted API. It is only a local-inference (vLLM) suggestion; the HF card does not set it, and the API does not honor it.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]
[source: huggingface.co/MiniMaxAI/MiniMax-M3, retrieved 2026-07-19]

- **`stop_sequences` is silently ignored** on the Anthropic path, as are `mcp_servers`, `context_management`, and `container`.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

- **Special-bracket chat-template vocabulary.** The open-weights template uses non-standard bracket tokens (for example `]~b]`, `[e~[`, `]<]image[>[`) rather than the ASCII `<|...|>` form common to other open-weights families. Use `apply_chat_template`; do not hand-assemble prompt strings.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]

- **Stale marketing copy.** The M3 marketing page's "coming soon" open-source claim is outdated; the weights are published. Do not conclude from the marketing page that weights are unavailable.
[source: huggingface.co/MiniMaxAI/MiniMax-M3, retrieved 2026-07-19]

- **Non-commercial default license.** Unlike MIT-licensed open-weights peers, `MiniMaxAI/MiniMax-M3` is non-commercial by default and imposes attribution plus a revenue-gated authorization requirement for commercial use.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/blob/main/LICENSE, retrieved 2026-07-19]

## 7. Anti-Patterns

- **Do not omit `thinking` on M3 and expect reasoning.** M3 defaults to off. Set `thinking: {"type": "adaptive"}`.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

- **Do not expect `{"type": "disabled"}` to silence M2.x.** It is accepted and ignored; M2.x always thinks.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

- **Do not strip `thinking` blocks from prior turns.** Re-append the full response content unchanged to preserve the reasoning chain, especially in tool-use conversations.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

- **Do not set `top_k` or `stop_sequences` expecting an effect on the hosted API.** Both are ignored. Control length via `max_tokens` and output shape via the prompt.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

- **Do not send image or video to any M2.x model.** Only M3 accepts multimodal input.
[source: platform.minimax.io/docs/api-reference/text-anthropic-api, retrieved 2026-07-19]

- **Do not hand-build open-weights prompt strings with ASCII delimiters.** The template uses special-bracket tokens; use `apply_chat_template`.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/chat_template.jinja, retrieved 2026-07-19]

- **Do not center integration work on `chatcompletion_v2`.** The vendor recommends the Anthropic-compatible `/anthropic` path; the native `chatcompletion_v2` endpoint is a legacy third surface.
[source: platform.minimax.io/docs/guides/text-generation, retrieved 2026-07-19]

- **Do not use `MiniMaxAI/MiniMax-M3` commercially without meeting the license terms.** Attribution and (above US$20M yearly revenue) written authorization are required.
[source: huggingface.co/MiniMaxAI/MiniMax-M3/blob/main/LICENSE, retrieved 2026-07-19]

## 8. Gaps

- **M3 max-output token cap** is not documented at `platform.minimax.io/docs/guides/text-generation`, checked 2026-07-19. The Anthropic path takes a user-set `max_tokens`; the ceiling is unstated. (M2 documents 128k output including CoT.)
- **Literal thinking wire-token** (whether the open-weights template emits a literal `<think>` delimiter or a special-bracket token) is unresolved; the markdown-rendered template stripped the `think_begin`/`think_end` token values. API-visible reasoning is structured `thinking` blocks (Anthropic) or `reasoning_details` (OpenAI split) regardless.
- **Knowledge cutoff for the M2.x line** is not documented; only M3's January 2026 cutoff appears in the template default system message.
- **Native `chatcompletion_v2` request/response field semantics** (for example `mask_sensitive_info`, `input_sensitive`, `base_resp`) are not documented at the surveyed pages, checked 2026-07-19.
