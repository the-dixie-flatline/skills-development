---
family: gemini
scope: prompt
versions:
  - gemini-3.1-pro-preview
  - gemini-3-flash-preview
  - gemini-3.1-flash-lite-preview
  - gemini-3.1-flash-live-preview
  - gemini-2.5-pro
  - gemini-2.5-flash
  - gemini-2.5-flash-lite
retrieved: 2026-04-18
primary_sources:
  - https://ai.google.dev/gemini-api/docs/models
  - https://ai.google.dev/gemini-api/docs/thinking
  - https://ai.google.dev/gemini-api/docs/function-calling
  - https://ai.google.dev/gemini-api/docs/caching
  - https://ai.google.dev/gemini-api/docs/structured-output
  - https://blog.google/products/gemini/gemini-3-flash/
maturity_note: |
  Gemini 3 is Google's current generation. Gemini 3.1 Pro, Gemini 3 Flash
  (released 2025-12-17), and Gemini 3.1 Flash-Lite are all in Preview;
  Gemini 2.5 Pro, Flash, and Flash-Lite remain stable/GA. A Gemini 3
  generation change replaces `thinkingBudget` with `thinkingLevel` — this is
  the most material migration point for prompt engineers coming from 2.5.
  Context-window sizes and per-model pricing are not fully present in the
  retrieved Tier 1 sources; consult the current models page at integration
  time.
---

# Gemini — Prompt-Layer Reference

Portable prompting guidance for the current Gemini 3 generation (and 2.5 where still relevant). API-layer detail (parameter shapes, caching APIs, SDK-specific field paths) lives in `gemini-prompt-api.md`.

## 1. Model Selection

Pick by task axis. Preview ≠ unstable, but does indicate the surface may still change.

| Target task                                                | Preferred model                       | Status       |
|------------------------------------------------------------|---------------------------------------|--------------|
| Advanced reasoning, long-horizon agentic work              | `gemini-3.1-pro-preview`              | Preview      |
| Balanced quality and speed on Gemini 3                     | `gemini-3-flash-preview`              | Preview (released 2025-12-17) |
| Lowest-latency, lowest-cost Gemini 3 text                  | `gemini-3.1-flash-lite-preview`       | Preview      |
| Real-time audio-to-audio dialogue (Live API)               | `gemini-3.1-flash-live-preview`       | Preview      |
| Reasoning workloads on stable GA                           | `gemini-2.5-pro`                      | GA           |
| Price-performance on stable GA                             | `gemini-2.5-flash`                    | GA           |
| Fast / cheapest stable GA                                  | `gemini-2.5-flash-lite`               | GA           |
| Native image generation                                    | `gemini-3.1-flash-image-preview` (Nano Banana 2), `gemini-3-pro-image-preview` (Nano Banana Pro), `gemini-2.5-flash-image` (Nano Banana) | Mixed GA/Preview |
| Text-to-speech                                             | `gemini-3.1-flash-tts-preview`, `gemini-2.5-flash-preview-tts`, `gemini-2.5-pro-preview-tts`    | Preview      |
| UI automation via screen interaction                       | `gemini-2.5-computer-use-preview-10-2025` | Preview   |
| Multi-step research automation                             | `deep-research-pro-preview-12-2025`   | Preview      |
| Video / music generation                                   | `veo-3.1-generate-preview`, `lyria-3-pro-preview` | Preview |
| Multimodal embeddings                                      | `gemini-embedding-2-preview`, `gemini-embedding-001` | Mixed |

[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]
[source: blog.google/products/gemini/gemini-3-flash, retrieved 2026-04-18]

### Deprecated / retired

- `gemini-3-pro-preview` (the **original** 3 Pro preview — superseded by `gemini-3.1-pro-preview`) was shut down **2026-03-09**.
- `gemini-2.0-flash` and `gemini-2.0-flash-lite` are deprecated; shutdown pending.

[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

## 2. Prompt Structure Conventions

Gemini's request shape differs from Anthropic-style roles in a few ways that trip people migrating prompts:

- **`systemInstruction` is a top-level field** on the request, separate from the turn list. It carries `parts: [{text: ...}]` content blocks, not a flat string. (See `gemini-prompt-api.md` for the exact JSON.)
- **Message roles are `user` and `model`** — not `assistant`. Using `"assistant"` as a role name is a no-op or an error depending on wrapper.
- **Content is parts-based from the start.** Text, images, video frames, audio, PDFs, and function calls all appear as entries in a `parts` array. There is no separate "chat template" to wrangle — multimodal content is first-class.
- **Tool results ride in the `user` role** as parts with `functionResponse`, not in a dedicated `tool` role.

[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

There is **no prefill mechanism** equivalent to Anthropic's pre-populated last assistant message. Force output shape via structured output (`responseSchema`) or explicit instructions.

### No canonical XML-tag convention

Unlike Anthropic, Google's Gemini documentation does not recommend a specific XML tag set for prompt structure. Gemini responds well to clear section labels (e.g. "# Instructions" / "# Context" markdown headings, or custom tags you define), but do not expect the `<example>`/`<document>` conventions portable from Claude to carry the same weight.

[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

## 3. Instruction Patterns

### Thinking defaults are high on Gemini 3

Gemini 3 flagships and Flash default to `thinkingLevel: "high"`. If the workload is latency-sensitive or token-sensitive, step down to `"medium"`, `"low"`, or `"minimal"` explicitly — do not assume reasoning is off by default the way it is on many OpenAI-compatible stacks.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

[applies-to: gemini-3.1-flash-lite-preview] Flash-Lite defaults to `thinkingLevel: "minimal"` — closer to a classical non-thinking chat model. For reasoning tasks on Flash-Lite, raise the level.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

### Grounding via Google Search is a built-in tool

Rather than stuffing web search results into the prompt manually, enable the `googleSearch` built-in tool (see `gemini-prompt-api.md`). This is Google's recommended grounding mechanism; the model issues searches, integrates results, and you get grounded responses without a custom tool implementation.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

### Thought signatures in multi-turn + tool use

[applies-to: gemini-3.1-pro-preview, gemini-3-flash-preview, gemini-3.1-flash-lite-preview]
Gemini 3 returns `thoughtSignature` fields on response parts — opaque encrypted representations of the model's internal reasoning. In multi-turn conversations that include function calling, **pass these signatures back unchanged** alongside the parts that carried them. Concatenating parts, merging parts across signatures, or dropping signatures can drop thought context silently. Google's guidance: "always send the thought_signature back to the model inside its original Part."
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

### Structured output as a replacement for prefill-style steering

Use `responseMimeType: "application/json"` with `responseSchema` to force JSON shape instead of prompting "respond only in JSON." Gemini 3 supports combining structured output with function calling, so the model's fallback when it does not call a function is schema-conforming JSON.
[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]

## 4. Context Window Practical Guidance

Context-window sizes per-model are not fully listed in the retrieved Tier 1 sources. Gemini 2.5 Pro and Gemini 2.0 Flash have historically advertised 1M to 2M windows; Gemini 3 Pro and 3 Flash are large-context models in the same generational lineage. Confirm the current number on the models page before budgeting.

Practical guidance that is sourced:

- **Implicit caching kicks in automatically** above per-model token floors (1024 tokens for Flash tier, 4096 for Pro tier). Long prompts that repeat benefit from caching without code changes — the savings are best-effort, not guaranteed, on implicit caching.
[source: ai.google.dev/gemini-api/docs/caching, retrieved 2026-04-18]

- **For cost-predictable caching** of long prompts, use explicit caching and reference the cache via `cachedContent` — see the API file. Discounts are substantial on Gemini 2.5+ (the caching page references significant savings on cache hit; exact percentage is in Gaps).
[source: ai.google.dev/gemini-api/docs/caching, retrieved 2026-04-18]

## 5. Multimodal Conventions

Gemini is natively multimodal — text, image, video, audio, and PDF inputs are all first-class `parts` entries in the same request. There are no special prompt-time markers to set up multimodal context; just include the content part.

- **Images and video** enter as either `inlineData` (base64 payload) or `fileData` (URI returned by the Files API).
- **PDFs** work the same way — pass as `inlineData` or `fileData`.
- **Audio** on the Live API variant (`gemini-3.1-flash-live-preview`) supports streaming bi-directional audio for real-time dialogue.
- **Screen interaction / computer use** is exposed via the specialized `gemini-2.5-computer-use-preview-10-2025` model, which takes screenshots as image parts and emits UI action commands.

[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

A common convention: place multimodal inputs **before** the text instructions that reference them. Gemini attends to ordering within a `user` turn's parts array.

## 6. Behavioral Quirks

- **Thinking is on by default on Gemini 3 flagships.** Unlike Claude Opus 4.7 (adaptive thinking opt-in) or most OpenAI-compatible stacks (no thinking by default), Gemini 3 defaults to thinking. If a workload is simple enough that thinking adds latency without benefit, explicitly set `thinkingLevel: "minimal"`.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

- **Unsupported JSON Schema fields are silently ignored.** If your schema uses `minLength`, `pattern` with lookaheads, or other constructs outside Gemini's supported subset, the model will not error — it will emit output that ignores those constraints. Schema validation on your side is required.
[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]

- **Function-call IDs are new in Gemini 3 responses.** Every function call carries a unique `id`. Tool-result returns **must** echo the `id` — parallel tool calls that come back out-of-order rely on `id` for matching. Code written for Gemini 2.5 that ignores `id` will still work, but Gemini 3 + parallel calls requires it.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

- **Implicit vs explicit caching is not interchangeable.** Implicit caching is automatic but offers no cost guarantee (you pay full price if the cache happens to miss). Explicit caching is opt-in and gives predictable savings but requires managing cache objects.
[source: ai.google.dev/gemini-api/docs/caching, retrieved 2026-04-18]

- **`gemini-3.1-pro-preview` and `gemini-3-pro-preview` are different models.** The original `gemini-3-pro-preview` was shut down 2026-03-09 and superseded by `gemini-3.1-pro-preview`. Don't hard-code the old ID.
[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

## 7. Anti-Patterns

- **Do not use `"assistant"` as a message role.** Gemini uses `"model"`. `"assistant"` is a Claude/OpenAI convention and not portable.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

- **Do not rely on `thinkingBudget` for Gemini 3 work.** `thinkingBudget` is accepted on Gemini 3 for backwards compatibility but the contract is `thinkingLevel` (minimal/low/medium/high). Mixing budgets on 3.1 Pro may cause unexpected performance.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

- **Do not use full JSON Schema for `responseSchema`.** Gemini supports an OpenAPI-derived subset. `minLength`, `maxLength`, `multipleOf`, complex `pattern` regexes, recursive schemas, and external `$ref` URLs are silently ignored or rejected. Validate against Gemini's supported construct list.
[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]

- **Do not drop `thoughtSignature` on multi-turn conversations that used function calling.** Signatures authenticate thought continuity; dropping them may downgrade reasoning quality or break signature verification silently.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

- **Do not treat "Preview" status as "stable."** Gemini's 3-series models are almost all Preview as of this retrieval. Surface behavior, model IDs, and pricing can change between Preview revisions. Pin a dated model ID when Google publishes one.
[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

- **Do not use server-side `googleSearch` results as implicit citation.** The tool returns grounded responses but separate citation/source tracking needs explicit handling; community practice varies and primary-source guidance is not as tight as Anthropic's citations feature.

## 8. Gaps

- **Context-window sizes per current model** are not listed in the retrieved primary source excerpts. Legacy pages document the 2.x series; authoritative Gemini 3 context-window numbers need a separate fetch or direct consult of the models page at integration time.
- **Exact cache-hit discount percentage** for explicit caching is referenced but not quoted numerically in the retrieved Tier 1 caching excerpt. Earlier community reporting cites 90% on Gemini 2.5+ but this is not replicated verbatim in the current primary page.
- **Vertex AI vs Gemini API prompt-level behavioral differences** (if any) are not covered here; the two platforms share model IDs but have distinct control planes.
- **Live API prompting conventions** (bidirectional audio, VAD, end-of-turn signals) are not covered here; the Live API is a distinct surface with its own prompting discipline.
- **Gemini 3 Deep Think mode** is referenced in public blog posts but not documented in-depth in the retrieved API excerpts.
- **Cross-language prompting quirks** across Gemini's multilingual support are not covered.
