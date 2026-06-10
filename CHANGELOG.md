# Changelog

All notable changes to this repository are tracked here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The repository holds multiple skills; entries are scoped per skill (and occasionally to repository-level docs).

## Repository infrastructure — 2026-04-19

Initial public-release scaffolding:

- `.github/ISSUE_TEMPLATE/` — structured bug and coverage-request forms, plus a `config.yml` that disables blank issues, redirects security reports to private vulnerability reporting, and redirects general questions to Discussions. Both forms require Tier 1 primary sources; the bug form additionally prompts submitters to manually confirm a URL in a browser before reporting it as broken, since automated fetchers are frequently blocked by Cloudflare on `.gov` and similar primary sources.
- `.github/PULL_REQUEST_TEMPLATE.md` — PR template covering type-of-change, scope, provenance, data-handling self-check, AI-assistance declaration.
- `.gitignore` — editor / IDE state, Claude Code local state, OS metadata, Python and Node artifacts, secret-scanning reports, logs.
- `README.md` — includes a "Source Validation — Known Limitation" section documenting bot-blocker 403s on several federal primary sources and the manual-browser-verification fallback.
- `TODO.md` — open workflow items, principally the source-validation backlog (automated fetch 403s on Cloudflare-protected primary sources; preference for structured-data endpoints like the Federal Register JSON API where HTML surfaces are blocked; open questions on a maintainer-facing verification helper).

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
