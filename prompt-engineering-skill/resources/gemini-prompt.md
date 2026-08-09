---
family: gemini
scope: prompt
versions:
  - gemini-3.6-flash
  - gemini-3.5-flash
  - gemini-3.5-flash-lite
  - gemini-3.1-flash-lite
  - gemini-3.1-pro-preview
  - gemini-2.5-pro
  - gemini-2.5-flash
  - gemini-2.5-flash-lite
retrieved: 2026-08-09
primary_sources:
  - https://ai.google.dev/gemini-api/docs/models
  - https://ai.google.dev/gemini-api/docs/models/gemini-3.6-flash
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
  - https://ai.google.dev/gemini-api/docs/changelog
  - https://ai.google.dev/gemini-api/docs/pricing
  - https://deepmind.google/models/model-cards/gemini-3-6-flash/
  - https://blog.google/innovation-and-ai/technology/developers-tools/interactions-api-general-availability/
  - https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/
maturity_note: |
  Gemini 3 is Google's current generation. A 2026-08-09 pass added Gemini 3.6
  Flash (GA 2026-07-21) — the new Flash-tier lead, positioned on output-token
  efficiency and cheaper output than 3.5 Flash — and Gemini 3.5 Flash-Lite
  (GA same day). Computer use is now a built-in client-side tool on Flash-tier
  3.x models (Preview), reversing this file's earlier "not a 3.x-text-model
  capability" claim. The 2026-10-16 sunset date for the 2.5-GA line is GONE
  from the live deprecations page (all three now read "No shutdown date
  announced"). Gemini 3.5 Pro is partner-only testing; 3.1 Pro Preview remains
  the only Pro-tier offering. The Interactions API (GA 2026-06-22) is the
  vendor-recommended default; `generateContent` is legacy but fully supported.
  `thinkingLevel` {minimal, low, medium, high} remains the Gemini 3 reasoning
  control. Claims not re-touched this pass retain their earlier inline dates.
---

# Gemini — Prompt-Layer Reference

Portable prompting guidance for the current Gemini 3 generation (and 2.5 where still relevant). API-layer detail (parameter shapes, caching APIs, SDK-specific field paths) lives in `gemini-prompt-api.md`.

**Not this file: Gemini Notebook** (formerly NotebookLM). That is a source-grounded retrieval product, not a model surface — it exposes no sampling, thinking level, system prompt, or chat template, and its chat path retrieves from an index rather than filling a context window. Nothing in this file or `gemini-prompt-api.md` applies to it. Load `gemini-notebook-surface.md` instead, and load these two alongside it only when the task also touches the Gemini API.

## 1. Model Selection

Pick by task axis. Preview ≠ unstable, but does indicate the surface may still change.

| Target task                                                | Preferred model                       | Status       |
|------------------------------------------------------------|---------------------------------------|--------------|
| Flash-tier lead: fast agentic work, fewer output tokens/steps per task, cheaper output | `gemini-3.6-flash`  | GA (released 2026-07-21) |
| Prior Flash-tier lead, still GA                            | `gemini-3.5-flash`                    | GA (released 2026-05-19) |
| Fastest / cheapest current Gemini 3 text (claimed 350 output tok/s) | `gemini-3.5-flash-lite`      | GA (released 2026-07-21) |
| Lowest-cost prior-gen Lite (replacement: `gemini-3.5-flash-lite`) | `gemini-3.1-flash-lite`        | GA (shutdown 2027-05-07) |
| Advanced reasoning, long-horizon agentic work              | `gemini-3.1-pro-preview`              | Preview      |
| Reasoning workloads on stable GA                           | `gemini-2.5-pro`                      | GA (no shutdown date announced) |
| Price-performance on stable GA                             | `gemini-2.5-flash`                    | GA (no shutdown date announced) |
| Fast / cheapest stable GA (2.5 line)                       | `gemini-2.5-flash-lite`               | GA (no shutdown date announced) |
| UI automation via screen interaction                       | `gemini-3.6-flash` computer-use built-in tool (Preview), or `gemini-2.5-computer-use-preview-10-2025` | Preview |

[source: ai.google.dev/gemini-api/docs/changelog, retrieved 2026-08-09]
[source: ai.google.dev/gemini-api/docs/models/gemini-3.6-flash, retrieved 2026-08-09]
[source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-08-09]
[source: blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/, retrieved 2026-08-09]

[applies-to: gemini-3.6-flash] 3.6 Flash: 1,048,576 input / 65,536 output tokens (same as 3.5 Flash); inputs Text/Image/Video/Audio/PDF, output text only; knowledge cutoff **March 2026** (with the Gemini-3-family caveat that some domains may still reflect the January 2025 cutoff); pricing $1.50/1M input, $7.50/1M output including thinking tokens — cheaper output than 3.5 Flash's $9.00. Google's speed positioning is efficiency, not raw decode rate: ~17% fewer output tokens than 3.5 Flash for equivalent work and fewer reasoning steps/tool calls per task (the 17% figure is Google's, computed on the third-party Artificial Analysis Index — treat the number as vendor-cited third-party measurement). The model card also lists Grounding with Google Maps and File search as supported, plus Flex/Priority inference tiers, and documents enhanced CBRN/cyber-offense safeguards with a near-flat unjustified-refusal delta vs 3.5 Flash (+0.25pp).
[source: ai.google.dev/gemini-api/docs/models/gemini-3.6-flash, retrieved 2026-08-09]
[source: ai.google.dev/gemini-api/docs/pricing, retrieved 2026-08-09]
[source: deepmind.google/models/model-cards/gemini-3-6-flash/, retrieved 2026-08-09]
[source: blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/, retrieved 2026-08-09]

[applies-to: gemini-3.5-flash-lite] 3.5 Flash-Lite (GA 2026-07-21) is the fastest model in the 3.5 series — claimed 350 output tokens/s — priced $0.30/1M input, $2.50/1M output, and is the documented replacement for `gemini-3.1-flash-lite`.
[source: blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/, retrieved 2026-08-09]
[source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-08-09]

`gemini-3.5-flash` carries a Jan 2025 knowledge cutoff with a latest-update stamp of May 2026.
[source: ai.google.dev/gemini-api/docs/models/gemini-3.5-flash, retrieved 2026-06-01]

Gemini 3.5 Pro is in partner-only testing (not released); Google states pre-training for Gemini 4 has started. `gemini-3.1-pro-preview` remains the only current Pro-tier offering.
[source: blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/, retrieved 2026-08-09]

A separate endpoint, `gemini-3.1-pro-preview-customtools`, is tuned to prioritize your custom tools over built-in tools.
[source: ai.google.dev/gemini-api/docs/models/gemini-3.1-pro-preview, retrieved 2026-06-01]

**Computer use is now a built-in client-side tool on Flash-tier 3.x models (Preview).** This supersedes the earlier state in which only `gemini-2.5-computer-use-preview-10-2025` supported screen interaction: `gemini-3.6-flash` and `gemini-3.5-flash-lite` list "Computer use Supported (Preview)" directly, via the Gemini API and Gemini Enterprise. The dedicated 2.5 computer-use model remains available. Whether the pre-3.6 text models (3.5 Flash, 3.1 line) gained the tool was not confirmed this pass — check the per-model pages.
[source: ai.google.dev/gemini-api/docs/models/gemini-3.6-flash, retrieved 2026-08-09]
[source: blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/, retrieved 2026-08-09]

### Deprecated / retired

- `gemini-2.0-flash` / `gemini-2.0-flash-lite` GA (and `-001` variants) shut down **2026-06-01** (executed; the models page now marks them "Shut down"). [source: ai.google.dev/gemini-api/docs/models, retrieved 2026-08-09]
- `gemini-2.5-pro` / `gemini-2.5-flash` / `gemini-2.5-flash-lite`: the previously-published **2026-10-16 GA sunset date has been removed** from the live deprecations page — all three now read "No shutdown date announced." Plans keyed to the October date should be re-based; a new date can still be announced. [source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-08-09]
- `gemini-3.1-flash-lite` (the **GA** model): shutdown **2027-05-07**, unchanged; the ledger now names `gemini-3.5-flash-lite` as the recommended replacement. [source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-08-09]
- `gemini-3.6-flash`: released 2026-07-21, no shutdown date announced. [source: ai.google.dev/gemini-api/docs/deprecations, retrieved 2026-08-09]
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

`thinkingLevel` {minimal, low, medium, high} is the recommended reasoning control for Gemini 3 models and onward; it replaces the 2.5-era `thinkingBudget`. Defaults differ by model: `gemini-3.1-pro-preview` defaults to `high`, `gemini-3.6-flash` and `gemini-3.5-flash` to `medium` (thinking On by default, full four-value enum supported), `gemini-3.1-flash-lite` to `minimal`. For latency- or token-sensitive workloads on Pro or Flash, step down explicitly — do not assume reasoning is off by default the way it is on many OpenAI-compatible stacks.
[source: ai.google.dev/gemini-api/docs/thinking, retrieved 2026-08-09]

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
- **Screen interaction / computer use** is available two ways as of 2026-08-09: as a built-in client-side tool on Flash-tier 3.x models (`gemini-3.6-flash`, `gemini-3.5-flash-lite` — Preview), or via the specialized `gemini-2.5-computer-use-preview-10-2025` model. Both take screenshots as image parts and emit UI action commands.

[source: ai.google.dev/gemini-api/docs/models/gemini-3.6-flash, retrieved 2026-08-09]
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
