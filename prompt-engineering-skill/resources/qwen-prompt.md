---
family: qwen
scope: prompt
versions:
  - qwen3.6-plus
  - qwen3.6-plus-2026-04-02
  - Qwen/Qwen3.6-35B-A3B
  - Qwen/Qwen3.6-35B-A3B-FP8
retrieved: 2026-04-18
primary_sources:
  - https://huggingface.co/Qwen/Qwen3.6-35B-A3B
  - https://www.alibabacloud.com/blog/qwen3-6-plus-towards-real-world-agents_603005
  - https://www.alibabacloud.com/help/en/model-studio/models
  - https://github.com/QwenLM/Qwen3.6
maturity_note: |
  Qwen3.6 is current as of this retrieval. The closed flagship (qwen3.6-plus)
  launched 2026-04-02; the first Qwen3.6 open-weight variant (Qwen3.6-35B-A3B)
  launched 2026-04-16. User documentation at qwen.readthedocs.io was marked
  "coming soon" on the upstream GitHub as of the retrieval date; some
  fine-grained guidance in this file draws from the Hugging Face model card and
  tokenizer configuration as the available Tier 1 sources.
---

# Qwen — Prompt-Layer Reference

Portable prompting guidance for the Qwen3.6 generation. API-layer detail (chat template tokens, sampling parameters, parser flags, deployment) lives in `qwen-prompt-api.md`.

## 1. Model Selection

The current Qwen lineup on Alibaba Cloud Model Studio spans agentic coding flagships, balanced general-purpose tiers, specialized reasoning and coding variants, and multimodal/omni tiers. The open-weights lineup covers the Qwen3.6-35B-A3B MoE plus several Qwen3.5 sizes. Pick on the task axis, not the brand.

| Target task                                          | Preferred model                                          | Notes                                                                                             |
|------------------------------------------------------|----------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| Agentic coding, repo-level reasoning, UI→code        | `qwen3.6-plus`                                           | Flagship; 1M context; multimodal; integrates with Claude Code, Cline, OpenCode, Qwen Code        |
| Autonomous coding agent on proprietary infra         | `qwen3-coder-plus`                                       | 1M context; coder-specialized; supports context caching                                           |
| Balanced general-purpose, closed                     | `qwen3.5-plus` / `qwen-plus`                             | 1M context; both support thinking mode                                                            |
| Fast / cheap general-purpose                         | `qwen3.5-flash` / `qwen-flash`                           | 1M context; lower cost                                                                            |
| Reasoning-heavy math/code                            | `qwq-plus`                                               | 131K context; reasoning-first variant                                                             |
| Visual reasoning (math, programming from images)     | `qvq-max`                                                | Visual + reasoning combined                                                                       |
| Vision / multimodal understanding                    | `qwen3-vl-plus` / `qwen3-vl-flash`                       | 262K context, thinking mode supported                                                             |
| Text + image + audio + video, speech output          | `qwen3.5-omni-plus` / `qwen3-omni-flash`                 | Omni-modal; preview pricing                                                                       |
| Very long text analysis (10M tokens)                 | `qwen-long`                                              | Mainland China only; summarization / information extraction                                       |
| Open-weights general + agentic coding                | `Qwen/Qwen3.6-35B-A3B`                                   | Apache 2.0; 35B total / 3B active; 262K native, YaRN-extensible to ~1M                           |
| Open-weights lower-cost or smaller footprint         | `Qwen/Qwen3.5-*` (4B / 9B / 27B / 35B-A3B / 122B-A10B)   | Use the smallest that meets quality bar                                                           |

[source: alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

[applies-to: qwen-turbo] `qwen-turbo` is deprecated in favor of `qwen-flash`. Do not start new work on `qwen-turbo`.
[source: alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]

## 2. Prompt Structure Conventions

Qwen uses a ChatML-derived role protocol across both closed-API and open-weights deployments. The template recognizes exactly four roles: `system`, `user`, `assistant`, `tool`. The roles bind to message positions:

- **System messages must come first.** The chat template validates that the system message, if present, appears at the beginning of the conversation.
- **Tool messages are wrapped inside user-message context.** `tool` role output is rendered as `<tool_response>...</tool_response>` embedded in the subsequent user turn, not as a top-level assistant-to-tool exchange.
- **Multimodal content (`image_url`, `video_url`, etc.) is not allowed in system messages.** The template explicitly rejects image/video content in the system role.
- **No default system prompt.** Qwen3.6 does not inject an implicit system prompt when one is omitted; the behavior carries forward from Qwen3.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

Thinking mode is enabled by default in Qwen3.6. When the template is rendered with `add_generation_prompt`, it appends an opening `<think>` block and the model is expected to emit reasoning before the final response.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Migration note: Qwen3 → Qwen3.6

[applies-to: Qwen/Qwen3.6-35B-A3B, qwen3.6-plus]
The in-message soft switches `/think` and `/nothink` that worked in Qwen3 **do not work in Qwen3.6**. They are ignored. To disable thinking, use the API-layer `enable_thinking` parameter (see `qwen-prompt-api.md`). Prompts that rely on `/no_think` as a silencing mechanism must be updated.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
[testable: id=qwen.slash-think-removed.v1, expected=prompt containing "/no_think" produces output with <think>...</think> blocks anyway when enable_thinking is unset]

## 3. Instruction Patterns

### Format-standardization prompts for benchmarking

The Qwen team's own recommendations for benchmarking drive portable prompt shapes that work equally well in production:

- Math problems: append to the user prompt — `Please reason step by step, and put your final answer within \boxed{}.`
- Multiple-choice: append — `Please show your choice in the 'answer' field with only the choice letter, e.g., "answer": "C".`

[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card best-practices, retrieved 2026-04-18]

### Output-length budgeting

The model card recommends allocating 32,768 output tokens for normal queries and 81,920 tokens for highly complex problems (e.g., competition math, programming contests). Under-allocating the output budget with a long thinking block in progress is the single most common failure mode for thinking-mode production use.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

### Thinking preservation across turns

[applies-to: Qwen/Qwen3.6-35B-A3B, qwen3.6-plus]
Qwen3.6 adds a `preserve_thinking` option that, when enabled, retains `<think>` traces from prior turns in the conversation. This is off by default. Turn it on for multi-turn agentic flows where decision-consistency and KV-cache reuse across turns matter; leave it off for stateless Q&A, where retained reasoning inflates input token counts without downstream benefit. The parameter lives in `chat_template_kwargs` for open-weights deployments, or in `extra_body` for Alibaba Cloud Model Studio calls — see `qwen-prompt-api.md` for placement.
[source: www.alibabacloud.com/blog/qwen3-6-plus-towards-real-world-agents_603005, retrieved 2026-04-18]
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

## 4. Context Window Practical Guidance

The closed flagship and most Qwen3.5+ closed variants advertise 1M-token context windows. The open-weights 35B-A3B is 262K native and extends to ~1M with YaRN scaling (configuration-level; see `qwen-prompt-api.md`).
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

Practical constraints:

- Keep effective context ≥ 128K to preserve thinking-mode quality. The model card explicitly flags that reducing context below this threshold (for OOM mitigation) degrades thinking.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
- Static YaRN scaling (used for extending past native 262K) measurably hurts shorter-context performance. Only enable YaRN when long context is actually required; do not leave it on as a default.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
- The 10M-token `qwen-long` tier exists for summarization and extraction workloads; it is a different model family from `qwen3.6-plus` and not a drop-in replacement for agentic use.
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]

## 5. Multimodal Conventions

Qwen3.6-Plus accepts images, video, and screen captures as input. Audio input on the flagship tier is not listed in the 2026-04-02 announcement; audio is surfaced through the `qwen3.5-omni-plus` and `qwen3-omni-flash` tiers.
[source: www.alibabacloud.com/blog/qwen3-6-plus-towards-real-world-agents_603005, retrieved 2026-04-18]
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]

The Qwen3.6-35B-A3B tokenizer exposes image, video, and audio pad tokens, indicating the open-weight base supports all three modalities at the template level; practical audio use with the 35B-A3B base is not documented in the retrieved primary sources.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

Placement rules that matter in the prompt:

- Multimodal content goes in `user` or `assistant` messages, never in `system`. The template rejects images and video in system messages.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]
- For long-video workloads, the `video_preprocessor_config.json` default settings are conservative. The Qwen team recommends raising `longest_edge` to 469,762,048 (≈224K video tokens) when long-video understanding is the primary task.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card best-practices, retrieved 2026-04-18]

## 6. Behavioral Quirks

- **Thinking on by default.** With no parameters set, Qwen3.6 emits a `<think>` block before the response. Callers expecting a terse response without thinking must explicitly disable it at the API layer.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
[testable: id=qwen.thinking-on-by-default.v1, expected=generation from a user-only prompt produces a <think>...</think> block]

- **Greedy decoding is a documented failure mode.** Temperature 0 / pure argmax decoding produces repetition loops. This is called out explicitly in the model card.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
[testable: id=qwen.greedy-repetition.v1, expected=temperature=0 top_p=1.0 top_k=1 on a non-trivial prompt yields repeating n-grams within the first 512 output tokens]

- **Presence-penalty tuning trades repetition for language mixing.** Raising `presence_penalty` above the recommended defaults (see `qwen-prompt-api.md`) reduces repetition but can cause the model to switch languages mid-response, especially on non-English input.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

- **Thinking mode and output budgets interact.** With thinking enabled and no explicit output cap, the model may burn a significant portion of the budget in the `<think>` block and truncate the final response. Budget the two together, not separately. See output-length guidance in Section 3.

## 7. Anti-Patterns

- **Do not rely on `/think` or `/nothink` in-message switches.** They are no-ops on Qwen3.6. Use `enable_thinking` at the API layer.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

- **Do not place images or video in a system message.** The chat template rejects them; the effective behavior depends on the server wrapper but is not reliable.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B/raw/main/tokenizer_config.json, retrieved 2026-04-18]

- **Do not run with temperature = 0 for production.** Greedy decoding produces repetition. Use the mode-specific sampling defaults in `qwen-prompt-api.md`.

- **Do not start new work on `qwen-turbo`.** It is deprecated in favor of `qwen-flash`.
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]

- **Do not treat `qwen-long` as a general-purpose 10M-context model.** It is specialized for long-text summarization and extraction, not agentic coding or multimodal reasoning.
[source: www.alibabacloud.com/help/en/model-studio/models, retrieved 2026-04-18]

- **Do not split a thinking-mode task across two independent calls expecting continuity.** Thinking across turns requires `preserve_thinking=true`; otherwise prior-turn reasoning is dropped.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]

## 8. Gaps

- **User-facing Qwen docs are in transition.** The upstream `qwen.readthedocs.io` was not reachable at retrieval time; the GitHub README marks full user documentation as "coming soon." Some prompting-level guidance that would normally come from a dedicated "Best Practices" docs page is sourced here from the model card instead.
- **Audio input for the open-weights 35B-A3B base is unverified.** Tokenizer support is present but usage guidance is not in the retrieved primary sources. Treat as community-territory until primary docs land.
- **Behavior of thinking mode at > 500K input tokens** is not quantified in the retrieved sources. The ≥128K guidance is a floor for thinking quality; the ceiling behavior is not documented.
- **Cross-language prompting quirks** (beyond the presence-penalty note) are not covered in the retrieved sources. The flagship advertises 201 languages; language-mix behavior and language-specific quirks are not documented in depth.
- **Anthropic-compatible API endpoint behavior** (beyond its existence being confirmed) is not detailed in retrieved Tier 1 sources. Endpoint URL, field mapping, and compatibility caveats are covered as gaps in `qwen-prompt-api.md`.
