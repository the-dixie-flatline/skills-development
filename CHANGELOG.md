# Changelog

All notable changes to this repository are tracked here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The repository holds multiple skills; entries are scoped per skill (and occasionally to repository-level docs).

## Repository infrastructure — 2026-04-19

Initial public-release scaffolding:

- `.github/ISSUE_TEMPLATE/` — structured bug and coverage-request forms, plus a `config.yml` that disables blank issues, redirects security reports to private vulnerability reporting, and redirects general questions to Discussions. Both forms require Tier 1 primary sources; the bug form additionally prompts submitters to manually confirm a URL in a browser before reporting it as broken, since automated fetchers are frequently blocked by Cloudflare on `.gov` and similar primary sources.
- `.github/PULL_REQUEST_TEMPLATE.md` — PR template covering type-of-change, scope, provenance, data-handling self-check, AI-assistance declaration.
- `.gitignore` — editor / IDE state, Claude Code local state, OS metadata, Python and Node artifacts, secret-scanning reports, logs.
- `README.md` — includes a "Source Validation — Known Limitation" section documenting bot-blocker 403s on several federal primary sources and the manual-browser-verification fallback.
- `TODO.md` — open workflow items, principally the source-validation backlog (automated fetch 403s on Cloudflare-protected primary sources; preference for structured-data endpoints like the Federal Register JSON API where HTML surfaces are blocked; open questions on a maintainer-facing verification helper).

## prompt-engineering-skill 0.4.1 — 2026-07-19

Manual browser-verification fold-in. Thirteen bot-blocked/JS-rendered primaries were
verified by the maintainer in a signed-in browser (exact quotes, retrieved 2026-07-19;
evidence retained in the round-3 audit records) and folded into the skill. `SKILL.md`
bumped to `0.4.1`; contract-version unchanged (2026-04-18).

### Claude (`claude-prompt.md`, `claude-prompt-api.md`)

- **Claude Sonnet 5 added** (GA 2026-06-30, drop-in Sonnet 4.6 successor): IDs incl.
  Bedrock/Google Cloud forms (dateless ID = pinned snapshot), 1M context default-and-max
  with no beta header, 128K sync / 300K batch output (`output-300k-2026-03-24`), intro
  $2/$10 per MTok through 2026-08-31 then $3/$15, adaptive thinking on by default with
  `{type:"disabled"}` supported (manual `budget_tokens` 400), effort default `high`,
  sampling params 400, ~30% larger tokenizer at unchanged per-token price, no Priority
  Tier, real-time cybersecurity safeguards (HTTP 200 `stop_reason:"refusal"`).
- **`whats-new-claude-4-7` citations re-anchored** (~13; the URL now 301s to
  `whats-new-claude-4-8`): thinking-display default re-pointed to the Adaptive thinking
  page (default `display` "omitted" on Fable 5 / Mythos 5 / Sonnet 5 / Opus 4.8 / 4.7 /
  Mythos Preview — silent change from Opus 4.6 "summarized"); literalism,
  response-length-calibration, tokenizer-1.0-1.35x, and other 4.7-only claims kept scoped
  to Opus 4.7 with explicit historical / no-longer-vendor-documented markers.
- Effort `max`/`xhigh` availability pinned per model (xhigh not on Mythos Preview /
  Opus 4.6 / Sonnet 4.6; no beta header); full per-model prompt-cache minimum table
  (Sonnet 5 = 1,024 incl. Bedrock) and 12-row tool-use overhead table promoted to Tier-1.
- Last-turn prefill rejection pinned: 400 `invalid_request_error` with exact message;
  closes the prefill gaps; disable-thinking on Fable 5 / Mythos 5 / Mythos Preview stated
  as rejected/not-supported without asserting a status code (only the manual-enabled 400
  is vendor-pinned). New testable markers for prefill and Sonnet 5 sampling/thinking.
- **Vertex AI structured outputs corrected to Preview (Pre-GA), not GA**: supported =
  Opus 4.7 / Sonnet 4.6 / Opus 4.6; coming soon = Opus 4.5 / Sonnet 4.5 / Haiku 4.5;
  Fable 5 / Sonnet 5 / Opus 4.8 / Mythos Preview absent; `output_config.format` +
  `tools[].strict` mechanics and the org-policy gate documented.
- Mythos Preview retirement 2026-07-21 re-confirmed on the deprecations page; the
  competing 2026-06-30 date found no live Anthropic surface and no [disputed] tag ships.

### Qwen (`qwen-prompt.md`, `qwen-prompt-api.md`)

- 1M-vs-256K context figures resolved as two serving modes, not a conflict: standard
  pay-as-you-go API caps input at 1M (256K is a qwen3.7-plus pricing-tier boundary);
  dedicated deployment (Provisioned Throughput / Model Unit) caps the -2026-05 snapshots
  at 256K (128K for qwen3.6-plus / qwen3.5-plus snapshots) with documented automatic
  fallback to pay-as-you-go.
- Per-tier Model Studio pricing added (qwen3.7-max $2.5/$7.5 0-1M; qwen3.7-plus $0.4/$1.6
  and $1.2/$4.8; qwen3.6-plus $0.5/$3 and $2/$6; limited-time promos dated) plus cache
  rates (explicit creation 125% of standard input, hits 10%).
- qwen3.6-plus confirmed NOT scheduled for retirement (appears on the deprecation ledger
  only as a migrate-to reference; current batch dated 2026-10-10).

### Llama (`llama-prompt.md`)

- Closed the quote-level gap on the Llama 4 current-generation tag (literal "LATEST",
  captured from the live DOM); citation updated to the working `/ai/models/llama-4/` path
  with a client-side-rendering caveat.

## prompt-engineering-skill 0.4.0 — 2026-07-19

Round-3 refinement pass. `SKILL.md` bumped to `0.4.0`; contract-version
unchanged (2026-04-18). Tier-1 point facts re-verified against live vendor docs
on 2026-07-18 (firecrawl scrapes of platform.claude.com, ai.google.dev, and
developers.openai.com). Field-observed additions are abstracted public-model
behavior per repo Directive 2, carry `[field-observed]` markers with N=1-2 (N=15
for the grammar-decoding note), and state ranges not fixed numbers. Deferred
items (generational briefs F1/F2/F3/F11/F12/F30/F31/F32 and test-gated F10) are
NOT in this draft.

### Claude (`claude-prompt.md`, `claude-prompt-api.md`) — Tier-1

- **Prompt-cache minimums corrected** (F25): Opus 4.7 4096→2048, Sonnet 4.6
  2048→1024; added Opus 4.8=1024, Sonnet 5=1024, Fable 5 / Mythos 5=512 (1024 on
  Bedrock). Closed the §10 gap on the Fable 5 first-party minimum.
- **`frontier_llm` refusal category** (F26): the invisible-safeguards prose is
  replaced by the now-documented visible refusal category (cyber / bio /
  frontier_llm / reasoning_extraction), all fallback-eligible; updated in both
  files' §6/§9 and the §2 category list.
- **Per-model tool-use overhead table** (F27): the flat 346/313 figure (matching
  no current model) replaced with the per-model table (Opus 4.8 290/410, Opus 4.7
  675/804, Sonnet 5 354/474, etc.).
- **Deprecation/tense fixes** (F28): Opus 4 / Sonnet 4 retired 2026-06-15
  (completed); Opus 4.1 now Deprecated (retires 2026-08-05); Mythos Preview
  retires 2026-07-21; web_search now `web_search_20260318` (response_inclusion);
  Fable 5 `max` effort confirmed available; Vertex AI structured-outputs now GA
  including Mythos Preview (prior "Vertex does not support Mythos Preview" claim
  corrected).
- **Anti-pattern / placeholder** (F18a, F19): CAPS-emphasis anti-pattern
  generalized beyond the single tool-command string (behavioral breadth
  test-gated with a `[testable]` marker); §2 gains a fill-in-placeholder
  convention note (`{{value}}` / `[VALUE]` / backtick-wrap over bare
  `<angle-bracket>`).

### Gemini (`gemini-prompt.md`, `gemini-prompt-api.md`) — Tier-1

- **Implicit-caching floors corrected** (F29): 3.5 Flash 1024→4096, 2.5 Flash
  1024→2048, 2.5 Pro 4096→2048 (3.1 Pro unchanged at 4096); fixed both the API
  table and the prose "Flash=1024/Pro=4096" split.
- **gemini-3.5-flash context window** published (F34): 1,048,576 in / 65,536 out
  (partially closes the declared gap).
- **gemini-3.1-flash-lite shutdown** 2027-05-07 added (F28g).
- **Deep Research cross-reference** (F6): `gemini-prompt.md` gains a short DR
  subsection (enumeration-bucket fix + pointer to `deep-research-agents.md`).

### Deep Research (`deep-research-agents.md`) — field-observed (Gemini DR)

New "Field-observed behavior (Gemini Deep Research)" subsection plus inline
refinements: dual citation schema with `grounding-api-redirect` wrapper URLs
(F4); structural/suppression-honored vs inline-markup-ignored compliance split
(F5); enumeration bucket fix (F6); non-terminal failure modes — silent
400-zombie plus steps-hidden-until-terminal; the earlier bare-domain-literal
wedging correlation was refuted by controlled matched-pair testing 2026-07-19
and is withdrawn (F7, revised); observed usage-schema divergence, shown NOT
Api-Revision-dependent (F13, revised); realized Max cost ~$40 (F14); ~5-job
concurrency cap (F15); terminal `outputs` schema (F20); narrow source pool (F21);
adjacent-entity substitution + unreachable-source padding (F22); latency
re-measured 3.6-8.9 min narrow/standard N=11, prior 39-50 min attributed to
observation lag + scope/tier (F23, revised); integration facts (F24);
seed-suppresses-fabrication (F35).
OpenAI o*-deep-research shutdown tightened to imminent (F33, 5 days out); the
Anthropic web-search version refreshed to `web_search_20260318`.

### Qwen / OpenAI-compat (`qwen-prompt-api.md`, `openai-compatibility-surface.md`)

- **Grammar-constrained-decoding traps** (F8): §6 gains a `[field-observed]`
  (N=15) note — a grammar constrains emitted, not read, tokens (enum values must
  also appear in the prompt); a closed enum with no fallback member cannot fail
  safe (~10/15 confident near-miss). Recommends an explicit fallback member plus
  an open free-text companion field; carries a `[testable]` marker.
  Cross-referenced from the openai-compat vLLM/llama.cpp rows.

### Routing (`SKILL.md`)

- Same-family-consumer tie-breaker (F9); agent-orchestration trigger scoped to
  hosted vendor endpoints with a local/CLI-runner exclusion (F16); reuse clause
  extended to re-check `[applies-to]` version-scope, not just `retrieved:`
  freshness (F17); coverage-table Claude row refreshed for the deprecation/tense
  and refusal-category changes.

### Family expansion — 2026-07-19 (three new families, two updated, one disambiguated)

Six family lanes landed against live vendor docs retrieved 2026-07-19. Three new
families join the skill; two existing families take flagship/lineup updates; one
existing family gains a scope-disambiguation note. `SKILL.md` stays at
`0.4.0`; contract-version unchanged (2026-04-18). New provider sections in
`openai-compatibility-surface.md` carry their own 2026-07-19 sources, so that
file's top-level `retrieved:` (the 2026-06-01 sweep) is unchanged.

**New — GLM (Z.ai / zai-org).** New `resources/glm-prompt.md` and
`resources/glm-prompt-api.md` covering GLM-5.2 (753B MoE, IndexShare sparse
attention, 1M context, MIT open weights) plus GLM-5.1 / GLM-5-Turbo / GLM-4.7.
Documents the seven-value `reasoning_effort` ladder and its documented shim onto
two thinking tiers (low/medium→high, xhigh→max, none/minimal→skip), the
`thinking.type` toggle, dynamic-vs-forced-thinking split, XML tool-call wire
format, the dual `stream`+`tool_stream` streaming requirement, three-stream
deltas, the `clear_thinking` multi-turn contract with its endpoint-dependent
default, automatic prefix caching, Tier-1 pricing, and the Anthropic-compatible
endpoint (`api.z.ai/api/anthropic`) as a Claude Code drop-in. GLM-5.2 max output
carried as a documented doc-surface disagreement (docs.z.ai 128K vs HF card
163,840). Parallel tool calls and a cache-block minimum recorded as confirmed
documentation absences; the `glm-5.2[1m]` alias and specific
`ANTHROPIC_DEFAULT_OPUS_MODEL` / `CLAUDE_CODE_AUTO_COMPACT_WINDOW` values declared
as gaps pending canonical vendor confirmation.

**New — MiniMax.** New `resources/minimax-prompt.md` and
`resources/minimax-prompt-api.md` covering MiniMax-M3 (1M context, ~428B-total /
~23B-active MoE with Multi-Scale Attention, multimodal), the M2.7 / M2.5 / M2.1 /
M2 line (204,800 context, text + tool-call), and the `M2-her` chat model (64K). M3
open weights are published on HuggingFace under the MiniMax Community License
(non-commercial by default) — the marketing page's "coming soon" copy is stale.
API guidance centers on the vendor-recommended Anthropic-compatible
`/anthropic/v1/messages` path, with the OpenAI-compatible and legacy native
`chatcompletion_v2` surfaces documented secondarily. Covers thinking off-by-default
on M3 vs un-disable-able on M2.x, the multi-turn reasoning-preservation contract,
MSA 1M context with the 512K pricing-tier / guaranteed-floor semantics, and
automatic (passive) caching on M3 vs explicit `cache_control` on M2.x.

**New — Kimi (Moonshot AI).** New `resources/kimi-prompt.md` and
`resources/kimi-prompt-api.md` covering Kimi K3 (`kimi-k3`), three days post-launch:
2.8T-param KDA architecture, 1M context, always-on reasoning via `reasoning_effort`
(only `max` accepted today, "more levels coming soon"), five fixed sampling
parameters that error on deviation, the multi-turn complete-assistant-message
replay contract, `response_format` JSON Schema (MFJS) structured output, partial
(prefix-continuation) mode, K3-only dynamic tool loading (content-less `system`
message), the official tool set, and deprecations vs K2.x (K2.5 / moonshot-v1
sunset 2026-08-31, k2 previews discontinued 2026-05-25). OpenAI-compatible
(`api.moonshot.ai/v1`) and Anthropic-compatible (`api.moonshot.ai/anthropic`,
`kimi-k3[1m]`) surfaces; Kimi Code (`api.kimi.com/coding`) kept distinct as a
separate product. Open-weights sections (chat template, special tokens, license,
inference-stack flags) declared as gaps — weights announced for release by
2026-07-27, not yet published.

**Updated — Gemma.** Added the fifth Gemma 4 size, **12B Unified**
(`google/gemma-4-12B` / `-it`): dense, encoder-free multimodal, 11.95B total, 256K
context, added 2026-06-03. Now in the front-matter `versions:` list, the
model-selection table, and every size enumeration in both `gemma-prompt.md` and
`gemma-prompt-api.md`. Corrected the family-wide "audio is E-series only" claim:
audio-capable sizes are now E2B, E4B, **and 12B Unified**; 26B-A4B and 31B remain
text/image/video only. Added the encoder-free multimodal ingestion pipeline
(Tier-2, HF Transformers docs), tightened the empty-thinking-token stabilizer claim
to the exact instruction-tuned IDs, and added the `skip_special_tokens=False`
decode mandate (Tier-1). Resolved the exact-release-date gap (original four sizes
2026-03-31; 12B Unified 2026-06-03); declared new gaps (12B Unified decoder layer
count / vocab size; whether it ships a dedicated speculative-decoding draft model).
`retrieved:` bumped to 2026-07-19.

**Updated — Grok.** grok-4.5 (500K context) supersedes grok-4.3 as flagship
(grok-4.3 demoted to prior-gen, still live). Reasoning defaults inverted and split
by model: grok-4.5 `reasoning_effort` ∈ {low, medium, high}, default `high`, `none`
removed (reasoning cannot be disabled); grok-4.3 retains {none, low, medium, high},
default `low`. Every reasoning statement scoped per model. Flagged the
counterintuitive context regression (grok-4.5 500K < grok-4.3 1M). Scoped the
`grok.reasoning-effort-none-disables.v1` testable to grok-4.3 and added
`grok.reasoning-effort-none-rejected-45.v1` (grok-4.5 rejects
`reasoning_effort:"none"`). Added bifurcated pricing and the 200K repricing cliff,
the `prompt_cache_key` / `x-grok-conv-id` cache-pin split, corrected 2026-05-15
retirement redirect targets, and recorded the grok-4.5 knowledge-cutoff conflict
(models page Feb 1 2026 vs system card Jan 2026). SKILL.md Grok row + reasoning-
portability line and the `openai-compatibility-surface.md` Grok section updated.
`retrieved:` bumped to 2026-07-19.

**Disambiguated — Llama.** Added a Muse Spark scope note to `llama-prompt.md` and
`llama-prompt-api.md`. Muse Spark (Muse Spark 1.1, Meta Model API public preview
2026-07-09) is Meta Superintelligence Labs' current closed-weight, API-only
flagship — a separate brand/distribution track from the open-weight Llama line — and
is explicitly out of scope for the Llama files. No Muse family file created. No
Llama 4 factual changes: Scout / Maverick lineup, specs, license, and architecture
re-confirmed live 2026-07-19; `retrieved:` bumped to 2026-07-19 on both files.

**Shared-file integration.** `SKILL.md` gained coverage rows for GLM, Kimi, and
MiniMax; the Gemma / Grok / Llama rows and the reasoning-controls portability line
were updated to match the lane changes; the description-line family enumeration
gained GLM, MiniMax, Kimi. `openai-compatibility-surface.md` gained MiniMax, GLM,
and Kimi provider sections plus quick-matrix rows, and its Grok reasoning-control
line was split by generation.

## prompt-engineering-skill 0.3.0 — 2026-06-10

Claude Fable 5 launch coverage (`retrieved: 2026-06-10` on the two Claude reference
files; other families untouched). Contract-version unchanged (2026-04-18).

### Claude — new flagship: Claude Fable 5 (`claude-fable-5`, GA 2026-06-09)

- **`claude-prompt.md`** — Fable 5 added as the model-selection top row ($10/$50 per
  MTok, 1M ctx, 128K output, knowledge cutoff Jan 2026); Mythos 5 noted as the
  same-weights limited-release variant (Project Glasswing, successor to Mythos
  Preview). New instruction patterns from Anthropic's Fable prompting guide: brief
  steering instructions over per-behavior enumeration, give-the-reason framing,
  anti-overplanning, and tool-result-grounded progress claims. Behavioral-quirks
  block covers longer turns by default, effort guidance, over-elaboration at higher
  effort, unrequested actions, parallel-subagent readiness, memory-system uplift,
  rare early stopping, context-budget anxiety on visible token countdowns, long-run
  readability degradation, and the send-to-user-tool pattern. System-card failure
  clusters (886-use sample) and the strategic-deference finding included with
  provenance. New anti-patterns: no reasoning-echo instructions
  (`reasoning_extraction` refusals), no token countdowns, no wholesale carry-over of
  prior-model skills, no simple-workload-only evaluation.
- **`claude-prompt-api.md`** — Fable 5 platform IDs (Claude API, Bedrock incl.
  geo/global inference IDs and `bedrock-mantle`; Vertex/Foundry available but IDs
  unpinned); Covered-Model data-retention constraint (30-day, no ZDR; Bedrock
  `provider_data_share` opt-in). Thinking semantics: adaptive always on, `disabled`
  unsupported, `display` defaults `"omitted"`, raw chain of thought never returned.
  Sampling bounds per the Bedrock model card. New §2 subsection documenting the
  refusal response shape (`stop_reason: "refusal"`, `stop_details` categories
  cyber/bio/reasoning_extraction), billing, server-side `fallbacks` parameter (beta
  `server-side-fallback-2026-06-01`), fallback content blocks and `usage.iterations`,
  echo rules, streaming semantics, SDK refusal-fallback middleware, and fallback
  credit (`fallback-credit-2026-06-01`). Beta-header table and breaking-changes
  section extended; new testable IDs for thinking-disabled rejection, omitted-display
  default, and sampling rejection. Gaps declared for prefill handling, first-party
  cache minimum, Vertex/Foundry IDs, `allowed_fallback_models`, sticky-routing/billing
  subsections, and `effort: "max"` availability.
- **`SKILL.md`** — Claude coverage row and cross-family thinking-controls line
  updated for Fable 5; version bumped to 0.3.0.

## prompt-engineering-skill 0.2.0 — 2026-06-01

Family-lineup refresh against live primary sources (`retrieved: 2026-06-01`) plus a
scope expansion from "how to prompt model X" to "how to prompt, configure, and deploy
against model X's surfaces." Contract-version unchanged (2026-04-18).

### Urgent retirement-cliff corrections

- **Claude** — `claude-opus-4-20250514` / `claude-sonnet-4-20250514` retire **2026-06-15**;
  the Opus 4 replacement is corrected to `claude-opus-4-8`. `claude-3-haiku-20240307` is
  already Retired (April 20, 2026).
- **DeepSeek** — legacy `deepseek-chat` / `deepseek-reasoner` hard-retire **2026-07-24**
  (now map to `deepseek-v4-flash`); V3.2-Speciale's temporary API endpoint expired
  **2025-12-15** (weights remain).
- **Gemini** — 2.0 Flash / Flash-Lite GA shut down **2026-06-01**; 2.5 GA sunsets
  **2026-10-16**.
- **Grok** — the prior fast / 4.x / 3 slugs were retired **2026-05-15** and redirect to
  `grok-4.3` (or `grok-build-0.1` for `grok-code-fast-1`).

### Flagship and lineup moves

- **Claude** — added Opus 4.8 (`claude-opus-4-8`, GA 2026-05-28, 1M ctx, cutoff Jan 2026)
  as flagship; Opus 4.7 still Active; adaptive thinking is the only mode on 4.7/4.8 (manual
  budgets rejected). Sonnet 4.6 knowledge cutoff flagged `[disputed]` (Aug 2025 vs May 2025
  across two live Anthropic surfaces).
- **OpenAI** — GPT-5.5 (`gpt-5.5-2026-04-23`) flagship + 5.5-pro; GPT-5.4 retained as the
  cheaper tier; top Codex `gpt-5.3-codex`. Assistants API removed 2026-08-26, migration target
  corrected to **Responses + Conversations**; o\*-deep-research IDs shut down 2026-07-23;
  multi-agent handoffs documented as Agents-SDK-only.
- **Gemini** — 3.5 Flash GA + 3.1 Flash-Lite GA + 3.1 Pro Preview; `thinkingLevel` replaces
  `thinkingBudget` on Gemini 3; Computer Use isolated to `gemini-2.5-computer-use-preview-10-2025`.
- **DeepSeek** — V4 production (`deepseek-v4-flash` / `deepseek-v4-pro`, MIT, 1M ctx, 384K out);
  tri-state reasoning (Non-think / Think / Think Max, the last recommending ≥384K context);
  reasoning_content round-trip is now conditional on tool calls.
- **Qwen** — Qwen3.7-Max GA flagship; Qwen3.6-Plus demoted-but-GA; added dense open-weights
  `Qwen3.6-27B` alongside the `qwen3.6-35b-a3b` MoE.
- **Mistral** — Mistral Medium 3.5 (Modified MIT, 256K) frontier + Small 4; unified
  `reasoning_effort` is binary (`high`|`none`) on the chat endpoint; Magistral → Legacy.
- **Gemma** — Gemma 4 lineup re-verified (128K/256K context split; per-size dedicated draft model).
- **Llama** — re-confirmed; no Llama 5; Scout + Maverick current.

### New cross-family resources

- `resources/deep-research-agents.md` — hosted deep-research agents (Gemini, OpenAI,
  Perplexity async submit/poll) vs web-search tool-use loops (Anthropic, Grok).
- `resources/openai-compatibility-surface.md` — single-source matrix of per-provider
  divergences through OpenAI-shaped endpoints (Grok, DeepSeek, Gemini, Qwen, Mistral, vLLM,
  llama.cpp). Family `-api.md` files cross-reference it.
- `resources/agent-orchestration-surfaces.md` — Anthropic Managed Agents (300/600 limits),
  OpenAI hosted/SDK split, the three disaggregated Gemini agent products.
- `resources/webui-surfaces-and-silent-degradation.md` — tier-labeled; leads with the honest
  gap (consumer default reasoning effort undocumented). Detection methodology routed to
  `prompt-engineering-architect`.

### SKILL.md routing and discipline

- Empty-arg routing (#5), generated-prompt subject-vs-consumer routing (#7), additive
  mixed-task scope (#6), a third "surface" routing axis (#9), reuse-from-context freshness
  check (#8), a strengthened declared-Gaps stance (#10), and reading-discipline notes —
  weight measured behavior over benchmark priors; test a newer variant before switching (#16).
- Coverage table rewritten; cross-family resource rows added. SCHEMA.md gained a cross-family
  resource-file convention. Corrected the Cross-Family Portability section (the Grok
  "reasoning_effort = agent count" claim was false; DeepSeek's reasoning round-trip is conditional).

### Known limitations at 0.2.0

- Several family-file sections not touched by this refresh retain their 2026-04-18/19 inline
  provenance (re-verification covered lineup, reasoning, deprecation, and the new surfaces).
- Family knowledge cutoffs for current Qwen and all DeepSeek models remain undocumented (Gaps).
- A few REINSTATED candidate facts failed their author-time canonical confirm and were NOT
  shipped: a 65,536 output cap for Gemini Deep Research (that figure belongs to the Antigravity
  Agent) and xAI "DeeperSearch" (Tier-3 only).

## prompt-engineering-skill 0.1.0 — 2026-04-19

### Added

Initial public release. Family-specific prompt-engineering references for nine current-generation LLM families — Claude, OpenAI (GPT-5.x / o-series), Gemini, Gemma, Llama, Qwen, Grok, Mistral, DeepSeek.

- `SKILL.md` — Routing layer with coverage table, reading discipline, and a Cross-Family Portability section enumerating the six most common ways prompts break when ported between families: reasoning-parameter naming differences, reasoning-artifact multi-turn rules (actively contradictory), role naming, "OpenAI-compatible" wire-format vs semantic divergence, open-weights chat-template hand-assembly hazards, dated model-ID pinning.
- `SCHEMA.md` — Normative schema for family reference files: front-matter shape, section order, source tiering (Tier 1 / 2 / 3), inline markers (`[applies-to]`, `[testable]`, `[unverified]`, `[disputed]`), required workflow.
- `README.md` — Human-facing framing, audience, source policy summary, known limitations.
- `resources/` — Nine pairs of reference files (prompt-layer + API-layer per family) with inline provenance per claim, a `retrieved:` date, declared version scope, and an honest `Gaps` section.

Contract-version pinned to 2026-04-18.

### Known limitations at 0.1.0

- Test harness not yet built; `[testable: ...]` claims are tagged but not asserted.
- Depth varies by family based on provider documentation density — families with denser public docs (Anthropic, OpenAI, Qwen) support deeper references than those with thinner coverage (Grok, Mistral function-calling).
- Point-in-time snapshot. Every file has a `retrieved:` date; content older than 90 days without re-verification should be treated as stale per repository contract.
- Gaps declared per file rather than silently omitted. Files under-documented by their providers carry larger Gaps sections.

## domain-research-skill 0.1.0 — 2026-04-19

### Added

Initial public release. A routing skill that layers domain-specific source taxonomies on top of a fixed, domain-neutral `Research_Protocol`. Thirteen domains covered at 0.1.0:

1. Technical research (AI/ML, infrastructure, software engineering)
2. GovCon proposal writing — US federal solicitation response
3. Federal capture & contract vehicle research — pre-solicitation
4. Small business set-aside & certification — HUBZone / 8(a) / WOSB / SDVOSB / SDB
5. State & local government procurement — state portals and cooperative vehicles
6. Nonprofit sector research — IRS 990s, foundations, grant databases
7. Salesforce integration architecture — release-versioned platform work
8. Healthcare data compliance — HIPAA / HITECH / state variants
9. Accessibility & Section 508 conformance — WCAG 2.2 / ISO/IEC 40500:2025, VPATs, DOJ Title II IFR (2026)
10. Web security & WAF configuration — CVE / NVD / OWASP Top 10 2025 / CISA KEV / CRS
11. AI/ML model & hardware evaluation — reproducible benchmarks (MLPerf Inference v6.0 etc.), not marketing
12. Japanese legal & regulatory — e-Gov法令検索, 判例, 官報 (electronic-primary), bilingual handling
13. Federal contractor cybersecurity — CMMC (32 CFR Part 170), NIST SP 800-171, DFARS cyber clauses

Files:

- `SKILL.md` — Routing layer with the locked `Research_Protocol` inline, activation logic, routing table by domain, multi-domain overlap handling (federal cluster), output protocol, deterministic-retreat procedure, coverage status, tuning rubric for authoring new domains.
- `README.md` — Human-facing framing, rationale for splitting invariant protocol from variable taxonomy, known limitations.
- `resources/` — One file per domain with a populated `<Source_Domain>` block (`Tier_1_Primary`, `Tier_2_Contextual`, `Disqualified`, `Evidence_Threshold`) naming canonical databases, registries, and noise patterns specific to that domain.

Several high-churn domains carry a **Validator Note — Post-Training-Cutoff Authority Changes** subsection documenting recent authoritative changes with primary-source citations so that audit passes do not mistake post-cutoff changes for fabrications. Covered changes include EO 14347 "Restoring the United States Department of War," the DFARS Case 2019-D041 Final Rule, FPDS.gov and eSRS.gov decommissionings, the DOJ ADA Title II Interim Final Rule extending compliance dates, elimination of SDVOSB / VOSB self-certification under NDAA 2024, the Japanese Kanpo electronic-publication transition, and the APPI 2025/2026 reform cycle.

Contract-version pinned to 2026-04-18.

### Known limitations at 0.1.0

- US-centric coverage for federal and state-procurement domains. Non-US public procurement is not covered.
- Healthcare domain is compliance-oriented; clinical / evidence-based-medicine research is out of scope.
- AI/ML evaluation domain describes how to accept or reject benchmark claims; it does not produce rankings or buying guides.
- Multi-domain overlap is handled by the union of taxonomies rather than precomputed overlap patterns.
- **Automated primary-source verification is partial.** Several primary sources this skill cites (`.gov` properties, `ecfr.gov`, `federalregister.gov` HTML pages, `salesforce.com`, some `iso.org` pages) sit behind Cloudflare / bot protection and return 403 to automated fetchers. Where possible the resource files cite a structured-data alternative (e.g., the Federal Register JSON API at `federalregister.gov/api/v1/documents/{doc-number}.json`); where none exists, manual-browser verification is the fallback. See repo-root `TODO.md` for the validation-workflow backlog.
- Point-in-time snapshot. Files older than 90 days without re-verification should be treated as stale. Authoritative-source lists change on regulatory, legislative, and programmatic cycles; authorities active at 0.1.0 include the DFARS Case 2019-D041 Final Rule (effective 2025-11-10), the CMMC Program Final Rule (effective 2024-12-16), NIST SP 800-171 Rev. 3 (2024-05-14), NIST SP 800-66 Rev. 2 (2024-02), and the DOJ ADA Title II IFR (effective 2026-04-20).
