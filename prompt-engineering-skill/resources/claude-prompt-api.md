---
family: claude
scope: api
versions:
  - claude-fable-5
  - claude-opus-5
  - claude-opus-4-8
  - claude-opus-4-7
  - claude-sonnet-5
  - claude-sonnet-4-6
  - claude-haiku-4-5
  - claude-haiku-4-5-20251001
retrieved: 2026-08-09
primary_sources:
  - https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5
  - https://platform.claude.com/docs/en/about-claude/models/whats-new-opus-5
  - https://platform.claude.com/docs/en/build-with-claude/thinking
  - https://platform.claude.com/docs/en/about-claude/pricing
  - https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback
  - https://docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html
  - https://www-cdn.anthropic.com/d00db56fa754a1b115b6dd7cb2e3c342ee809620.pdf
  - https://platform.claude.com/docs/en/about-claude/models/overview
  - https://docs.claude.com/en/docs/about-claude/models/overview
  - https://docs.claude.com/en/docs/about-claude/model-deprecations
  - https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-8
  - https://platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5
  - https://platform.claude.com/docs/en/release-notes/overview
  - https://platform.claude.com/docs/en/api/errors
  - https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude/structured-outputs
  - https://platform.claude.com/docs/en/build-with-claude/extended-thinking
  - https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking
  - https://platform.claude.com/docs/en/build-with-claude/effort
  - https://platform.claude.com/docs/en/build-with-claude/prompt-caching
  - https://platform.claude.com/docs/en/build-with-claude/structured-outputs
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool
  - https://www.anthropic.com/claude/opus
  - https://www.anthropic.com/news/claude-opus-4-8
  - https://www.anthropic.com/transparency
maturity_note: |
  Claude Fable 5 (`claude-fable-5`) is the current flagship, GA 2026-06-09.
  Its API surface diverges from Opus 4.8 in three ways treated as
  first-class here: adaptive thinking is always on (`disabled` unsupported),
  raw thinking is never returned (`display` defaults to `"omitted"`), and
  safety-classifier refusals (`stop_reason: "refusal"` with `stop_details`)
  are a primary response path with new server-side (`fallbacks` param, beta)
  and SDK-middleware fallback mechanisms. Opus 4.8 and 4.7 remain Active and
  reject sampling parameters and manual thinking budgets. Files API,
  Citations, MCP connector are out of scope; see Anthropic docs. Fable 5
  facts added 2026-06-10; 4.x facts last re-verified 2026-06-01. A 2026-07-19
  Tier-1 browser-verification pass added Claude Sonnet 5 and pinned prefill/
  effort/tool-use/prompt-cache tables as Tier-1. A 2026-08-09 pass added Claude
  Opus 5 (`claude-opus-5`, launched 2026-07-24): thinking on by default with a
  disable-only-at-effort<=high 400 rule, classifiers + refusal surface (fifth
  category `general_harms`), 512-token cache minimum, fast mode (speed:"fast",
  research preview), and the 2026-07-01 fallback beta headers with a "default"
  server-selected mode. Same pass corrected: raw chain of thought is never
  returned on ANY model; batch refusals DO carry full stop_details; forced tool
  use now works with adaptive thinking; Opus 4.1 retirement executed 2026-08-05;
  Mythos Preview did NOT retire 2026-07-21 (deprecated, undated); the thinking
  docs moved from /adaptive-thinking to /thinking. Claims not re-touched retain
  their earlier inline dates.
---

# Claude — API-Layer Reference

API-call-level detail for the current Claude 4.x generation. Portable prompt-layer content lives in `claude-prompt.md`.

## 1. API Surface

### Endpoints and SDKs

The Messages API (`POST /v1/messages`) is the primary endpoint. Anthropic provides first-party SDKs in Python, TypeScript, C#, Go, Java, PHP, and Ruby.
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]

The Message Batches API (`/v1/messages/batches`) supports asynchronous batching at discounted pricing. Extended output tokens up to **300K** are available on Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 5, and Sonnet 4.6 via the `output-300k-2026-03-24` beta header.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-07-19]

### Platforms and model IDs

| Platform           | Fable 5 ID                    | Opus 4.8 ID                   | Opus 4.7 ID                   | Sonnet 4.6 ID                  | Haiku 4.5 ID                                      |
|--------------------|-------------------------------|-------------------------------|-------------------------------|--------------------------------|---------------------------------------------------|
| Claude API         | `claude-fable-5`              | `claude-opus-4-8`             | `claude-opus-4-7`             | `claude-sonnet-4-6`            | `claude-haiku-4-5-20251001` (alias `claude-haiku-4-5`) |
| Amazon Bedrock     | `anthropic.claude-fable-5` (geo: `us.`/`eu.` prefix; global: `global.anthropic.claude-fable-5`) | (see Bedrock model catalog) | `anthropic.claude-opus-4-7`   | `anthropic.claude-sonnet-4-6`  | `anthropic.claude-haiku-4-5-20251001-v1:0`        |
| Google Vertex AI   | (available; ID not pinned)    | (see Vertex model garden)     | `claude-opus-4-7`             | `claude-sonnet-4-6`            | `claude-haiku-4-5@20251001`                       |

The current flagship is `claude-fable-5` (GA 2026-06-09), available on the Claude API, Claude Platform on AWS, Amazon Bedrock, Vertex AI, and Microsoft Foundry. Pricing: $10/MTok input, $50/MTok output. 1M context, 128K max output. On Bedrock it is served via both `bedrock-runtime` (Invoke/Converse/Messages APIs) and `bedrock-mantle` (`https://bedrock-mantle.{region}.api.aws/anthropic/v1/messages`); the `Responses` and `Chat Completions` APIs are not supported.
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html, retrieved 2026-06-10]

`claude-mythos-5` (same weights, no blocking classifiers) is limited-release through Project Glasswing and is not generally available; it is the successor to Claude Mythos Preview.
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]

**Data-retention constraint.** Fable 5 and Mythos 5 are designated Covered Models: 30-day data retention, not available under zero-data-retention agreements. On Bedrock, using Fable 5 requires opting in to provider data sharing (`provider_data_share` via the Data Retention API; no console UI at launch). Relevant when ZDR is a procurement requirement — Fable 5 is excluded from those deployments.
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html, retrieved 2026-06-10]

The Claude API ID for the prior flagship is `claude-opus-4-8` (GA 2026-05-28). The exact Bedrock and Vertex AI ID strings for Opus 4.8 were not pinned in the retrieved primary sources; fetch the respective model catalog at integration time.
[source: docs.claude.com/en/docs/about-claude/models/overview, retrieved 2026-06-01]
[source: anthropic.com/claude/opus, retrieved 2026-06-01]
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

Claude Opus 5 (`claude-opus-5`, launched 2026-07-24) is the new Opus tier and Anthropic's recommended starting model. Platform IDs: `claude-opus-5` on the Claude API and Google Cloud; `anthropic.claude-opus-5` on Amazon Bedrock (also reachable via `InvokeModel` on `bedrock-runtime`); available on Microsoft Foundry. The dateless ID is a pinned snapshot. Context: 1M tokens is both the default and the maximum — no smaller variant, no beta header; 128K max output synchronous. Pricing $5/$25 per MTok (unchanged from Opus 4.8); cache writes $6.25 (5m) / $10 (1h), cache reads $0.50, batch $2.50/$12.50. No special data-retention requirement for general access (unlike Fable 5's Covered-Model 30-day constraint). Opus 4.8 remains available on all platforms. In Claude Code v2.1.219+ the `opus` alias resolves to Opus 5.
[source: docs.claude.com/en/release-notes/api, retrieved 2026-08-09]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-opus-5, retrieved 2026-08-09]
[source: platform.claude.com/docs/en/about-claude/pricing, retrieved 2026-08-09]
[source: anthropic.com/news/claude-opus-5, retrieved 2026-08-09]
[source: github.com/anthropics/claude-code/blob/main/CHANGELOG.md, retrieved 2026-08-09]

Claude Sonnet 5 (`claude-sonnet-5`, GA 2026-06-30) is the drop-in successor to Sonnet 4.6. Model IDs: `claude-sonnet-5` on the Claude API (both API ID and alias); `anthropic.claude-sonnet-5` on Amazon Bedrock; `claude-sonnet-5` on Google Cloud. The dateless form is a **pinned snapshot**, not an evergreen pointer. Context is 1M tokens by default (also the maximum; no smaller-context variant, no beta header); max output is 128K synchronous, 300K via the Batch API with `output-300k-2026-03-24`. Introductory pricing is $2/$10 per MTok through 2026-08-31, then standard $3/$15. Priority Tier is not available. Sonnet 5 is the first Sonnet-tier model with real-time cybersecurity safeguards; a declined request returns HTTP 200 with `stop_reason: "refusal"` (see §2), not an error.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/release-notes/overview, retrieved 2026-07-19]

Bedrock exposes **global** (dynamic-routing) and **regional** endpoints from Sonnet 4.5 onward. Vertex AI exposes **global**, **multi-region**, and **regional** endpoints. Claude is also available on Microsoft Foundry.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

The Models API (`GET /v1/models`) returns `max_input_tokens`, `max_tokens`, and a `capabilities` object per model, programmatically.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

## 2. Chat Template / Message Structure

Claude's request shape:

```json
{
  "model": "claude-opus-4-7",
  "max_tokens": 16000,
  "system": "...",
  "messages": [
    { "role": "user", "content": "..." },
    { "role": "assistant", "content": [ /* content blocks */ ] }
  ]
}
```

`system` is a top-level field (string or content-block array), not a message role. `max_tokens` is required.
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]

Content-block types on assistant output include `text`, `thinking`, `tool_use`. On user input they include `text`, `image`, `document`, `tool_result`. Thinking blocks carry `thinking` (summarized text or empty) and `signature` (encrypted full thinking, used for multi-turn continuity).
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]

### Prefilled assistant messages

Prefilled content on the **last** assistant turn returns a **400 `invalid_request_error`** on Claude Fable 5, Mythos 5, Mythos Preview, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6; the errors page states the message verbatim as "Prefilling assistant messages is not supported for this model." Sonnet 5 is not enumerated in that list, but whats-new-sonnet-5 states last-turn prefill "returns a 400 error, unchanged from Claude Sonnet 4.6." Prefills placed **elsewhere** in the conversation continue to work. This pins the previously-open Fable 5 prefill handling. Live testing confirms both halves on Opus 4.8 and the not-enumerated Sonnet 5 (verified 2026-07-19, 3/3 each tier, via OpenRouter→Anthropic): a trailing-prefill request returned HTTP 400 `invalid_request_error` with the verbatim string "This model does not support assistant message prefill. The conversation must end with a user message.", while the same prefill placed mid-conversation returned 200 with a normal answer. Migrate to Structured Outputs (§6) or system-prompt directives.
[source: platform.claude.com/docs/en/api/errors, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]
[testable: id=claude.prefill-last-turn-rejected.v1, expected=request to claude-opus-4-8 with a prefilled last assistant message returns HTTP 400 invalid_request_error]

### Refusal responses and fallback (Fable 5 and Opus 5)

[applies-to: claude-fable-5, claude-opus-5] Safety classifiers can decline a request — the docs now name both models: "Claude Fable 5 and Claude Opus 5 include safety classifiers that can decline a request." A refusal is a **successful HTTP 200**, not an error:

```json
{
  "type": "message",
  "model": "claude-fable-5",
  "content": [],
  "stop_reason": "refusal",
  "stop_details": {
    "type": "refusal",
    "category": "cyber",
    "explanation": "This request was declined because it could enable cyber harm."
  },
  "usage": { "input_tokens": 412, "output_tokens": 0 }
}
```

- `stop_details.category` values: `"cyber"`, `"bio"`, `"frontier_llm"`, `"reasoning_extraction"`, and (new as of the Opus 5 launch) `"general_harms"` — "The request could be related to an area that was determined as harmful. Benign work might sometimes trigger this category." All five categories are eligible for the same fallback handling (§2 server-side / SDK). Benign cybersecurity and beneficial life-sciences work can also trigger `cyber`/`bio`. Fallback routing changed with Opus 5: bio-blocked requests on Fable 5 now route to **Opus 5**, not Opus 4.8 (cyber-flagged still to Opus 4.8).
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-08-09]
[source: anthropic.com/news/claude-opus-5, retrieved 2026-08-09]
- `category` and `explanation` are both `null` when the refusal maps to no named category — `null` is a normal permanent value, not a placeholder. `stop_details` itself is `null` for every other stop reason. **Branch on `stop_reason == "refusal"`, never on `stop_details` or `content`.** `explanation` text is not stable; display it, do not parse it.
- A refusal can arrive before any output or mid-stream after partial output; treat partial output as incomplete and discard it.
- **Billing:** a refusal before any output is not billed and does not count against rate limits. A mid-stream refusal bills input plus the output already streamed.

[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-06-10]

**Server-side fallback (beta).** Two header generations now exist: the newer **`server-side-fallback-2026-07-01`** supports both a `"default"` mode (server-selected fallback per refusal category) and explicit model lists; the older `server-side-fallback-2026-06-01` accepts only explicit lists. Explicit list entries may override `max_tokens`, `thinking`, `output_config`, and `speed` for that attempt only. Name up to three fallback models in a `fallbacks` parameter; on a classifier decline, the API retries the next model in the chain within the same call:
[source: platform.claude.com/docs/en/about-claude/models/whats-new-opus-5, retrieved 2026-08-09]
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-08-09]

```json
{
  "model": "claude-fable-5",
  "max_tokens": 1024,
  "fallbacks": [{ "model": "claude-opus-4-8" }],
  "messages": [ ... ]
}
```

- Entries are tried in order; each must be distinct, must be one of the requested model's permitted targets (published as `allowed_fallback_models` on the Models API when the beta header is set), and may override `max_tokens`, `thinking`, `output_config`, and `speed` for that attempt only. The request must be valid as a direct request to every model named.
- Only a safety-classifier decline triggers fallback — rate limits, overloads, and server errors are returned as-is.
- The response's top-level `model` names the model that answered; a `{"type": "fallback", "from": {...}, "to": {...}}` content block marks each model boundary, and `usage.iterations` records every attempt (declining model as a `message` entry, serving model as a `fallback_message` entry).
- **Echo rules on the next turn:** keep the `fallback` block exactly where it appeared; keep `text` and everything after the final `fallback` block; drop `thinking`/`redacted_thinking`/`connector_text` and client-side `tool_use` blocks from before the final `fallback` block; keep `server_tool_use` only when paired with its result.
- Streaming: the retry happens on the same stream; mid-output, the `fallback` block arrives as an empty `content_block_start`/`content_block_stop` pair and the fallback model continues from the partial `text` output. Non-streaming mid-output declines instead discard partial output and the fallback answers from scratch.
- Not available on the Message Batches API (per-item errored result), Amazon Bedrock, Vertex AI, or Microsoft Foundry — use the SDK middleware there.
- The beta header must carry exactly the date `2026-06-01`; other `server-side-fallback-*` values reject the `fallbacks` parameter with 400.

**Client-side fallback (SDK middleware).** TypeScript, Python, Go, Java, and C# SDKs ship a refusal-fallback middleware (`BetaRefusalFallbackMiddleware` in Python) configured once on the client; it retries refusals on any platform, strips Fable 5 thinking blocks from the retry, manages `fallback` blocks in history, sends the **`fallback-credit-2026-07-01`** header automatically (superseding the 2026-06-01 header this file previously pinned), and pins follow-ups to the model that accepted via a shared `BetaFallbackState`. Not yet in Ruby/PHP SDKs. Configure the middleware **or** the server-side `fallbacks` parameter, never both on one request.
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-08-09]

Documented operational pitfalls: budget retries per request (an agent plus subagents can produce several refusals in one turn); the `fallbacks` parameter does not propagate into model calls made inside tool execution — subagent calls need their own; refusals are HTTP 200, so error-rate monitoring never sees them — instrument refusals and fallback-served responses as their own signals.
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-06-10]

Fallback behavior is surface-dependent per the system card: client applications (web/desktop/mobile) fall back to the most recent Opus model automatically with user notification; the Messages API blocks by default with opt-in server-side fallback; some Claude interfaces apply non-configurable automatic fallback with a session event.
[source: www-cdn.anthropic.com/d00db56fa754a1b115b6dd7cb2e3c342ee809620.pdf, system card 2026-06-09, retrieved 2026-06-10]

## 3. Sampling Parameters

[applies-to: claude-opus-4-7, claude-opus-4-8, claude-sonnet-5]
`temperature`, `top_p`, and `top_k` are **rejected (400 error)** when set to any non-default value on Claude Opus 4.7 and later (including Claude Opus 4.8) and on Claude Sonnet 5. Omit these parameters entirely. If you were using `temperature = 0` for determinism, note that it never produced identical outputs on any Claude model.
[source: platform.claude.com/docs/en/about-claude/model-deprecations, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/release-notes/overview, retrieved 2026-07-19]
[testable: id=claude.opus47-sampling-rejected.v1, expected=request to claude-opus-4-7 with temperature=0.5 returns HTTP 400]
[testable: id=claude.sonnet5-sampling-rejected.v1, expected=request to claude-sonnet-5 with temperature=0.5 returns HTTP 400]

On Opus 4.6, Sonnet 4.6, and Haiku 4.5 these parameters remain accepted, but `output_config.effort` is the recommended control for response characteristics.
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-04-18]

[applies-to: claude-fable-5]
Per the Bedrock model card: `temperature` must be **1.0 or unset**; `top_p` must be **≥ 0.99 and < 1.0, or unset**; `top_k` is **not supported**. Practically: omit all three, same as Opus 4.7/4.8. The card states this for the Bedrock surface; the first-party announcement does not enumerate sampling constraints separately — assume the same non-default rejection applies on the Claude API.
[source: docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html, retrieved 2026-06-10]
[testable: id=claude.fable5-sampling-rejected.v1, expected=request to claude-fable-5 with temperature=0.5 is rejected]

## 4. Reasoning / Thinking Control

### The `thinking` field

```json
"thinking": {
  "type": "enabled" | "disabled" | "adaptive",
  "budget_tokens": N,
  "display": "summarized" | "omitted"
}
```

- `type: "adaptive"` — model decides when and how much to think. The **only** supported thinking mode on Fable 5, Opus 4.8, and Opus 4.7.
- `type: "enabled"` with `budget_tokens` — manual budget. Rejected on Opus 4.8, Opus 4.7, and Sonnet 5 (400; bare `thinking.type: "enabled"` is also rejected with HTTP 400), deprecated but still functional on Opus 4.6 and Sonnet 4.6 (which still support adaptive **and** manual), still supported on Opus 4.5 / Sonnet 4.5 and older.
- `type: "disabled"` or field omitted — no thinking on Opus 4.8/4.7 and earlier. **On Fable 5, Mythos 5, and Mythos Preview the semantics differ**: adaptive thinking is always on, and `{"type": "disabled"}` is **not supported** (rejected; the docs label it "not supported" and pin an explicit 400 only for the manual `{type: "enabled"}` case, so treat the disabled case as rejected without asserting a specific status code). Control depth with `output_config.effort` instead. **Sonnet 5 differs again**: adaptive thinking is on by default, but `thinking: {type: "disabled"}` **is** supported and turns thinking off without error. **Opus 5 is a fourth pattern**: thinking is on by default (unlike Opus 4.8, where omitting the field meant no thinking); adaptive is the default and `thinking: {"type": "adaptive"}` stays valid; `{"type": "disabled"}` is accepted **only when effort is `high` or below** — combined with `xhigh` or `max` effort it returns 400 (applies to Opus 5 and later, enforced per request).
[source: platform.claude.com/docs/en/about-claude/models/whats-new-opus-5, retrieved 2026-08-09]
[testable: id=claude.opus5-thinking-disabled-effort-gate.v1, expected=claude-opus-5 with thinking.type="disabled" and effort xhigh or max returns HTTP 400; with effort high or below it succeeds]
- `budget_tokens` must be `< max_tokens` in manual mode (except with interleaved thinking).
- `display` controls whether the `thinking` field on returned thinking blocks carries summarized reasoning or is empty. Defaults: `"summarized"` on Opus 4.6, Sonnet 4.6, Haiku 4.5, earlier Claude 4; `"omitted"` on Fable 5, Mythos 5, Opus 5, Sonnet 5, Opus 4.8, Opus 4.7, and Mythos Preview. **The docs now state that no `display` setting returns the raw chain of thought on ANY model** — "Summarized thinking provides the full intelligence benefits of thinking while preventing misuse. No display setting returns the raw chain of thought." — so summarized is the maximum visibility everywhere, not only on Fable 5 / Mythos 5 as this file previously scoped it. Pass thinking blocks back unchanged in multi-turn conversations on the same model.
[source: platform.claude.com/docs/en/build-with-claude/thinking, retrieved 2026-08-09]
- **Effort/thinking configuration changes invalidate the prompt cache**: the resolved value is rendered into the prompt, so changing effort (or any thinking configuration) between requests does not preserve cached prefixes. Hold effort constant within cached conversations.
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-08-09]
- **Docs moved**: the adaptive-thinking page this file previously cited now lives at `/build-with-claude/thinking`, with satellite pages (thinking-steering-and-cost, thinking-tool-workflows, thinking-troubleshooting, extended-thinking for manual mode).
[source: platform.claude.com/docs/en/build-with-claude/thinking, retrieved 2026-08-09]

[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-06-01]
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[testable: id=claude.opus47-manual-thinking-rejected.v1, expected=request to claude-opus-4-7 with thinking.type="enabled" returns HTTP 400]
[testable: id=claude.opus47-display-default-omitted.v1, expected=request to claude-opus-4-7 with thinking.type="adaptive" and no display field returns thinking blocks with empty thinking strings]
[testable: id=claude.fable5-thinking-disabled-unsupported.v1, expected=request to claude-fable-5 with thinking.type="disabled" is rejected]
[testable: id=claude.fable5-display-default-omitted.v1, expected=request to claude-fable-5 with thinking unset returns thinking blocks with empty thinking strings]
[testable: id=claude.sonnet5-thinking-disabled-supported.v1, expected=request to claude-sonnet-5 with thinking.type="disabled" succeeds with thinking off and no error]

### `output_config.effort`

```json
"output_config": {
  "effort": "max" | "xhigh" | "high" | "medium" | "low"
}
```

| Level   | Availability                                               | Guidance                                                                                 |
|---------|------------------------------------------------------------|------------------------------------------------------------------------------------------|
| `max`   | Fable 5, Opus 5, Mythos 5, Opus 4.8, Mythos Preview, Opus 4.7, Opus 4.6, Sonnet 5, Sonnet 4.6 | Frontier problems only; often overthinks on structured tasks                             |
| `xhigh` | Fable 5, Opus 5, Mythos 5, Opus 4.8, Opus 4.7, Sonnet 5 (**not** Mythos Preview, Opus 4.6, or Sonnet 4.6, which have `max` but not `xhigh`) | Recommended starting point for coding and agentic workloads on Opus 4.7                  |
| `high`  | All models supporting effort; API default                  | Strong quality balance; default on Opus 5 and Sonnet 5 (Claude API and Claude Code); Opus 4.7 recommendation for intelligence-sensitive workloads |
| `medium`| All models supporting effort                               | Cost-sensitive workloads; recommended default for Sonnet 4.6                             |
| `low`   | All models supporting effort                               | Short, scoped tasks; latency-sensitive subagents                                         |

[applies-to: claude-opus-5] Opus 5 supports the full ladder including `xhigh` and `max`; effort is the primary steering control and defaults to `high` on the Claude API and Claude Code. Effort controls thinking volume but does not reliably shorten visible responses — prompt for length instead, and run a fresh effort sweep rather than carrying settings from earlier models.
[source: docs.claude.com/en/release-notes/api, retrieved 2026-08-09]
[source: docs.claude.com/en/docs/about-claude/models/overview, retrieved 2026-08-09]
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-08-09]

The `effort` parameter itself requires no beta header.
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-07-19]

Setting `effort: "high"` is identical to omitting the parameter. At `high` and above on adaptive-capable models, Claude almost always thinks; at `low` / `medium` it may skip thinking on simple queries.
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-04-18]

Opus 4.8 uses the same reasoning-control model as Opus 4.7: adaptive thinking only (manual budgets rejected) with `output_config.effort` as the lever. Its effort availability is now pinned: both `max` and `xhigh` are listed for Opus 4.8 (see the table above). Per-effort-level token multipliers remain unpublished; measure on your own evals.
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-06-01]
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-07-19]

[applies-to: claude-fable-5] Effort is the primary control for the intelligence/latency/cost trade-off on Fable 5 (thinking cannot be disabled, so there is no off switch — only depth). Documented guidance: `high` as the default for most tasks, `xhigh` for the most capability-sensitive workloads, `medium`/`low` for routine work; lower effort settings still perform well and often exceed `xhigh` on prior models. The effort doc lists the full `max` / `xhigh` / `high` / `medium` / `low` ladder as available on Fable 5 — `max` **is** available (the Fable prompting guide simply omits it from its worked examples).
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-07-18]

### `output_config.task_budget` (beta)

[applies-to: claude-opus-4-7, claude-fable-5]
Beta header: `task-budgets-2026-03-13` (Fable 5 supports task budgets at launch under the same header). Shape:

```json
"output_config": {
  "task_budget": { "type": "tokens", "total": 128000 }
}
```

Advisory budget across the full agentic loop (thinking + tool calls + tool results + final output). The model sees a running countdown and uses it to prioritize; it is **not** a hard cap (that is `max_tokens`). Minimum value: 20K tokens. For open-ended agentic tasks where quality matters more than speed, omit `task_budget`.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, vendor page retired as of 2026-07-19 (301 to whats-new-claude-4-8); the Opus 4.7 task-budget detail is historical, no longer vendor-documented]
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]

[applies-to: claude-fable-5] Other launch-supported features: the memory tool, tool-result clearing via context editing (beta header `context-management-2025-06-27`), compaction, and vision.
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]

### Interleaved thinking

Adaptive thinking **automatically enables interleaved thinking** — Claude can reason between tool calls. On manual thinking with Sonnet 4.6, interleaved thinking requires the `interleaved-thinking-2025-05-14` beta header; on Opus 4.6 manual thinking, interleaved is not available.
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]

### Preserving thinking blocks in multi-turn tool use

When extended/adaptive thinking is active and tool calls are involved, pass the prior assistant turn's **thinking blocks back unchanged** alongside the `tool_use` block when sending `tool_result`. Omitting them in tool loops raises an error. The `signature` field authenticates the thinking; any text inside a round-tripped `thinking` block with `display: "omitted"` is ignored (server decrypts the signature). Signatures are cross-platform compatible (Claude API, Bedrock, Vertex AI).
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]

[testable: id=claude.thinking-tool-loop-preservation.v1, expected=omitting thinking blocks from the prior assistant turn in a tool_result continuation returns HTTP 400 or downgrades the thinking mode silently]

### `tool_choice` interaction with thinking

**Forced tool use now works with adaptive thinking**: "Adaptive thinking, including on models where thinking is on by default, supports forced tool use" — `{type: "any"}` and `{type: "tool", name: "..."}` are no longer restricted to `auto` on adaptive-thinking models. The old rule (tool_choice must be `auto` or unset, with graceful degradation auto-disabling thinking) now applies only to **manual extended thinking**.
[source: platform.claude.com/docs/en/build-with-claude/thinking, retrieved 2026-08-09]
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]

## 5. Tool Use / Function Calling

### Request shape

```json
"tools": [
  {
    "name": "get_weather",
    "description": "Get current weather for a location.",
    "input_schema": {
      "type": "object",
      "properties": { "location": { "type": "string" } },
      "required": ["location"],
      "additionalProperties": false
    },
    "strict": true
  }
],
"tool_choice": { "type": "auto" }
```

`tool_choice` options: `{type: "auto"}` (default), `{type: "any"}`, `{type: "tool", name: "..."}`, `{type: "none"}`.

Set `strict: true` on a tool to guarantee that Claude's `input` payload conforms to the `input_schema` exactly. Strict mode requires `additionalProperties: false` on the schema.
[source: platform.claude.com/docs/en/agents-and-tools/tool-use/overview, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-04-18]

### Response shape

When Claude calls a tool, the response carries `stop_reason: "tool_use"` and a `tool_use` content block:

```json
{
  "type": "tool_use",
  "id": "toolu_01A09q90qw...",
  "name": "get_weather",
  "input": { "location": "Paris, FR" }
}
```

Return results as a `tool_result` content block in the next user message:

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01A09q90qw...",
  "content": "20°C, sunny",
  "is_error": false
}
```

`content` accepts string or an array of text/image blocks. `is_error: true` signals tool failure; Claude adjusts accordingly.
[source: platform.claude.com/docs/en/agents-and-tools/tool-use/overview, retrieved 2026-04-18]

### Parallel tool use

All current 4.x models emit multiple `tool_use` blocks in a single assistant turn when tasks are independent. Prompting can raise this to ~100% when needed; see `claude-prompt.md` §3 for prompt patterns.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Client vs server tools

- **Client tools** run in the caller's application: user-defined tools plus Anthropic-schema tools like `bash`, `text_editor`, and `memory`.
- **Server tools** run in Anthropic's infrastructure: `web_search`, `code_execution`, `web_fetch`, `tool_search`. Server tool types are **versioned**. Check the tool reference for the current versioned type string before use.

[source: platform.claude.com/docs/en/agents-and-tools/tool-use/overview, retrieved 2026-04-18]

For the web search server tool specifically, the current latest type is `web_search_20260318` (adds a `response_inclusion` parameter); the earlier `web_search_20260209` (adds dynamic filtering; requires the code-execution tool enabled) and `web_search_20250305` remain available. Citations are always on, returned as `web_search_result_location` blocks (url, title, cited_text up to 150 chars). Pricing is $10 per 1,000 searches. An organization admin must enable the tool in the Console before use.
[source: platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool, retrieved 2026-07-18]

### Tool-use system-prompt overhead

When a request includes `tools`, a tool-use system prompt is added automatically. The overhead is **per-model**, not a flat figure (the previous 346/313 figure this file carried matched no current model):

| Model                 | `tool_choice: auto \| none` | `tool_choice: any \| tool` |
|-----------------------|-----------------------------|----------------------------|
| Opus 5                | 286                         | 406                        |
| Opus 4.8              | 290                         | 410                        |
| Opus 4.7              | 675                         | 804                        |
| Opus 4.6              | 497                         | 589                        |
| Opus 4.5              | 496                         | 588                        |
| Opus 4.1 (deprecated) | 313                         | 315                        |
| Opus 4 (retired)      | 313                         | 315                        |
| Sonnet 5              | 354                         | 474                        |
| Sonnet 4.6            | 497                         | 589                        |
| Sonnet 4.5            | 496                         | 588                        |
| Sonnet 4 (retired)    | 313                         | 315                        |
| Haiku 4.5             | 496                         | 588                        |
| Haiku 3.5 (retired)   | 264                         | 355                        |

No `tools` field + `tool_choice: none` → 0 additional tokens. These figures were re-verified against the tool-use overview on 2026-07-19 (Tier-1); the Opus 5 row (286/406) was added from the pricing page on 2026-08-09, which also documents the bash tool adding **325 tokens** on Opus 5/4.8/4.7. Fable 5 still has no row in the vendor table; fetch the tool-use overview at integration time for its row and re-check the others, since these values can drift per model revision.
[source: platform.claude.com/docs/en/agents-and-tools/tool-use/overview, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/pricing, retrieved 2026-08-09]

## 6. Structured Outputs

```json
"output_config": {
  "format": {
    "type": "json_schema",
    "schema": {
      "type": "object",
      "properties": { "...": { "type": "string" } },
      "required": ["..."],
      "additionalProperties": false
    }
  }
}
```

- **Parameter:** `output_config.format`. (The earlier `output_format` parameter and `structured-outputs-2025-11-13` beta header are deprecated but still accepted during transition.)
- **Required schema constraint:** `additionalProperties: false` on every object.
- **Supported models:** Fable 5, Opus 5, Mythos 5, Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 5, Sonnet 4.6, Sonnet 4.5, Opus 4.5, Haiku 4.5, Mythos Preview (Claude API). No beta header required. Opus 5 is natively **GA on Amazon Bedrock** for structured outputs.
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-08-09]
- **Capitalization caveat (new):** string `enum`/`const` value capitalization is not guaranteed — Claude may return a value differing from the schema only in case. Compare case-insensitively and avoid enums that differ only in case. A **180-second grammar-compilation timeout** is also documented for large schemas.
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-08-09]
- **Platform support:** Claude API GA across the current-gen set (incl. Fable 5, Opus 4.8, Sonnet 5, Mythos Preview). **Google Cloud Vertex AI: Preview (Pre-GA), not GA.** Supported there only on Opus 4.7, Sonnet 4.6, and Opus 4.6, with Opus 4.5, Sonnet 4.5, and Haiku 4.5 marked "coming soon"; Fable 5, Sonnet 5, Opus 4.8, and Mythos Preview are absent entirely. See the Vertex structured-outputs subsection below for the feature split and constraints (this corrects an earlier "Vertex is GA, including Mythos Preview" statement). Amazon Bedrock supports 4.6 and earlier current-gen plus Opus 4.7 / Sonnet 5 / Mythos Preview. Microsoft Foundry: 4.6 and earlier GA; Sonnet 5 / Opus 4.7 / Mythos Preview require the Hosted-on-Anthropic deployment.
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-07-18]
[source: docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude/structured-outputs, retrieved 2026-07-19]
- **Combined with tool use:** Max 20 strict tools per request, 24 optional parameters total across schemas, 16 parameters with `anyOf` unions.
- **Schema violation:** returns HTTP 400 (rare due to constrained decoding). Safety refusals return `stop_reason: "refusal"` with status 200; output may not match schema in that case. `stop_reason: "max_tokens"` truncation may also produce non-conforming output.

[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-04-18]

### Supported JSON Schema subset

Supported: `object`, `array`, `string`, `integer`, `number`, `boolean`, `null`, `enum`, `const`, `anyOf`, `allOf`, `$ref`, `$def`, `required`, `additionalProperties: false`, string formats (`date-time`, `date`, `time`, `duration`, `email`, `hostname`, `uri`, `ipv4`, `ipv6`, `uuid`), `minItems` (0 or 1 only), simple `pattern` regex.

Not supported: recursive schemas, complex enum types, external `$ref` URLs, numeric constraints (`minimum`, `maximum`, `multipleOf`), string length constraints (`minLength`, `maxLength`), most array constraints beyond `minItems`, `additionalProperties: true`.
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-04-18]

### Vertex AI structured outputs (Preview)

On Google Cloud Vertex AI, structured outputs for Claude partner models is a **Preview** feature under the Pre-GA Offerings Terms, not GA. Model availability differs from the Claude API set:

- **Supported:** Claude Opus 4.7, Claude Sonnet 4.6, Claude Opus 4.6.
- **Coming soon:** Claude Opus 4.5, Claude Sonnet 4.5, Claude Haiku 4.5.
- **Absent entirely** (neither supported nor coming soon): Claude Fable 5, Claude Sonnet 5, Claude Opus 4.8, Claude Mythos Preview.

Two complementary features are exposed:

- **JSON outputs** via `output_config.format` (`json_schema` only).
- **Strict tool use** via `tools[].strict`.

Both require `additionalProperties: false` on every object and every property listed in `required`; numeric and string-length constraints are not supported (the same subset limits as the Claude API surface above). The feature is disabled by default and gated by the org-policy constraint `constraints/vertexai.allowedPartnerModelFeatures`, which must allow `structured_outputs`.
[source: docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude/structured-outputs, retrieved 2026-07-19]

## 7. Caching, Batch, Streaming

### Prompt caching

```json
"cache_control": { "type": "ephemeral", "ttl": "5m" | "1h" }
```

- **Placement:** on individual blocks in `tools`, `system`, or `messages[].content`; or at request level for automatic caching.
- **Minimum cacheable tokens** (Claude API / Claude Platform on AWS / Google Cloud / Microsoft Foundry): Fable 5 / Mythos 5 / **Opus 5** = **512** (Opus 5 down from Opus 4.8's 1,024); Mythos Preview / Opus 4.7 = **2048**; Opus 4.6 / Opus 4.5 = **4096**; Opus 4.8 / Sonnet 5 / Sonnet 4.6 / Sonnet 4.5 = **1024** (also Opus 4.1 retired, Opus 4 retired except Google Cloud, Sonnet 4 retired except Bedrock/Google Cloud); Haiku 4.5 = **4096**; Haiku 3.5 = **2048**. **Bedrock override:** on Amazon Bedrock the minimum for Fable 5 and Mythos 5 is **1,024**; no separate Bedrock override is stated for Sonnet 5, so Sonnet 5 is 1,024 on both the standard platforms and Bedrock. (Opus 4.7 dropped 4096→2048 and Sonnet 4.6 dropped 2048→1024 from the prior figures.)
  [source: platform.claude.com/docs/en/build-with-claude/prompt-caching, retrieved 2026-07-19]
  [source: platform.claude.com/docs/en/about-claude/models/whats-new-opus-5, retrieved 2026-08-09]
- **Breakpoints:** max 4 explicit per request (1 slot consumed by automatic caching if used together).
- **Cost model:** cache write = 1.25× base input (5m TTL) or 2.0× base input (1h TTL). Cache read = 0.1× base input for both TTLs.
- **Thinking blocks** cannot carry explicit `cache_control` but are cached alongside content in tool-use loops; they count as input tokens on cache read.
- **Cache invalidation:** `tool_choice` change, images added/removed, thinking parameter change, and non-tool-result user content in a thinking-enabled conversation all bump messages cache. Tools and system cache survive most message-level changes.
- **Workspace isolation** since **2026-02-05**: caches are keyed per workspace, not per organization.

[source: platform.claude.com/docs/en/build-with-claude/prompt-caching, retrieved 2026-04-18]

[applies-to: claude-fable-5] On the first-party Claude API the Fable 5 (and Mythos 5) minimum is **512 tokens** per checkpoint; on Amazon Bedrock it is **1,024 tokens** per cache checkpoint. Both surfaces allow max 4 checkpoints per request, 5-minute and 1-hour TTLs, on `system`, `messages`, and `tools`.
[source: platform.claude.com/docs/en/build-with-claude/prompt-caching, retrieved 2026-07-19]
[source: docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html, retrieved 2026-06-10]

[applies-to: claude-fable-5] **Fallback credit.** When a Fable 5 request is refused and retried on another model, fallback credit refunds the prompt-cache cost of switching. Server-side fallback and the SDK middleware apply it automatically; manual retries opt in with the `fallback-credit-2026-06-01` beta header. See §2 for the fallback mechanics.
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-06-10]

Response `usage` fields:

```json
"usage": {
  "input_tokens": 50,
  "cache_creation_input_tokens": 0,
  "cache_read_input_tokens": 100000,
  "output_tokens": 503,
  "cache_creation": {
    "ephemeral_5m_input_tokens": 456,
    "ephemeral_1h_input_tokens": 100
  }
}
```

Verify caching occurred by checking `cache_creation_input_tokens` and `cache_read_input_tokens`; both 0 means the prompt fell below the minimum and cached silently as no-op.
[source: platform.claude.com/docs/en/build-with-claude/prompt-caching, retrieved 2026-04-18]

### Streaming (SSE)

Stream events:

- `message_start` → `content_block_start` → `content_block_delta` (multiple) → `content_block_stop` → ... → `message_delta` → `message_stop`.
- Delta types: `text_delta`, `thinking_delta`, `signature_delta`, `input_json_delta` (tool call input streaming).
- With `thinking.display: "omitted"`, **no `thinking_delta` events** are emitted; only `signature_delta` arrives in the thinking block, and text streaming begins immediately after.

[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]

### Batch API

Extended output up to 300K tokens available on Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 5, and Sonnet 4.6 with the `output-300k-2026-03-24` beta header. Caching multipliers **stack** with Batch discounts.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/build-with-claude/prompt-caching, retrieved 2026-04-18]

## 8. Deployment Flags

For Claude this section covers **beta headers** and **platform routing**, rather than self-hosted inference flags (no open-weights release).

### Beta headers

| Header                              | Purpose                                                    | Applies to                     |
|-------------------------------------|------------------------------------------------------------|--------------------------------|
| `server-side-fallback-2026-07-01`   | Enable `fallbacks` incl. the `"default"` server-selected mode | Fable 5 / Opus 5 refusal flows (Claude API, Claude Platform on AWS) |
| `server-side-fallback-2026-06-01`   | Older header: `fallbacks` with explicit lists only         | Fable 5 (Claude API, Claude Platform on AWS) |
| `fallback-credit-2026-07-01`        | Reprice manual refusal retries (prompt-cache refund); sent automatically by the SDK middleware | Fallback flows |
| `mid-conversation-tool-changes-2026-07-01` | Add/remove tools between turns while preserving the prompt cache (beta) | Fable 5, Mythos 5, Opus 4.8, Opus 5 |
| `task-budgets-2026-03-13`           | Enable `output_config.task_budget`                         | Opus 4.7, Fable 5              |
| `context-management-2025-06-27`     | Tool-result clearing via context editing                   | Fable 5 (launch-supported)     |
| `interleaved-thinking-2025-05-14`   | Enable interleaved thinking in manual thinking mode        | Sonnet 4.6 manual mode         |
| `output-300k-2026-03-24`            | Extended output up to 300K tokens via Batch API            | Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 5, Sonnet 4.6 |
| `structured-outputs-2025-11-13`     | Legacy; Structured Outputs no longer requires a beta header| Deprecated, still accepted     |

[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-06-10]
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, vendor page retired as of 2026-07-19 (301 to whats-new-claude-4-8); the task-budgets header detail is historical, no longer vendor-documented]
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/structured-outputs, retrieved 2026-04-18]

### Fast mode (research preview)

A per-request `speed: "fast"` parameter runs Opus 5 and Opus 4.8 at roughly 2.5x speed for premium pricing — Opus 5 fast mode is $10/MTok input, $50/MTok output. Claude API only; **incompatible with the Batch API**; caching multipliers stack on the fast-mode rates. Lifecycle: fast mode was removed from Opus 4.7 on 2026-07-24 (`speed: "fast"` now returns a hard error there) and from Opus 4.6 on 2026-06-29 (silently downgrades to standard speed — a divergent failure mode worth handling explicitly).
[source: platform.claude.com/docs/en/about-claude/models/whats-new-opus-5, retrieved 2026-08-09]
[source: docs.claude.com/en/release-notes/api, retrieved 2026-08-09]

### Platform routing

Bedrock (Sonnet 4.5+): choose **global** endpoints (dynamic routing across regions) or **regional** endpoints (guaranteed geographic routing).

Vertex AI: choose **global**, **multi-region** (within a geographic area), or **regional** endpoints.

`signature` values on thinking blocks are compatible across Claude API, Bedrock, and Vertex AI — a value generated on one platform validates on another.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/extended-thinking, retrieved 2026-04-18]

## 9. Deprecations and Breaking Changes

### Fable 5 changes from Opus 4.8

[applies-to: claude-fable-5]

- **Thinking cannot be disabled.** Adaptive thinking applies whenever `thinking` is unset; `thinking: {"type": "disabled"}` is not supported. Code that explicitly disables thinking must drop the field and tune `output_config.effort` instead.
- **Raw thinking is never returned.** `display` defaults to `"omitted"` (empty `thinking` fields); `"summarized"` is the maximum visibility. Any pipeline depending on raw chain-of-thought has no Fable 5 path.
- **Refusals become a primary response path.** Safety classifiers (`cyber`, `bio`, `frontier_llm`, `reasoning_extraction`) decline requests as HTTP 200 with `stop_reason: "refusal"`; refusal rates are materially higher than on previous Claude models. Code that only handles `end_turn` / `tool_use` / `max_tokens` will mis-handle these. Configure fallback (§2) and instrument refusals as their own monitoring signal.
- **Batches:** a refused batch item returns `result.type: "succeeded"` with `stop_reason: "refusal"`. Corrected 2026-08-09: batch results **carry the same full `stop_details` object as synchronous responses** — refusals are detectable via either `stop_reason` or `stop_details.type` (this supersedes the earlier "stop_details may be null on batch results" claim). Batch refusals never mint a `fallback_credit_token`. The `fallbacks` parameter is rejected on the Batches API; collect refused items, strip Fable 5 thinking blocks from multi-turn histories, and resubmit on a fallback model.
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-08-09]
- **Sampling:** non-default `temperature`/`top_p` and any `top_k` rejected (carried over from Opus 4.7/4.8 — see §3 for the exact Bedrock-documented bounds).
- **Data retention:** Covered Model — 30-day retention, no ZDR (§1).

[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-06-10]
[source: docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html, retrieved 2026-06-10]

### Opus 4.7 and Opus 4.8 breaking changes from Opus 4.6

[applies-to: claude-opus-4-7, claude-opus-4-8]

Opus 4.8 carries the same three hard API-rejection changes as Opus 4.7 (sampling, manual thinking, default `thinking.display`). The tokenizer and behavioral-shift bullets below were documented against Opus 4.7; the retrieved sources did not confirm identical figures for Opus 4.8.

- **Sampling parameters rejected.** `temperature`, `top_p`, `top_k` non-default values → 400 on Opus 4.7 and later, including Opus 4.8. Omit them.
[source: platform.claude.com/docs/en/about-claude/model-deprecations, retrieved 2026-07-19]
- **Manual thinking rejected.** `thinking.type: "enabled"` → 400 (adaptive is the only supported mode on both Opus 4.8 and Opus 4.7). Use `{type: "adaptive"}` with `output_config.effort`.
[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-06-01]
- **`thinking.display` default changed** from `"summarized"` (Opus 4.6) to `"omitted"` on Opus 4.7, Opus 4.8, Sonnet 5, Fable 5, Mythos 5, and Mythos Preview. The `thinking` field returns empty unless you set `display: "summarized"` explicitly. (This default is now documented on the Adaptive thinking page.)
- [applies-to: claude-opus-4-7] **New tokenizer.** Token counts for the same text run higher than Opus 4.6; the original 1.0× to 1.35× figure is historical and no longer vendor-documented as of 2026-07-19. Update `max_tokens` and compaction thresholds with headroom.
- [applies-to: claude-opus-4-7] **Behavioral shifts** (more literal instruction following, shorter default verbosity, fewer default tool calls) are not API errors but require prompt tuning — see `claude-prompt.md` §6.

[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, vendor page retired as of 2026-07-19 (301 to whats-new-claude-4-8, which no longer documents the tokenizer/behavioral claims); historical, no longer vendor-documented]

### Claude 4.6-generation deprecations

- `thinking.type: "enabled"` with `budget_tokens` is **deprecated** on Opus 4.6 and Sonnet 4.6; still functional. Migrate to `{type: "adaptive"}` + `effort`.
- **Prefilled assistant messages** on the last turn are deprecated across 4.6+ models (and rejected by Mythos Preview).

[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Model retirements

- **Claude Sonnet 4** (`claude-sonnet-4-20250514`), deprecated 2026-04-14, was **retired 2026-06-15** (completed). Replacement `claude-sonnet-4-6`.
- **Claude Opus 4** (`claude-opus-4-20250514`), deprecated 2026-04-14, was **retired 2026-06-15** (completed). Replacement `claude-opus-4-8`.
- **Claude Opus 4.1** (`claude-opus-4-1-20250805`) — retirement **executed on schedule 2026-08-05**; the deprecations table shows state Retired. The Aug 5 release note recommends upgrading to Opus 5 while the deprecation-history table still lists `claude-opus-4-8` as the replacement; both statements are live.
- **Claude Mythos Preview** (`claude-mythos-preview`) — **did NOT retire 2026-07-21** as previously scheduled. As of 2026-08-09 the deprecations page lists it as Deprecated with no executed or dated retirement, and the models overview still describes it operating within Project Glasswing. Replacement `claude-mythos-5`.
- **Claude Haiku 3** (`claude-3-haiku-20240307`) is **Retired** — retirement date **April 20, 2026**. Replacement `claude-haiku-4-5-20251001`.
- **Lifecycle floors:** Opus 5 Active, tentative retirement not sooner than **2027-07-24**; Opus 4.8 not sooner than **2027-05-28**; Opus 4.7, Sonnet 5, Sonnet 4.6, Haiku 4.5 Active.
- **Flagship-continuity note:** Fable 5 and Mythos 5 access was suspended and restored on 2026-07-01 (release-note bullet: "We've restored access to Claude Fable 5 and Claude Mythos 5"; statement at anthropic.com/news/redeploying-fable-5, not fetched — see Gaps).

[source: platform.claude.com/docs/en/about-claude/model-deprecations, retrieved 2026-08-09]
[source: docs.claude.com/en/release-notes/api, retrieved 2026-08-09]

## 10. Gaps

- **Fable 5 Vertex AI and Microsoft Foundry ID strings** were not pinned; availability is documented, the exact IDs are not. Fetch the respective catalogs at integration time.
- **`allowed_fallback_models` contents** for `claude-fable-5` (which models are permitted fallback targets beyond the documented Opus 4.8 example) must be read live from the Models API with the `server-side-fallback-2026-06-01` header set.
- **Server-side fallback sticky-routing and billing subsections** of the refusals-and-fallback doc were not captured in the retrieval pass; consult the live page before building billing-sensitive fallback logic.
- **Files API, PDF support, Citations, MCP connector, Claude Code–specific API behaviors** are all real product surfaces but out of scope here. See Anthropic's per-feature docs.
- **Claude Managed Agents and the web-search-tool research pattern** are documented in `resources/agent-orchestration-surfaces.md` and `resources/deep-research-agents.md`.
- **`redacted_thinking` content-block type** is referenced in some primary sources for safety-redacted reasoning; the retrieval pass did not pin down its exact semantics or trigger conditions. Treat as partial until re-verified.
- **Anthropic SDK per-language parameter name variance** (e.g. `outputConfig` in TypeScript, `OutputConfig` in C#/Java) is predictable from the JSON field names but not exhaustively documented here.
- **Exact current version of each server-tool type** (e.g. `web_search_20260209`) drifts. Fetch the tool reference page at integration time rather than relying on a stamped value.
- **Quantitative `effort`-level token-spend multipliers** are not published per-model per-workload; Anthropic's guidance is qualitative. Measure on your own evals.
