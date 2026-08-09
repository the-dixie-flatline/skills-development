---
family: deepseek
scope: prompt
versions:
  - deepseek-v4-flash
  - deepseek-v4-pro
  - deepseek-ai/DeepSeek-V4-Flash-0731
  - deepseek-ai/DeepSeek-V3.2
retrieved: 2026-08-09
primary_sources:
  - https://api-docs.deepseek.com/updates
  - https://api-docs.deepseek.com/quick_start/pricing
  - https://api-docs.deepseek.com/guides/thinking_mode
  - https://api-docs.deepseek.com/guides/tool_calls
  - https://api-docs.deepseek.com/guides/responses_api
  - https://api-docs.deepseek.com/guides/kv_cache
  - https://api-docs.deepseek.com/api/create-chat-completion
  - https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.2
maturity_note: |
  DeepSeek V4 is the production generation. On 2026-07-31 the model behind
  `deepseek-v4-flash` was replaced with DeepSeek-V4-Flash-0731 — the official
  release (the April launch is retroactively labeled Preview) — without a new
  API model ID. 0731 is a re-post-train focused on agentic capability;
  vendor benchmarks show it beating the V4-Pro Preview on agentic suites.
  Thinking is now documented default-on at effort high, and flash gained a
  real `low` effort tier. `deepseek-v4-pro` still serves the Preview build.
  Legacy `deepseek-chat` / `deepseek-reasoner` retired 2026-07-24. This pass
  adds field-observed session-establishment and long-session guidance
  (probed 2026-08-09 via OpenRouter, provider-pinned; see inline markers).
  Models remain text-only. Open weights MIT.
---

# DeepSeek — Prompt-Layer Reference

Portable prompting guidance for the current DeepSeek generation. API-layer detail (chat template tokens, API shapes, caching mechanics, Responses API) lives in `deepseek-prompt-api.md`.

## 1. Model Selection

Current generation is DeepSeek V4. The API model IDs are stable aliases; the pricing page documents which model version each serves.

| API Model ID        | Serving version         | Notes                                                              |
|---------------------|-------------------------|--------------------------------------------------------------------|
| `deepseek-v4-flash` | DeepSeek-V4-Flash-0731  | Official release (public beta since 2026-07-31); 284B total / 13B active; effort low/high/max |
| `deepseek-v4-pro`   | DeepSeek-V4-Pro (Preview build) | 1.6T total / 49B active; official release "will follow soon"; effort high/max only |

The 0731 update did **not** mint a new API ID — "simply set the model name to `deepseek-v4-flash` to use the latest version." It is a re-post-train (same architecture and size as the Preview) targeted at agentic capability. Context length is 1M tokens; max output 384K. Aggregators pin the checkpoint under dated slugs (OpenRouter: `deepseek/deepseek-v4-flash-0731`) even though the native API does not.
[source: api-docs.deepseek.com/updates, retrieved 2026-08-09]
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]

Vendor-published agentic benchmarks for 0731 vs the flash Preview: Terminal Bench 2.1 61.8 → 82.7, DeepSWE 7.3 → 54.4, Toolathlon-Verified 49.7 → 70.3 — and 0731 **outperforms the V4-Pro Preview** on every listed agentic suite despite far fewer active parameters. An independent 445-trial public-harness run reproduced the Terminal-Bench 82.7% figure. Vendor benchmark configs used max effort, temperature=1.0, top_p=0.95 under a harness that is not yet released, so treat the absolute numbers as vendor-grade until the harness ships.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]
[source: api-docs.deepseek.com/updates, retrieved 2026-08-09]

### Legacy API model IDs (retired)

`deepseek-chat` and `deepseek-reasoner` retired **2026-07-24, 15:59 UTC** as scheduled; both are gone from the Chat Completions model enum and the pricing page. Any remaining code paths naming them are dead.
[source: api-docs.deepseek.com/news/news260424, retrieved 2026-08-09]
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]

### Open weights

- **`deepseek-ai/DeepSeek-V4-Flash-0731`** — HF, MIT, official release superseding the preview; ships with a bundled DSpark speculative-decoding module (hence 304B safetensors vs the 284B/13B base architecture). Deployment detail in `deepseek-prompt-api.md` §8.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]
- **`deepseek-ai/DeepSeek-V3.2`** — HF, MIT, 685B total, DSA architecture. Prior generation.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### Selection rules

- **Daily-driver chat and agentic work** — `deepseek-v4-flash` (0731). Thinking on by default at effort high; drop to `low` for quick conversational turns, raise to `max` for hard problems.
- **Hardest reasoning** — `deepseek-v4-pro` at `max`, until its own official release lands; note the vendor's own agentic benchmarks currently favor flash-0731 over the pro Preview for agent workloads.
- **Cost-sensitive high-volume** — flash: $0.14/M input (cache miss), $0.0028/M on cache hit, $0.28/M output as of 2026-08-09 — but a significant across-the-board price increase is pre-announced with no date. Budget accordingly.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]
- **Self-hosted** — 0731 open weights (MIT) — most permissive license of any frontier-scale model in this reference library.
- **Multimodal use** — DeepSeek is **text-only** on current versions. Not the right family.

## 2. Prompt Structure Conventions

The API is exposed over OpenAI ChatCompletions, OpenAI Responses (flash-only), and Anthropic interfaces. Prompts portable from OpenAI Chat Completions generally work with a `base_url` swap.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]

### Roles

- `system` / `user` / `assistant` / `tool` on Chat Completions.
- On the **Responses API**, `developer` is additionally accepted and **is treated as `system`** — a general alias on that surface only.
[source: api-docs.deepseek.com/guides/responses_api, retrieved 2026-08-09]
- [applies-to: deepseek-ai/DeepSeek-V3.2] `<｜Developer｜>` at the tokenizer level is **used exclusively for search-agent scenarios** on V3.2. Not a general-purpose role.
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

### Chat template

At the open-weights level DeepSeek uses full-width special tokens (`｜`, `▁`) and ships **no Jinja template**; 0731 provides the `encoding_dsv4` Python encoder, whose `encode_messages(...)` takes `thinking_mode` and `reasoning_effort` arguments — effort is rendered into the token stream, not only an API field. Exact tokens and encoder detail in `deepseek-prompt-api.md`.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]

## 3. Instruction Patterns

### Thinking is on by default; effort is the main lever

Thinking mode is **enabled by default with effort `high`**. Controls: `thinking: {"type": "enabled" | "disabled"}` or `reasoning_effort: "low" | "high" | "max"` — flash supports all three levels (`low` is a real tier on 0731); pro temporarily supports only high/max.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]

[field-observed] Effort tiers separate on reasoning volume: on a moderate two-part reasoning prompt, `low` spent ~180 reasoning tokens vs ~840 (`high`) and ~620 (`max`) — low is materially lighter; high vs max did not separate at N=1 (2026-08-09, via OpenRouter/Novita fp8).

Community-observed quality notes, both Tier 2: a practitioner's generation smoke test produced a degraded result at the default reasoning level via OpenRouter but a clean one with `reasoning_effort` explicitly raised to `high` — when output quality matters, set effort explicitly rather than trusting the routed default. Separately, an effort-ladder writeup reports `low` being *more verbose in final output* than `high` in some cases — verbosity and reasoning depth are not the same axis.
[tier: 2, source: simonwillison.net/2026/Jul/31/deepseek-v4-flash-0731/, retrieved 2026-08-09]
[tier: 2, source: reddit.com/r/LocalLLaMA/comments/1vdqsod/, retrieved 2026-08-09 via search snippet — Reddit blocks automated fetch; thread not independently re-verified]

### Sampling is dead in thinking mode

In thinking mode, `temperature`, `top_p`, `presence_penalty`, and `frequency_penalty` are all silently ignored. Since thinking is the default, assume prompt wording — not sampling — is your only steering surface unless you explicitly disable thinking.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

### Reasoning content round-trip rule (V4, unchanged on 0731)

`reasoning_content` comes back beside `content` in thinking mode. On turns without tool calls, passing it back is ignored; **for requests carrying `tools`, it must be passed back or the native API returns HTTP 400**. Via OpenRouter this hard-400 does not reproduce (stripped tool-turn resubmits returned 200 — DeepInfra 2026-07-19 2/2, Novita on 0731 2026-08-09 2/2); code written against OpenRouter's tolerance breaks on the native API. Full mechanics and the testable claim in `deepseek-prompt-api.md` §4.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

### Local format instructions override session rules

[field-observed] When a turn carries a terse local format instruction ("Answer with just the number"), 0731 obeys it and drops standing session-format rules (a required sign-off line, a required addressee) on that turn — 10/12 such turns dropped the session sign-off while surrounding turns kept it (2026-08-09, scripted 12-40-turn sessions, via OpenRouter/Novita, N=9 sessions). This is precedence, not drift: compliance returns on the next turn. If a session rule must survive terse-format turns, restate it inside those turns or scope it explicitly ("even when asked for a bare answer, append the sign-off").

### Function calling

OpenAI-shaped `tools`; `tool_choice` `none`/`auto`/`required` plus named-function form (defaults: none without tools, auto with tools); max 128 functions; strict mode on the beta endpoint works in both thinking and non-thinking mode. Detail in `deepseek-prompt-api.md` §5.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]

## 4. Context Window Practical Guidance

- Context length is **1M tokens**; max output **384K**. The vendor recommends a maximum output length of 384K for both `high` and `max` effort.
[source: api-docs.deepseek.com/quick_start/pricing, retrieved 2026-08-09]
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]
- **`max_tokens` includes the chain-of-thought.** The observable failure when the CoT exhausts the budget is a **silent empty `content`** on a 200 response, not an error. [field-observed] With thinking on (default) and `max_tokens: 4096`, log-analysis turns returned reasoning_tokens = 4096 and an empty reply (4/63 such turns, 2026-08-09, via OpenRouter/Novita). For analysis or summarization turns with thinking on, budget `max_tokens` at 8K+ and treat an empty `content` as the CoT-exhaustion signature.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

### Establishing a session (daily-driver pattern)

[field-observed] **System-role placement and first-user-message placement of a session contract performed identically** on 0731: a 6-rule session contract (language, addressee, sign-off, length cap, exact-reply protocol, code-language default) held at the same rate whether sent as the `system` message or prepended to the first user turn (23/27 sign-off, 36/36 language, 9/9 exact-protocol, 3/3 recall in each arm; a no-contract control scored ~0 on the same checks). N=3 twelve-turn sessions per arm, one provider (Novita fp8, via OpenRouter), 2026-08-09 — treat as indicative. Practical rule: use the `system` role when you control it (convention, and it keeps the cacheable prefix clean), but a wrapper that can only prepend to the first user message loses nothing measurable.

What belongs in the session-establishment prompt, given this family's mechanics:

- **State rules positively and scope them against terse-format turns** (§3 — local instructions win otherwise).
- **Put the stable material first** — persona, rules, reference data — and keep it byte-stable across the session: caching is automatic prefix matching, and a cache-hit token costs 2% of a miss ($0.0028 vs $0.14 per M). Any edit to earlier content invalidates the prefix from that point.
[source: api-docs.deepseek.com/guides/kv_cache, retrieved 2026-08-09]
- **Pick the thinking on/off state up front and hold it.** [field-observed] Toggling thinking off mid-session invalidated the cached prefix and shrank the rendered prompt by ~79 tokens (a thinking-mode template preamble); changing `reasoning_effort` (high→low→max→high) did **not** invalidate the cache (N=1 per cell, 2026-08-09, via OpenRouter/Novita). Effort is the cheap mid-session knob; the thinking toggle is the expensive one.

### Managing long sessions

[field-observed] **No constraint drift or recall loss was observed out to 40 turns / ~237K tokens of context.** Session-rule compliance held flat across turns 1-12 / 13-24 / 25-40 (~95-100% of checks in each phase, in both a ~70K and a ~237K regime); a fact planted in turn 1 was recalled correctly at turns 20 and 40 (6/6, including at 237K); an exact-reply protocol ("STATUS?" → "GREEN") stayed exact at every 4-turn checkpoint through 237K. Single scripted fixture, one provider (Novita fp8), N=1 session per regime, 2026-08-09 — a floor-level result ("no gross degradation"), not a guarantee across workloads.

Practical consequences for a daily driver:

- **Append-only history is the whole cache game.** With a stable prefix and append-only turns, 40/40 long-session requests hit the cache at a median 95.9% cached fraction even past 200K context [field-observed, same fixture]. Do not rewrite, summarize-in-place, or reorder earlier turns mid-session — that converts ~$0.0028/M input back into ~$0.14/M and rebuilds the cache from the edit point. If you compact, compact once at a session boundary and start a fresh session with the summary as the new stable prefix.
- **Long-session reliability in agent loops is contested territory** (chat-style sessions above held clean). Tier 2 reports conflict: one practitioner reports increased infinite-looping and "talking to itself" without executing tool calls plus unprompted topic drift vs the pre-0731 flash; another reports long agentic sessions degrading *less* than expected, with 0731 fixing most of the context-rot of earlier flash builds. A recurring community pattern for unattended loops: pair flash with a supervisory "advisor" model watching for loops and rabbit-holes; with one it is described as reliable and cheap, without one erratic. [community-reported, all Tier 2]
[tier: 2, source: news.ycombinator.com/item?id=49214008, retrieved 2026-08-09]
[tier: 2, source: reddit.com/r/DeepSeek/comments/1vg6r9z/, retrieved 2026-08-09 via search snippet — Reddit blocks automated fetch; thread not independently re-verified]
- **Tool-heavy requests slow down.** A practitioner reports tool calling becoming very slow once many tools are attached, while reasoning-only turns are fine. Keep daily-driver tool rosters small; the 128-function ceiling is not an invitation. [community-reported]
[tier: 2, source: news.ycombinator.com/item?id=49214008, retrieved 2026-08-09]

## 5. Multimodal Conventions

**DeepSeek is text-only.** No image, video, or audio input on any current version. For multimodal workloads, choose a different family.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]

## 6. Behavioral Quirks

- **Thinking default-on changes the migration baseline.** Code migrated from the retired `deepseek-chat` (non-thinking) now gets thinking by default on `deepseek-v4-flash` — latency, cost, and the `reasoning_content` field all appear unrequested. Disable thinking or set effort `low` to approximate the old profile.
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

- **Local format instructions override session rules** on the turn where they appear; compliance returns next turn (§3). [field-observed]

- **CoT exhaustion returns empty content, not an error** (§4). [field-observed]

- **Sampling parameters are silently ignored in thinking mode** — temperature, top_p, and both penalties (§3).
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

- **Aggregator-surface variance is large.** [field-observed, 2026-08-09, OpenRouter endpoint metadata + probes] The 0731 slug is served by ~25 providers spanning fp8 to **fp4** quantization, 131K-1M context caps, and 32K-1M max-output caps; `response_format` acceptance varies by provider (one fp8 host 400'd even `json_object`, which the native API supports); penalties are inert on some hosts and active on others (measured 2026-07-19). DeepSeek's native `thinking: {"type":"disabled"}` body is **not honored** through OpenRouter — use OpenRouter's own `reasoning` controls there. Pin a provider (and check its quantization) before attributing any behavior to the model.

- **Instruction-following under roleplay/text-completion harnesses is reported harder on 0731** — one chat-harness user reports hours of prompt rework to restore instruction adherence. Narrow use case, marked as such. [community-reported]
[tier: 2, source: reddit.com/r/SillyTavernAI/comments/1vbp4x4/, retrieved 2026-08-09 via search snippet — Reddit blocks automated fetch; thread not independently re-verified]

- **Tool-call error rates differ by host for the same weights** — OpenRouter telemetry showed 1.78-5.39% on DeepSeek's own endpoint vs 0.23-0.72% on the best third-party hosts. Aggregator telemetry, not vendor data. [community-reported]
[tier: 2, source: openrouter.ai/deepseek/deepseek-v4-flash-0731, retrieved 2026-08-09]

- **MIT license.** No MAU clause, no regional restriction. Materially more permissive than Llama 4's Community License.
[source: huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731, retrieved 2026-08-09]

## 7. Anti-Patterns

- **Do not budget `max_tokens` small with thinking on.** The CoT spends from the same budget; the failure is a silent empty reply (§4). [field-observed]

- **Do not tune temperature/top_p/penalties in thinking mode.** They are silently ignored; with thinking on by default, wording and effort are the levers (§3).
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

- **Do not toggle thinking on/off mid-session when cache economics matter.** The toggle changes the rendered prefix and invalidates the cache; effort changes were cache-safe (§4). [field-observed]

- **Do not rewrite or compact history mid-session.** Prefix caching is automatic and unit-based; edits upstream of the append point forfeit the ~50x cache discount (§4).
[source: api-docs.deepseek.com/guides/kv_cache, retrieved 2026-08-09]

- **Do not drop `reasoning_content` on tool-carrying turns against the native API.** HTTP 400. OpenRouter's tolerance of stripping does not transfer (§3).
[source: api-docs.deepseek.com/guides/thinking_mode, retrieved 2026-08-09]

- **Do not expect standing session rules to survive terse-format turns without scoping.** Restate or scope the rule for bare-answer turns (§3). [field-observed]

- **Do not run unattended agent loops without supervision.** Community pattern: an advisor model watching for loops/rabbit-holes is the difference between "reliable and cheap" and "erratic." [community-reported]
[tier: 2, source: news.ycombinator.com/item?id=49214008, retrieved 2026-08-09]

- **Do not assume provider parity on aggregator routes.** Quantization (fp4 vs fp8), context caps, output caps, param support, and reasoning-control mapping all vary per routed provider (§6). Pin and verify. [field-observed]

- **Do not start or keep work on `deepseek-chat` / `deepseek-reasoner`.** Retired 2026-07-24; removed from the model enum.
[source: api-docs.deepseek.com/api/create-chat-completion, retrieved 2026-08-09]

- **Do not substitute ASCII `|` for the full-width `｜` in hand-built chat strings.** Wrong tokenization; use the shipped encoder (`deepseek-prompt-api.md` §2).
[source: huggingface.co/deepseek-ai/DeepSeek-V3.2, retrieved 2026-04-19]

- **Do not build for multimodal inputs.** Text-only family.

## 8. Gaps

- **Knowledge cutoff** — still undocumented for any V4 model (api-docs and the 0731 model card both silent).
- **All session/long-context field observations here are OpenRouter-surface** (Novita fp8 pinned; first-party endpoint unpinnable from the test account). Native-API session behavior — including cache-unit interaction with the thinking toggle — was not directly probed.
- **Degradation beyond ~237K context** is unmeasured here; the 1M window's upper region is unprobed.
- **Chat-style vs agent-loop long-session behavior** — the clean drift results are chat-style; agent-loop reliability is contested in Tier 2 reports (§4) and was not independently probed.
- **The consumer app/web surface was explicitly unchanged by the 0731 update** — nothing in this file's 0731 observations applies to chat.deepseek.com.
[source: api-docs.deepseek.com/updates, retrieved 2026-08-09]
- **`Developer` role full behavior** (V3.2 search-agent scenarios) remains undocumented in depth.
- **`deepseek-v4-pro` official release** was promised "soon" with several "early August 2026" surface promises unfulfilled as of 2026-08-09 — re-check before relying on pro-side claims.
