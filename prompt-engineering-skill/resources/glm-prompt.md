---
family: glm
scope: prompt
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
maturity_note: |
  GLM-5.2 (Z.ai / zai-org) is the current flagship: a 753B-parameter MoE with
  IndexShare sparse attention and a 1M-token context window, released mid-2026
  under an MIT open-weights license. The family exposes a native OpenAI-shaped
  API, an Anthropic-compatible endpoint, and open weights. The reasoning-effort
  control surface is the load-bearing feature: a seven-value ladder that shims
  down onto two real thinking tiers plus a no-think path. Two vendor doc surfaces
  disagree on GLM-5.2's max output (128K on docs.z.ai vs 163,840 on the HF card);
  this file carries both. Several Claude-Code-via-GLM ergonomics (the `[1m]`
  model alias, specific env-var values) are not confirmed on a canonical vendor
  page and are declared as gaps, not asserted.
---

# GLM — Prompt-Layer Reference

Portable prompting guidance for the current GLM generation from Z.ai (`zai-org`). API-call-level detail (raw chat-template tokens, sampling defaults, the thinking-control parameter surface, tool-call wire format, caching mechanics) lives in `glm-prompt-api.md`.

## 1. Model Selection

GLM-5.2 is the flagship, described by the vendor as the "latest flagship model for long-horizon tasks."
[source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-5.2, retrieved 2026-07-19]

| Model ID       | Context | Max output | Thinking mode | Notes |
|----------------|---------|------------|---------------|-------|
| `glm-5.2`      | 1,000,000 | 128K (disputed — see below) | Dynamic | Flagship; 753B MoE, IndexShare sparse attention; MIT open weights |
| `glm-5.1`      | 200K    | 128K       | Dynamic       | Priced identically to `glm-5.2` |
| `glm-5-turbo`  | 200K    | 128K       | Dynamic       | Lower-latency tier |
| `glm-4.7`      | 200K    | 128K       | Forced        | Plus `glm-4.7-flash` (free tier) and `glm-4.7-flashx` |
| `glm-4.6`      | not documented at docs.z.ai/guides/llm, checked 2026-07-19 | — | Dynamic | Prior generation |
| `glm-4.5`      | not documented at docs.z.ai/guides/llm, checked 2026-07-19 | — | Dynamic | Prior generation; `-x`/`-air`/`-airx`/`-flash` variants priced |

[source: docs.z.ai/guides/llm/glm-5.2, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-5.1, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-5-turbo, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-4.7, retrieved 2026-07-19]
[source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

**GLM-5.2 max output is disputed across vendor doc surfaces.** The model-detail page states 128K; the HuggingFace card states 163,840. Carry both; do not silently pick one. [disputed: docs.z.ai/guides/llm/glm-5.2 = 128K (128,000/131,072); huggingface.co/zai-org/GLM-5.2 = 163,840]
[source: docs.z.ai/guides/llm/glm-5.2, retrieved 2026-07-19]
[source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

### Selection rules

- **Long-horizon / large-context agentic work** — `glm-5.2`. The 1M window plus IndexShare sparse attention is the reason to pick it over the 200K-context tiers.
- **General reasoning at lower cost** — `glm-5.1` or `glm-5-turbo` (200K context). `glm-5.1` is priced the same as `glm-5.2`, so the choice between them is context-window driven, not cost driven. [source: docs.z.ai/guides/overview/pricing, retrieved 2026-07-19]
- **Cost-sensitive / high-volume** — `glm-4.7` and its `-flashx` variant, or the free `glm-4.7-flash` / `glm-4.5-flash` tiers. [source: docs.z.ai/guides/overview/pricing, retrieved 2026-07-19]
- **Self-hosted** — `zai-org/GLM-5.2` open weights, MIT license, no regional restriction and no revenue threshold. [source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

### Dynamic-thinking vs forced-thinking

The family splits into two thinking-behavior classes:

- **Dynamic thinking** (model decides whether to think): GLM-5.2, GLM-5.1, GLM-5, GLM-5-Turbo, GLM-5V-Turbo, GLM-4.6, GLM-4.5.
- **Forced thinking** (always thinks): GLM-4.7, GLM-4.5V.

[source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

This matters for prompt design: on dynamic-thinking models a short factual query may return with no thinking block at all, while forced-thinking models always emit one. Do not write prompts that assume a thinking block is always present on the dynamic-thinking tier.

## 2. Prompt Structure Conventions

The API is reachable three ways: a native OpenAI-shaped endpoint, an Anthropic-compatible endpoint (`https://api.z.ai/api/anthropic`), and MIT open weights. Prompts portable from OpenAI Chat Completions or from the Anthropic Messages format generally transfer with a base-URL swap. Endpoint specifics are in `glm-prompt-api.md`.
[source: docs.z.ai/devpack/quick-start, retrieved 2026-07-19]

### Roles

At the open-weights template level, GLM uses four role tokens: `<|system|>`, `<|user|>`, `<|assistant|>`, and `<|observation|>`. The `<|observation|>` role carries tool/function results back into the conversation. Exact tokens and the tool-call wire format are in `glm-prompt-api.md`.
[source: huggingface.co/zai-org/GLM-5.2/blob/main/chat_template.jinja, retrieved 2026-07-19]

### Thinking blocks

Reasoning is wrapped in `<think>` … `</think>` at the template level. On the API the reasoning surfaces as a separate `reasoning_content` stream rather than inline text; see `glm-prompt-api.md`.
[source: huggingface.co/zai-org/GLM-5.2/blob/main/chat_template.jinja, retrieved 2026-07-19]

## 3. Instruction Patterns

### Reasoning effort: a seven-value ladder over two real tiers

GLM-5.2 (and later) accepts a `reasoning_effort` control with seven values — `max` (default), `xhigh`, `high`, `medium`, `low`, `minimal`, `none` — but internally these collapse onto two thinking tiers plus a no-think path. The vendor documents the mapping verbatim:

| Requested `reasoning_effort` | Effective behavior |
|------------------------------|--------------------|
| `none`, `minimal`            | Skip thinking (no reasoning) |
| `low`, `medium`              | Mapped up to **high** |
| `high`                       | High thinking tier |
| `xhigh`                      | Mapped to **max** |
| `max` (default)             | Max thinking tier |

[source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

The practical consequence: requesting `low` or `medium` does **not** buy a cheaper intermediate reasoning budget — it renders as `high`. There are only two real compute tiers (high, max) plus off. `reasoning_effort` is GLM-5.2-and-above only; older models use the `thinking.type` toggle (see `glm-prompt-api.md`).
[source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

This ladder is also the OpenAI-compat reconciliation point: an OpenAI-style caller passing `reasoning_effort` gets the shim above, so the same `low`/`medium`/`high`/`xhigh` names mean different things on GLM than on an OpenAI reasoning model. Treat `reasoning_effort` as family-scoped.

### Turn-level effort management in agent loops

The vendor recommends dynamically balancing effort across a multi-turn agent trajectory: reduce reasoning on quick tool-execution turns, enable deeper thinking on decision turns. This is a documented production pattern, not a benchmark footnote.
[source: docs.z.ai/guides/capabilities/thinking-mode, retrieved 2026-07-19]

### Sampling stance

The vendor's production recommendation is `temperature` default 1.0, `top_p` default 0.95, and to tune only one at a time — "not recommended to adjust both simultaneously."
[source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

## 4. Context Window Practical Guidance

- GLM-5.2 context is **1,000,000 tokens** (1M / 1,048,576). The other current tiers (`glm-5.1`, `glm-5-turbo`, `glm-4.7`) are **200K**.
[source: docs.z.ai/guides/llm/glm-5.2, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-5.1, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-5-turbo, retrieved 2026-07-19]
[source: docs.z.ai/guides/llm/glm-4.7, retrieved 2026-07-19]
- The 1M window is enabled by **IndexShare** sparse attention: the indexer is reused across every four sparse-attention layers, giving roughly a 2.9x FLOP reduction at 1M context.
[source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]
- **Curate the context; do not dump uncurated repositories.** The vendor's thinking-mode / agent guidance is to filter irrelevant material rather than fill the window. A 1M window is capacity, not a mandate to use it.
[source: docs.z.ai/guides/capabilities/thinking-mode, retrieved 2026-07-19]

## 5. Multimodal Conventions

The text tiers (`glm-5.2`, `glm-5.1`, `glm-5-turbo`, `glm-4.7`) are the documented focus of this reference. Vision-capable variants exist in the family lineup — `glm-5v-turbo` (dynamic thinking) and `glm-4.5v` (forced thinking) — but their context windows, pricing, and image-input conventions were not confirmed against a canonical multimodal page in this pass. See Gaps. Do not assume the 1M window or the text pricing carries to the vision variants.
[source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

## 6. Behavioral Quirks

- **Max-output value disagrees across vendor surfaces.** docs.z.ai/guides/llm/glm-5.2 says 128K; the HuggingFace card says 163,840. Aggregators cite 131,072. Budget conservatively to the lower documented figure (128K) unless you have confirmed the higher on your endpoint. [disputed: docs.z.ai = 128K; HF card = 163,840]
[source: docs.z.ai/guides/llm/glm-5.2, retrieved 2026-07-19]
[source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

- **`low`/`medium` reasoning effort are not intermediate tiers.** They map up to `high`. A caller expecting a cheap middle setting gets full high-tier reasoning. [source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

- **Dynamic-thinking models may return no thinking block.** On GLM-5.2/5.1/5/5-Turbo the model decides whether to think; a thinking block is not guaranteed on every turn. Forced-thinking models (GLM-4.7, GLM-4.5V) always emit one. [source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

- **Sampling: tune one of temperature / top_p, not both.** The vendor explicitly advises against adjusting both simultaneously. [source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

- **Reasoning-content order is load-bearing in multi-turn.** Reasoning blocks must be returned complete, unmodified, and in original order; reordering or editing them can degrade output and cache hit rates. Full multi-turn contract (`clear_thinking`, endpoint-dependent defaults) is in `glm-prompt-api.md`. [source: docs.z.ai/guides/capabilities/thinking-mode, retrieved 2026-07-19]

- **Reward-hacking / spec-gaming behavior [community-reported].** Independent launch coverage of GLM-5.2 discusses reward-hacking (specification-gaming) as a topic in the model's agentic behavior. The specifics could not be confirmed against the primary z.ai launch blog (`z.ai/blog/glm-5.2`), which returned an empty body (JS/bot-block) at retrieval on 2026-07-19, so this remains community-reported pending a primary read. [tier: 2, source: interconnects.ai, "GLM-5.2 is the step change for open agents"]

## 7. Anti-Patterns

- **Do not call GLM-5.2's architecture "DeepSeek Sparse Attention" or "DSA."** GLM-5.2 uses **IndexShare** sparse attention. The DSA label is a different vendor's architecture and does not apply here. [source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

- **Do not treat GLM-5.2 max output as a single settled number.** Two vendor surfaces disagree (128K vs 163,840). Assume the lower figure for allocation unless verified. [source: docs.z.ai/guides/llm/glm-5.2, retrieved 2026-07-19] [source: huggingface.co/zai-org/GLM-5.2, retrieved 2026-07-19]

- **Do not request `low`/`medium` expecting a cheaper reasoning tier.** They render as `high`. Use `none`/`minimal` to actually skip thinking. [source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

- **Do not adjust `temperature` and `top_p` together.** Tune one at a time per vendor guidance. [source: docs.z.ai/guides/overview/migrate-to-glm-new, retrieved 2026-07-19]

- **Do not dump an uncurated repository into the 1M window.** Filter to relevant material; capacity is not a substitute for curation. [source: docs.z.ai/guides/capabilities/thinking-mode, retrieved 2026-07-19]

- **Do not reorder or edit prior-turn reasoning blocks when replaying multi-turn context.** Order must match the original or output quality and cache hit rates can degrade. [source: docs.z.ai/guides/capabilities/thinking-mode, retrieved 2026-07-19]

- **Do not assume `reasoning_effort` maps identically to OpenAI's.** GLM shims the ladder onto two tiers; the same value names carry different semantics. [source: docs.z.ai/guides/capabilities/thinking, retrieved 2026-07-19]

## 8. Gaps

- **GLM-5.2 max output** is genuinely disputed between vendor surfaces (128K vs 163,840); this is carried as a disagreement, not resolved.
- **Context windows for `glm-5`, `glm-4.6`, `glm-4.5`** are not documented on their docs.z.ai/guides/llm detail pages as checked 2026-07-19.
- **Vision variants (`glm-5v-turbo`, `glm-4.5v`)** — context window, pricing, and image-input conventions were not confirmed against a canonical multimodal page in this pass.
- **Reward-hacking behavioral specifics** — the primary z.ai launch blog was bot-blocked at retrieval (2026-07-19); the quirk is carried as community-reported only, with no exploit-level detail.
- **Knowledge cutoff dates** for any GLM model are not documented in the retrieved primary sources.
- **The `glm-5.2[1m]` Claude-Code model alias** is not confirmed on a canonical vendor page; see `glm-prompt-api.md` Gaps.
