---
family: kimi
scope: prompt
versions:
  - kimi-k3
  - kimi-k2.6
  - kimi-k2.7-code
  - kimi-k2.7-code-highspeed
retrieved: 2026-08-09
primary_sources:
  - https://platform.kimi.ai/docs/guide/kimi-k3-quickstart
  - https://platform.kimi.ai/docs/api/models-overview
  - https://platform.kimi.ai/docs/guide/use-reasoning-effort
  - https://platform.kimi.ai/docs/guide/response_format
  - https://platform.kimi.ai/docs/guide/use-dynamic-tool-loading
  - https://platform.kimi.ai/docs/guide/use-official-tools
  - https://platform.kimi.ai/docs/guide/kimi-k3-tool-calling-best-practice
  - https://platform.kimi.ai/docs/guide/claude-code-kimi
  - https://platform.kimi.ai/docs/pricing/chat-k3
  - https://platform.kimi.ai/docs/models
  - https://www.kimi.com/blog/kimi-k3
  - https://huggingface.co/moonshotai/Kimi-K3
  - https://github.com/MoonshotAI/Kimi-K3
maturity_note: |
  Kimi K3 (`kimi-k3`, Moonshot AI) launched ~2026-07-17. A 2026-08-09
  re-verification pass superseded the launch-week snapshot: `reasoning_effort`
  now accepts low / high / max (default max) — no longer max-only — and the
  open weights shipped on schedule at `moonshotai/Kimi-K3` under the custom
  "Kimi K3 License" (MIT-style with revenue/MAU thresholds), so the
  open-weights gaps are narrowed rather than open. Vendor blog now documents
  two behavioral limitations (excessive proactiveness; sensitivity to broken
  thinking-history replay) and a self-assessed UX gap versus Claude Fable 5 /
  GPT-5.6 Sol. The vendor's own web-search caution is now internally
  inconsistent across doc pages and carried here as disputed. API-layer detail
  (message shapes, sampling params, tool protocol, structured output) lives
  in `kimi-prompt-api.md`.
---

# Kimi (Moonshot AI) — Prompt-Layer Reference

Portable prompting guidance for the current Kimi generation. API-call-level detail (message shapes, sampling parameters, tool protocol, structured-output wire format, deployment env vars) lives in `kimi-prompt-api.md`.

Vendor is Moonshot AI; the platform documentation is at `platform.kimi.ai` and the API host is `api.moonshot.ai`. A separate membership-billed coding product, "Kimi Code" (`api.kimi.com/coding`), is documented elsewhere and is not the platform API surface — see §1.

## 1. Model Selection

Current flagship is **`kimi-k3`**: 2.8 trillion total / 104B activated parameters (MoE: 93 layers, 896 experts with 16 selected + 2 shared per token, 160K vocabulary), using Kimi Delta Attention (KDA), a hybrid linear attention mechanism, with native MXFP4 weights / MXFP8 activations via quantization-aware training and a MoonViT-V2 401M vision encoder. Context window is 1,048,576 tokens (1M).
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]
[source: github.com/MoonshotAI/Kimi-K3, retrieved 2026-08-09]

Access requirement: `kimi-k3` is a flagship model unlocked only after a minimum $1 top-up; cumulative top-up determines the rate-limit tier.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-08-09]

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
- **Self-hosting K3 open weights** — now possible: weights shipped ~2026-07-27 at `moonshotai/Kimi-K3` under the Kimi K3 License. Hardware floor is high (vendor recipe: at least 8x GB300, or MI355X/MI350X on ROCm; multi-node for production) — see `kimi-prompt-api.md` §8.
[source: huggingface.co/moonshotai/Kimi-K3, retrieved 2026-08-09]

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

K3 weights are now published (`moonshotai/Kimi-K3`, ~2026-07-27), but this pass did not verify the shipped chat template, special tokens, or tokenizer config against the repo contents (§8). Work through the hosted API message structure above, or read the repo's template files directly; do not hand-assemble a token stream from memory.
[source: huggingface.co/moonshotai/Kimi-K3, retrieved 2026-08-09]

## 3. Instruction Patterns

### Reasoning is always on

K3 always reasons and may return a separate `reasoning_content` field alongside `content`. Reasoning depth is governed by the top-level `reasoning_effort` parameter (API detail in `kimi-prompt-api.md`), which now accepts **`low`, `high`, and `max`** with `max` as the default — the launch-week max-only restriction is lifted (the promised "more levels" shipped).
[source: platform.kimi.ai/docs/guide/use-reasoning-effort, retrieved 2026-08-09]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-08-09]

**Decide the effort level before the session starts.** Switching `reasoning_effort` mid-session invalidates prefix-cache hits; the vendor's own recommendation is to pick the level up front and hold it for the conversation.
[source: platform.kimi.ai/docs/api/models-overview, retrieved 2026-08-09]
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

- **`reasoning_effort` now has three levels.** `low`, `high`, and `max` are accepted, default `max` (as of 2026-08-09; supersedes the launch-week max-only state). Off-ladder values are not documented — do not send them. Mid-session switches bust the prefix cache (§3).
[source: platform.kimi.ai/docs/guide/use-reasoning-effort, retrieved 2026-08-09]

- **Excessive proactiveness (vendor-documented).** K3's training emphasizes long-horizon, challenging tasks; on ambiguous tasks it "may make unexpected decisions on the user's behalf." Scope ambiguous requests explicitly and state what is out of bounds.
[source: kimi.com/blog/kimi-k3, retrieved 2026-08-09]

- **Sensitive to broken thinking-history replay (vendor-documented).** The vendor flags degraded behavior when harnesses drop or mangle reasoning history across turns or switch models mid-session — consistent with the load-bearing full-replay contract in §2. Keep the assistant message intact and avoid mid-session model swaps.
[source: kimi.com/blog/kimi-k3, retrieved 2026-08-09]

- **Vendor self-assessment: UX gap vs frontier peers.** Despite benchmark scores, the vendor states K3 "exhibits a noticeable gap in user experience compared with Claude Fable 5 and GPT 5.6 Sol." Community sentiment (partially verified — the underlying threads are automation-blocked; corroborated only via search snippets) reads the same: faster and much cheaper, weaker on the hardest long-horizon agentic tasks. [community-reported]
[source: kimi.com/blog/kimi-k3, retrieved 2026-08-09]

- **Fixed sampling parameters.** `temperature`, `top_p`, `n`, `presence_penalty`, and `frequency_penalty` are fixed and cannot be modified; passing a non-default value returns an error. Omit them from requests. Full values and details are in `kimi-prompt-api.md`.
[source: platform.kimi.ai/docs/api/models-overview, retrieved 2026-07-19]
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-07-19]

- **The `web-search` official tool's status is vendor-inconsistent.** [disputed: the kimi-k3-quickstart "Important limits" section still states web search "is being updated and is not recommended for production workflows in the near term," while the pricing page flags that caution as outdated documentation and the official-tools guide ships a working web-search example with no caution attached] Both positions are live vendor pages as of 2026-08-09. Treat web-search as usable-but-unsettled; do not build production dependence on it until the docs converge.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-08-09]
[source: platform.kimi.ai/docs/pricing/chat-k3, retrieved 2026-08-09]

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

- **Do not build production dependence on the `web-search` official tool yet.** Vendor pages disagree on whether the launch-week caution still applies (§6); until they converge, treat it as usable for experimentation only.
[source: platform.kimi.ai/docs/guide/kimi-k3-quickstart, retrieved 2026-08-09]

- **Do not port K3's `reasoning_effort` to `kimi-k2.7-code`.** K2.7-code uses its own `thinking` object; the dynamic-tool-loading construct also fails on non-K3 models (§ tool use in `kimi-prompt-api.md`).
[source: platform.kimi.ai/docs/models, retrieved 2026-07-19]

- **Do not treat `api.kimi.com/coding` as a platform API variant.** It is the separate Kimi Code product.
[source: www.kimi.com/code/docs, retrieved 2026-07-19]

## 8. Gaps

- **RESOLVED (2026-08-09): open weights released.** `moonshotai/Kimi-K3` (HF) and `MoonshotAI/Kimi-K3` (GitHub) shipped ~2026-07-27 under the **Kimi K3 License** — an MIT-style permissive grant with a $20M/12-month Model-as-a-Service revenue threshold requiring a separate agreement, and a >100M-MAU / $20M-monthly-revenue attribution clause. Not plain MIT and not the previously-rumored "Modified MIT" label; read the license file for the thresholds.
[source: huggingface.co/moonshotai/Kimi-K3/blob/main/LICENSE, retrieved 2026-08-09]
- **Shipped chat template / special tokens not verified.** The weights repo exists but this pass did not read its tokenizer/template files; thinking delimiters and tool-call wire format at the token level remain unconfirmed here.
- **Context-window degradation points and ordering effects** within the 1M window are not documented.
- **Refusal patterns and verbosity defaults** for K3 are only partially documented — the vendor blog's two behavioral limitations (§6) are the extent of it.
- **Few-shot / CoT prompting behavior** specific to K3 (beyond "reasoning is always on") is not documented.
- **Audio input support** is not documented (image and video are — §5).
- **Rate-limit and 429 semantics** are still not documented on the pricing/quickstart pages (re-checked 2026-08-09; API-layer gap; see `kimi-prompt-api.md`).
- **Kimi Code product docs unreadable by automation.** `www.kimi.com/code/docs` renders as an empty client-side SPA shell on automated fetch (HTTP 200, no content); Kimi Code claims could not be re-verified this pass.
