---
family: gemini
scope: prompt
versions:
  - gemini-3.5-flash
  - gemini-3.1-flash-lite
  - gemini-3.1-pro-preview
  - gemini-2.5-pro
  - gemini-2.5-flash
  - gemini-2.5-flash-lite
retrieved: 2026-07-19
primary_sources:
  - https://ai.google.dev/gemini-api/docs/models
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.5-flash
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.1-flash-lite
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview
  - https://ai.google.dev/gemini-api/docs/models/gemini-2.5-computer-use-preview-10-2025
  - https://ai.google.dev/gemini-api/docs/interactions-overview
  - https://ai.google.dev/gemini-api/docs/thinking
  - https://ai.google.dev/gemini-api/docs/function-calling
  - https://ai.google.dev/gemini-api/docs/caching
  - https://ai.google.dev/gemini-api/docs/structured-output
  - https://ai.google.dev/gemini-api/docs/deprecations
  - https://blog.google/innovation-and-ai/technology/developers-tools/interactions-api-general-availability/
maturity_note: |
  Gemini 3 is Google's current generation. The Interactions API reached GA on
  2026-06-22 and is the vendor-recommended default for new work; `generateContent`
  is legacy but fully supported. Prompt craft is portable across both surfaces
  (see the surface-selection note in section 2). Gemini 3.5 Flash reached GA on
  2026-05-19 and Gemini 3.1 Flash-Lite reached GA on 2026-05-07; Gemini 3.1
  Pro remains Preview (no GA). The Gemini 3 line uses `thinkingLevel`
  {minimal, low, medium, high} as the reasoning control, replacing the 2.5-era
  `thinkingBudget` — this is the most material migration point for prompt
  engineers coming from 2.5. Deprecation pressure is high: Gemini 2.0 Flash /
  Flash-Lite GA shut down 2026-06-01; Gemini 2.5 Pro / Flash / Flash-Lite GA
  sunset 2026-10-16; gemini-3.1-flash-lite (GA) is scheduled to shut down
  2027-05-07. Per-model pricing is not fully present in the retrieved Tier 1
  sources; consult the current models page at integration time. Implicit-caching
  floors, the gemini-3.5-flash context window, and deprecation dates were
  re-verified 2026-07-18 and carry that date inline; other claims retain their
  2026-06-01 date.
---

# Gemini — Prompt-Layer Reference

Portable prompting guidance for the current Gemini 3 generation (and 2.5 where still relevant). API-layer detail (parameter shapes, caching APIs, SDK-specific field paths) lives in `gemini-prompt-api.md`.

## 1. Model Selection

Pick by task axis. Preview ≠ unstable, but does indicate the surface may still change.

| Target task                                                | Preferred model                       | Status       |
|------------------------------------------------------------|---------------------------------------|--------------|
| Most intelligent; sustained frontier agentic and coding    | `gemini-3.5-flash`                    | GA (released 2026-05-19) |
| Lowest-latency, lowest-cost Gemini 3 text                  | `gemini-3.1-flash-lite`               | GA (since 2026-05-07) |
| Advanced reasoning, long-horizon agentic work              | `gemini-3.1-pro-preview`              | Preview      |
| Reasoning workloads on stable GA                           | `gemini-2.5-pro`                      | GA (sunset 2026-10-16) |
| Price-performance on stable GA                             | `gemini-2.5-flash`                    | GA (sunset 2026-10-16) |
| Fast / cheapest stable GA                                  | `gemini-2.5-flash-lite`               | GA (sunset 2026-10-16) |
| UI automation via screen interaction                       | `gemini-2.5-computer-use-preview-10-2025` | Preview   |

[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-06-01]
[source: ai.google.dev/gemini-api/docs/models/gemini-3.5-flash, retrieved 2026-06-01]
[source: ai.google.dev/gemini-api/docs/models/gemini-3.1-flash-lite, retrieved 2026-06-01]
[source: ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview, retrieved 2026-06-01]

`gemini-3.5-flash` carries a Jan 2025 knowledge cutoff with a latest-update stamp of May 2026.
[source: ai.google.dev/gemini-api/docs/models/gemini-3.5-flash, retrieved 2026-06-01]

A separate endpoint, `gemini-3.1-pro-preview-customtools`, is tuned to prioritize your custom tools over built-in tools.
[source: ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview, retrieved 2026-06-01]

**Computer use is not a 3.x-text-model capability.** Only `gemini-2.5-computer-use-preview-10-2025` (Preview) supports screen interaction. `gemini-3.5-flash`, `gemini-3.1-flash-lite`, and `gemini-3.1-pro-preview` do **not** support computer use.
[source: ai.google.dev/gemini-api/docs/models/gemini-2.5-computer-use-preview-10-2025, retrieved 2026-06-01]
[source: ai.google.dev/gemini-api/docs/models/gemini-3.5-flash, retrieved 2026-06-01]
[source: ai.google.dev/gemini-api/docs/models/gemini-3.1-flash-lite, retrieved 2026-06-01]

### Deprecated / retired

- `gemini-2.0-flash` / `gemini-2.0-flash-lite` GA (and `-001` variants) shut down **2026-06-01**.
- `gemini-2.5-pro` / `gemini-2.5-flash` / `gemini-2.5-flash-lite` GA sunset **2026-10-16**.
- `gemini-3.1-flash-lite` (the **GA** model) has an announced shutdown date of **2027-05-07**. [source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-07-18]
- `gemini-3-pro-preview` (the **original** 3 Pro preview — superseded by `gemini-3.1-pro-preview`) shut down **2026-03-09**.
- `gemini-3.1-flash-lite-preview` shut down **2026-05-25** (superseded by GA `gemini-3.1-flash-lite`).
- `gemini-3-flash-preview` was the preview that `gemini-3.5-flash` replaced.

[source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-06-01]

## 2. Prompt Structure Conventions

### Surface selection: Interactions API vs `generateContent`

Gemini exposes two request surfaces. For new work the **Interactions API** is the vendor-recommended default (GA 2026-06-22, public beta since December 2025); `generateContent` is now legacy but fully supported and still receiving new mainline models. Prompt craft — roles, system instruction, few-shot structure, output-format intent — is portable across both; only the request envelope and field paths differ (see `gemini-prompt-api.md`).
[source: ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]
[source: blog.google/innovation-and-ai/technology/developers-tools/interactions-api-general-availability/, published 2026-06-22, retrieved 2026-07-19]

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

### Thinking defaults vary by tier on Gemini 3

`thinkingLevel` {minimal, low, medium, high} is the recommended reasoning control for Gemini 3 models and onward; it replaces the 2.5-era `thinkingBudget`. Defaults differ by model: `gemini-3.1-pro-preview` defaults to `high`, `gemini-3.5-flash` to `medium`, `gemini-3.1-flash-lite` to `minimal`. For latency- or token-sensitive workloads on Pro or Flash, step down explicitly — do not assume reasoning is off by default the way it is on many OpenAI-compatible stacks.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-06-01]

[applies-to: gemini-3.1-pro-preview] 3.1 Pro cannot set `minimal` and cannot disable thinking.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-06-01]

[applies-to: gemini-3.1-flash-lite] Flash-Lite defaults to `thinkingLevel: "minimal"` — closer to a classical non-thinking chat model. For reasoning tasks on Flash-Lite, raise the level.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-06-01]

### Grounding via Google Search is a built-in tool

Rather than stuffing web search results into the prompt manually, enable the `googleSearch` built-in tool (see `gemini-prompt-api.md`). This is Google's recommended grounding mechanism; the model issues searches, integrates results, and you get grounded responses without a custom tool implementation.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

### Deep Research prompting (hosted agent)

The hosted Gemini Deep Research agent (Interactions API) is a separate surface with its own prompting contract; the full field-observed guidance lives in `resources/deep-research-agents.md`. One prompt-craft point worth surfacing here: on "list all X" prompts the agent tends to name a few items and collapse the rest into an explicit "others / various / unlisted" **bucket**. A single directive eliminated the bucket language entirely in before/after testing — "a bucket is a failed answer; enumerate every item individually; if an item is confirmed to exist but you cannot detail it, still list it and flag that it is undetailed." [field-observed, N=1-2; before/after on the hosted Deep Research agent] Because the agent does not honor inline per-claim markup, put load-bearing shape in structural/suppression form (see `deep-research-agents.md`). The completed report's text is read from the terminal step (`interaction.steps[-1].content[0].text`), not a separate outputs list.

### Thought signatures in multi-turn + tool use

[applies-to: gemini-3.5-flash, gemini-3.1-flash-lite, gemini-3.1-pro-preview]
Gemini 3 returns `thoughtSignature` fields on response parts — opaque encrypted representations of the model's internal reasoning. In multi-turn conversations that include function calling, **pass these signatures back unchanged** alongside the parts that carried them. Concatenating parts, merging parts across signatures, or dropping signatures can drop thought context silently. Google's guidance: "always send the thought_signature back to the model inside its original Part."
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

### Structured output as a replacement for prefill-style steering

Use `responseMimeType: "application/json"` with `responseSchema` to force JSON shape instead of prompting "respond only in JSON." Gemini 3 supports combining structured output with function calling, so the model's fallback when it does not call a function is schema-conforming JSON.
[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]

## 4. Context Window Practical Guidance

`gemini-3.5-flash` is published at **1,048,576 input tokens / 65,536 output tokens**. [source: ai.google.dev/gemini-api/docs/models/gemini-3.5-flash, retrieved 2026-07-18] Context-window sizes for `gemini-3.1-pro-preview` and `gemini-3.1-flash-lite` are still not fully listed in the retrieved Tier 1 sources; Gemini 2.5 Pro and Gemini 2.0 Flash historically advertised 1M to 2M windows and the 3.x line is in the same large-context lineage. Confirm the remaining numbers on the models page before budgeting.

Practical guidance that is sourced:

- **Implicit caching kicks in automatically** above per-model token floors. The floors are per-model, not a clean Flash-vs-Pro split: `gemini-3.5-flash` = **4096**, `gemini-3.1-pro-preview` = **4096**, `gemini-2.5-flash` = **2048**, `gemini-2.5-pro` = **2048**. Long prompts that repeat benefit from caching without code changes — the savings are best-effort, not guaranteed, on implicit caching.
[source: ai.google.dev/gemini-api/docs/caching, retrieved 2026-07-18]

- **For cost-predictable caching** of long prompts, use explicit caching and reference the cache via `cachedContent` — see the API file. Discounts are substantial on Gemini 2.5+ (the caching page references significant savings on cache hit; exact percentage is in Gaps).
[source: ai.google.dev/gemini-api/docs/caching, retrieved 2026-04-18]

## 5. Multimodal Conventions

Gemini is natively multimodal — text, image, video, audio, and PDF inputs are all first-class `parts` entries in the same request. There are no special prompt-time markers to set up multimodal context; just include the content part.

- **Images and video** enter as either `inlineData` (base64 payload) or `fileData` (URI returned by the Files API).
- **PDFs** work the same way — pass as `inlineData` or `fileData`.
- **Screen interaction / computer use** is exposed only via the specialized `gemini-2.5-computer-use-preview-10-2025` model, which takes screenshots as image parts and emits UI action commands. The 3.x text models do not support computer use.

[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]
[source: ai.google.dev/gemini-api/docs/models/gemini-2.5-computer-use-preview-10-2025, retrieved 2026-06-01]

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
[source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-06-01]

## 7. Anti-Patterns

- **Do not use `"assistant"` as a message role.** Gemini uses `"model"`. `"assistant"` is a Claude/OpenAI convention and not portable.
[source: ai.google.dev/gemini-api/docs/function-calling, retrieved 2026-04-18]

- **Do not use `thinkingBudget` for Gemini 3 work.** `thinkingLevel` (minimal/low/medium/high) is the control for Gemini 3 and onward. `thinkingBudget` is retained for the 2.5 series only — "2.5 series models don't support thinkingLevel; use thinkingBudget instead." Do not carry 2.5 budget code forward to 3.x.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-06-01]

- **Do not use full JSON Schema for `responseSchema`.** Gemini supports an OpenAPI-derived subset. `minLength`, `maxLength`, `multipleOf`, complex `pattern` regexes, recursive schemas, and external `$ref` URLs are silently ignored or rejected. Validate against Gemini's supported construct list.
[source: ai.google.dev/gemini-api/docs/structured-output, retrieved 2026-04-18]

- **Do not drop `thoughtSignature` on multi-turn conversations that used function calling.** Signatures authenticate thought continuity; dropping them may downgrade reasoning quality or break signature verification silently.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-04-18]

- **Do not treat "Preview" status as "stable."** Gemini's 3-series models are almost all Preview as of this retrieval. Surface behavior, model IDs, and pricing can change between Preview revisions. Pin a dated model ID when Google publishes one.
[source: ai.google.dev/gemini-api/docs/models, retrieved 2026-04-18]

- **Do not use server-side `googleSearch` results as implicit citation.** The tool returns grounded responses but separate citation/source tracking needs explicit handling; community practice varies and primary-source guidance is not as tight as Anthropic's citations feature.

## 8. Gaps

- **Context-window sizes per current model** are partially closed: `gemini-3.5-flash` is published at 1,048,576 in / 65,536 out (§4). `gemini-3.1-pro-preview` and `gemini-3.1-flash-lite` windows still need a direct consult of the models page at integration time.
- **Exact cache-hit discount percentage** for explicit caching is referenced but not quoted numerically in the retrieved Tier 1 caching excerpt. Earlier community reporting cites 90% on Gemini 2.5+ but this is not replicated verbatim in the current primary page.
- **Vertex AI vs Gemini API prompt-level behavioral differences** (if any) are not covered here; the two platforms share model IDs but have distinct control planes.
- **Live API prompting conventions** (bidirectional audio, VAD, end-of-turn signals) are not covered here; the Live API is a distinct surface with its own prompting discipline.
- **Gemini Deep Research, the Antigravity managed agent, and Computer Use** are documented in `resources/deep-research-agents.md` and `resources/agent-orchestration-surfaces.md`.
- **Cross-language prompting quirks** across Gemini's multilingual support are not covered.
