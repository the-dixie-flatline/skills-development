---
family: kimi
scope: prompt
versions:
  - kimi-k3
  - kimi-k2.6
  - kimi-k2.7-code
  - kimi-k2.7-code-highspeed
retrieved: 2026-07-19
primary_sources:
  - https://platform.kimi.ai/docs/guide/kimi-k3-quickstart
  - https://platform.kimi.ai/docs/api/models-overview
  - https://platform.kimi.ai/docs/guide/use-thinking-effort
  - https://platform.kimi.ai/docs/guide/response_format
  - https://platform.kimi.ai/docs/guide/use-dynamic-tool-loading
  - https://platform.kimi.ai/docs/guide/use-official-tools
  - https://platform.kimi.ai/docs/guide/kimi-k3-tool-calling-best-practice
  - https://platform.kimi.ai/docs/guide/claude-code-kimi
  - https://platform.kimi.ai/docs/pricing/chat-k3
  - https://platform.kimi.ai/docs/models
  - https://www.kimi.com/blog/kimi-k3
  - https://huggingface.co/moonshotai
  - https://github.com/moonshotai
maturity_note: |
  Kimi K3 (`kimi-k3`, Moonshot AI) launched ~2026-07-17 and is three days
  old at this retrieval date. Content is launch-week and volatile: every
  vendor "currently" / "coming soon" qualifier is preserved and dated
  2026-07-19. Reasoning is always on and `reasoning_effort` accepts only
  `max` today. Open weights are announced for release by 2026-07-27 but are
  NOT yet released, so all open-weights guidance (chat template, special
  tokens, license) is a declared gap, not an omission. API-layer detail
  (message shapes, sampling params, tool protocol, structured output) lives
  in `kimi-prompt-api.md`.
---

# Kimi (Moonshot AI) — Prompt-Layer Reference

Portable prompting guidance for the current Kimi generation. API-call-level detail (message shapes, sampling parameters, tool protocol, structured-output wire format, deployment env vars) lives in `kimi-prompt-api.md`.

Vendor is Moonshot AI; the platform documentation is at `platform.kimi.ai` and the API host is `api.moonshot.ai`. A separate membership-billed coding product, "Kimi Code" (`api.kimi.com/coding`), is documented elsewhere and is not the platform API surface — see §1.

## 1. Model Selection

Current flagship is **`kimi-k3`**: 2.8 trillion parameters, using Kimi Delta Attention (KDA), a hybrid linear attention mechanism. The vendor frames it as its "first open-source model to reach 2.8 trillion parameters" / "3-trillion-parameter class." Context window is 1,048,576 tokens (1M).
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/pricing/chat-k3, retrieved 2026-07-19]

K3 reasons on every request (see §3); there is no non-thinking variant of K3.

### Active K2.x models (post-K3 launch)

| Model ID | Notes |
|---|---|
| `kimi-k2.6` | 256K context; optional thinking. General-purpose predecessor still available. |
| `kimi-k2.7-code` | Coding model; retains its own always-on thinking contract (`{"type":"enabled","keep":"all"}`), distinct from K3's `reasoning_effort`. |
| `kimi-k2.7-code-highspeed` | High-throughput coding variant: ~180 tok/s short-context, up to ~260 tok/s (this figure is specific to the highspeed variant). |

[source: platform.kimi.ai/docs/models, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/api/models-overview, retrieved 2026-07-19]

### Selection rules

- **Long-horizon reasoning, deep knowledge work, agentic coding over long context** — `kimi-k3`. Always-on reasoning, 1M context.
- **General chat where a smaller context and optional thinking suffice** — `kimi-k2.6`.
- **Coding with a distinct thinking contract or throughput-sensitive coding** — `kimi-k2.7-code` / `kimi-k2.7-code-highspeed`. Do not port K3's `reasoning_effort` semantics to these; K2.7-code uses its own thinking object.
- **Self-hosting K3 open weights** — not possible at this retrieval date. Weights are announced for release by 2026-07-27 but not yet published (§8).

### Kimi Code is a separate product

`api.kimi.com/coding` belongs to a distinct, membership-billed product ("Kimi Code"), documented at `www.kimi.com/code/docs`, not to the platform API. It exposes its own base URLs and a model id `k3`. Do not conflate it with the platform `api.moonshot.ai` surface; the two are billed and documented separately.
[source: www.kimi.com/code/docs, retrieved 2026-07-19]

## 2. Prompt Structure Conventions

Messages are OpenAI-shaped: roles `system`, `user`, `assistant`, `tool`.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

### Multi-turn replay contract (load-bearing)

For multi-turn conversations and tool calls, K3 requires the complete assistant message returned by the API to be passed back into `messages` as-is, including `reasoning_content` and `tool_calls`. The vendor states this verbatim. Omitting parts of the returned assistant message risks errors or quality degradation.
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

This places K3 in the "preserve the full assistant turn" camp (like Claude and OpenAI reasoning models), not the "strip reasoning before replay" camp. A pipeline that keeps only `content` across turns will violate the contract. API-layer round-trip mechanics are in `kimi-prompt-api.md`.

### Open-weights chat template

No canonical chat template, special tokens, or tokenizer config is published for K3 at this retrieval date (weights unreleased — §8). There is no vendor encoder to cite yet. Work through the hosted API message structure above; do not hand-assemble a token stream from memory.
[source: huggingface.co/moonshotai, retrieved 2026-07-19]
[source: github.com/moonshotai, retrieved 2026-07-19]

## 3. Instruction Patterns

### Reasoning is always on

K3 always reasons and may return a separate `reasoning_content` field alongside `content`. Reasoning depth is governed by the top-level `reasoning_effort` parameter (API detail in `kimi-prompt-api.md`), which currently accepts only `max`; the vendor states more reasoning-effort levels are "coming soon."
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]
Verified 2026-07-19: kimi-k3 with `reasoning_effort` omitted returned non-empty reasoning content even on a trivial prompt ("What is 2+2? one word") — 5/5 reasoning present (23–75 reasoning tokens, via OpenRouter/Moonshot). Confirms K3 reasons on every request; there is no non-thinking variant.

Because reasoning is unconditional, do not prompt for explicit step-by-step chain-of-thought as a way to "turn on" deliberation; the model already reasons internally and surfaces it in `reasoning_content`. Prompts that demand the model print its scratch reasoning into `content` fight the model's own separation of `reasoning_content` from `content`.

### K2.x `thinking` parameter does not apply to K3

The K2.x-era `thinking` parameter must not be used on K3. K3's reasoning is controlled exclusively by `reasoning_effort`. (K2.7-code retains its own `thinking` object; that is a different model contract — §1.)
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

### Few-shot / output-format guidance

Vendor guidance on few-shot prompting and output-format directives specific to K3 is thin in the retrieved sources. For strict machine-readable output, prefer the structured-output path (`response_format` with a JSON Schema) over prose instructions; that path is documented and enforced (see `kimi-prompt-api.md`). Prompt-layer few-shot behavior beyond that is not separately documented (§8).
[source: platform.kimi.ai/docs/guide/response_format, retrieved 2026-07-19]

## 4. Context Window Practical Guidance

- Context window is **1,048,576 tokens (1M)**. Pricing is flat pay-as-you-go with no tiering by context length.
[source: platform.kimi.ai/docs/pricing/chat-k3, retrieved 2026-07-19]
- `max_completion_tokens` defaults to 131072 and can be set up to 1048576.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]
- Output token accounting includes reasoning-trace tokens (reasoning is always on), so budget the completion window with the reasoning trace in mind.
[source: platform.kimi.ai/docs/pricing/chat-k3, retrieved 2026-07-19]

Where quality degrades within the 1M window, and any position/ordering effects, are not documented (§8). Do not assert a degradation point from memory.

## 5. Multimodal Conventions

K3 accepts image and video input through the hosted API.

- **Images**: the message `content` must be an **array of objects**, not a plain string. Image references use `image_url.url` set to either a base64 data URI (`data:image/...;base64,...`) or an uploaded-file reference (`ms://<file-id>`). **Public HTTP/HTTPS image URLs are rejected** — you must base64-encode or upload first.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]
- **Video**: upload the file (`purpose="video"`), then reference it as `ms://{video.id}`.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

Audio input is not documented in the retrieved sources.

## 6. Behavioral Quirks

- **Reasoning is always on and separated.** K3 emits its chain-of-thought in `reasoning_content`, distinct from `content`. Consumers that read only `content` still get the answer; consumers that log the whole message object see the reasoning trace as well.
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]

- **`reasoning_effort` is single-valued today.** Only `max` is accepted as of 2026-07-19; the vendor says more levels are "coming soon." Do not hard-code an assumption that lower-effort values will be rejected forever, but do not send them today.
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]

- **Fixed sampling parameters.** `temperature`, `top_p`, `n`, `presence_penalty`, and `frequency_penalty` are fixed and cannot be modified; passing a non-default value returns an error. Omit them from requests. Full values and details are in `kimi-prompt-api.md`.
[source: platform.kimi.ai/docs/api/models-overview, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

- **The `web-search` official tool is not recommended right now.** The vendor states verbatim: "The web search (web_search) is currently being updated. We do not recommend using this functionality in the near term."
[source: platform.kimi.ai/docs/guide/use-official-tools, retrieved 2026-07-19]

- **Public image URLs fail.** See §5; a common OpenAI-style prompt that passes an `image_url` pointing at a public URL will be rejected. Base64 or upload first.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

Refusal patterns, verbosity defaults, and hedging tendencies are not documented for K3 at this retrieval date (§8).

## 7. Anti-Patterns

- **Do not send the K2.x `thinking` parameter to K3.** K3 reasoning is controlled by `reasoning_effort` only.
[source: platform.kimi.ai/docs/guide/use-thinking-effort, retrieved 2026-07-19]

- **Do not pass the fixed sampling parameters.** `temperature`, `top_p`, `n`, `presence_penalty`, `frequency_penalty` cannot be modified; passing a non-default value returns an error. Omit them.
[source: platform.kimi.ai/docs/api/models-overview, retrieved 2026-07-19]

- **Do not keep only `content` across turns.** The complete assistant message (including `reasoning_content` and `tool_calls`) must be replayed as-is, or you risk errors and quality degradation.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

- **Do not parse the whole response object as your structured payload.** With `response_format` JSON Schema, parse only `choices[0].message.content`; `reasoning_content` sits outside schema scope, and deserializing the entire response object will not match your schema (see `kimi-prompt-api.md`).
[source: platform.kimi.ai/docs/guide/response_format, retrieved 2026-07-19]

- **Do not pass a public image URL.** K3 rejects public HTTP/HTTPS image URLs; use a base64 data URI or an `ms://` upload reference.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

- **Do not rely on the `web-search` official tool right now.** The vendor advises against using it in the near term while it is being updated.
[source: platform.kimi.ai/docs/guide/use-official-tools, retrieved 2026-07-19]

- **Do not port K3's `reasoning_effort` to `kimi-k2.7-code`.** K2.7-code uses its own `thinking` object; the dynamic-tool-loading construct also fails on non-K3 models (§ tool use in `kimi-prompt-api.md`).
[source: platform.kimi.ai/docs/models, retrieved 2026-07-19]

- **Do not treat `api.kimi.com/coding` as a platform API variant.** It is the separate Kimi Code product.
[source: www.kimi.com/code/docs, retrieved 2026-07-19]

## 8. Gaps

- **Open weights not released.** No public repo, license file, chat template, special tokens, thinking delimiters, or tool-call wire format on Moonshot/Kimi channels, HF `moonshotai`, or GitHub `moonshotai` as of 2026-07-19. Vendor target: "The full model weights will be released by July 27, 2026." A technical report is also pending. Until then, no canonical encoder exists to cite.
[source: www.kimi.com/blog/kimi-k3, retrieved 2026-07-19]
[source: huggingface.co/moonshotai, retrieved 2026-07-19]
[source: github.com/moonshotai, retrieved 2026-07-19]

- **K3 weights license unknown.** "Modified MIT" for K3 appears only in third-party aggregators, not on any vendor page; the K3 launch blog carries no license statement. The K2 lineage was modified-MIT, but K3 is unconfirmed. Not stated as fact here until the license file ships. [community-reported]
[source: www.kimi.com/blog/kimi-k3, retrieved 2026-07-19]

- **Context-window degradation points and ordering effects** within the 1M window are not documented.
- **Refusal patterns, verbosity defaults, hedging tendencies** for K3 are not documented.
- **Few-shot / CoT prompting behavior** specific to K3 (beyond "reasoning is always on") is not documented.
- **Audio input support** is not documented (image and video are — §5).
- **Rate-limit and 429 semantics** are not documented on the pricing/quickstart pages checked 2026-07-19 (API-layer gap; see `kimi-prompt-api.md`).
