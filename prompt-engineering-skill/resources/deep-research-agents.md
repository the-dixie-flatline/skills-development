---
family: cross-family
scope: deep-research-agents
families: [gemini, openai, perplexity, anthropic, grok]
retrieved: 2026-07-19
primary_sources:
  - https://ai.google.dev/gemini-api/docs/interactions/deep-research
  - https://ai.google.dev/gemini-api/docs/interactions
  - https://ai.google.dev/gemini-api/docs/interactions-overview
  - https://ai.google.dev/api/interactions-api
  - https://ai.google.dev/gemini-api/docs/tokens
  - https://developers.openai.com/api/docs/guides/deep-research
  - https://developers.openai.com/api/docs/deprecations
  - https://docs.perplexity.ai/docs/sonar/models/sonar-deep-research
  - https://docs.perplexity.ai/docs/cookbook/articles/async-deep-research/README
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool
  - https://x.ai/news/grok-3
  - https://docs.x.ai/developers/tools/web-search
maturity_note: |
  This resource covers hosted deep-research agents across providers. Two
  structurally different shapes exist: (A) async hosted agents you submit and
  then poll (Gemini, OpenAI, Perplexity), and (B) synchronous web-search
  tool-use loops that are not managed async agents (Anthropic, xAI Grok). Every
  agent and model ID below is preview or otherwise volatile and rotates without
  notice; treat all IDs as date-stamped to the retrieval date and re-verify
  before pinning. A "Cross-Agent Behavioral Observations" section (added
  2026-06-02) records recurring Tier-2 (academic/community) behaviors that span
  agents; those claims carry their own [community-reported] markers and
  2026-06-02 retrieval dates, distinct from the Tier-1 vendor facts above.
  Vendor Tier-1 facts are dated 2026-06-01 unless a later inline date appears;
  a 2026-07-18 pass added a "Field-observed behavior (Gemini Deep Research)"
  subsection (N=1-2 first-party observations of the public hosted agent,
  abstracted to model/API behavior, carrying [field-observed] markers and
  sample-size caveats), refreshed the Anthropic web-search tool version, and
  tightened the OpenAI deep-research shutdown proximity. A 2026-07-19 pass
  reconciled the Gemini section against the now-GA Interactions API: the terminal
  response schema is the vendor-documented `steps` array (not a flat `outputs`
  list), `agent_config` fields and the 55-day/1-day retention window are promoted
  to Tier-1, the documented usage list is trimmed to the 6-key core schema, SDK
  floors move to >= 2.3.0, and the OpenAI deep-research shutdown date is pinned to
  2026-07-23 (the earlier Dec-11 reading conflated it with the base o3-2025-04-16
  snapshot).
---

# Deep Research Agents — Cross-Family Reference

Hosted research agents do not share one shape. Two are documented here, and they are not interchangeable.

- **Shape A — async hosted agents.** Submit a request, get an identifier, poll for a terminal state, read the result. The provider runs a multi-step research loop server-side, sometimes for tens of minutes. Gemini Deep Research, OpenAI Deep Research, and Perplexity Sonar Deep Research are in this shape.
- **Shape B — web-search tool-use loops.** A single synchronous model request in which the model decides to search, the API executes searches inline, and the loop iterates within that one request. There is no separate agent resource to submit to or poll. Anthropic and xAI Grok deliver agentic web research this way through server-side tools.

Pick the shape first. The integration code, latency profile, and citation surface differ between them. Do not template a Shape B tool call as if it were a Shape A submit/poll job.

## Shape A: Async Hosted Agents (submit -> poll)

### Gemini Deep Research

**Agent IDs.** `deep-research-preview-04-2026` (fast tier) and `deep-research-max-preview-04-2026` (max tier), both Public Preview — these are the two agent rows on the current supported-models table. The older `deep-research-pro-preview-12-2025` is no longer listed on the interactions overview page but still appears in the `agent` enum and worked example on the API reference page, so it likely still resolves; treat it as stale/legacy and pin the current IDs. All are preview IDs and rotate.
[source: https://ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]
[source: https://ai.google.dev/api/interactions-api, retrieved 2026-07-19]

**Submit.** `POST /v1beta/interactions` (SDK `client.interactions.create`). Required body fields: `input`, `agent`. `background: true` is mandatory — the docs state "Agents are required to use background=True". `store: true` is required; `store=false` is incompatible with `background=true`.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]
[source: https://ai.google.dev/api/interactions-api, retrieved 2026-06-01]

**Poll.** `GET /v1beta/interactions/{id}` (SDK `interactions.get(id)`). Loop on `.status`.
[source: https://ai.google.dev/api/interactions-api, retrieved 2026-06-01]

**Terminal response.** The completed report's text is read from the last step of the `steps` array — `interaction.steps[-1].content[0].text` — not a separate `outputs` list. This is the vendor-documented Interactions terminal shape.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-07-19]

**Status enum.** Full set: `in_progress`, `requires_action`, `completed`, `failed`, `cancelled`, `incomplete`, `budget_exceeded`. Terminal success/failure values are lowercase `completed` / `failed` — not `SUCCEEDED`. Do not match on uppercase or on Perplexity-style status strings.
[source: https://ai.google.dev/api/interactions-api, retrieved 2026-06-01]
[testable: id=gemini.deep-research-status-lowercase.v1, expected=a completed interaction returns status string "completed", not "SUCCEEDED"]

**Time budget.** Maximum research time is 60 minutes; most tasks finish within roughly 20 minutes.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

**Structured outputs.** Not supported — "The Deep Research Agent currently doesn't support structured outputs." Steer output shape with formatting instructions in the prompt, not a schema.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

Instruction compliance splits by class: **structural** and **suppression** instructions are honored — a "no charts/images" instruction zeroed a default embedded PNG, and section layout / negative-result reporting followed — but **inline per-claim machine-readable markup** (e.g. "tag every claim `[primary]`/`[secondary]`") is NOT honored (zero literal tags emitted). Put load-bearing shape in structural or suppression form; treat inline per-claim markup as best-effort only. [field-observed, N=2 before/after; hosted Gemini Deep Research]

**Usage and cost.** The core Gemini API tokens page documents a 6-key `usage` schema: `total_input_tokens`, `total_output_tokens`, `total_thought_tokens`, `total_cached_tokens`, `total_tool_use_tokens`, and `total_tokens`. (The `cached_tokens_by_modality` and `grounding_tool_count` fields appear on the Interactions API `Usage` schema reference but not on the core tokens page; see the field-observed GET-time note below.) There is no monetary cost field in the response; cost is a narrative estimate only — roughly $1–3 per task for Deep Research and roughly $3–7 per task for Deep Research Max.
[source: https://ai.google.dev/gemini-api/docs/tokens, retrieved 2026-07-19]
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

The **observed GET-time usage schema diverges from the documented field list and is NOT `Api-Revision`-dependent.** Controlled re-GETs of terminal interactions with and without the header returned an identical 8-key `usage` object — `total_cached_tokens`, `input_tokens_by_modality` / `output_tokens_by_modality`, and the `total_*` counters — while the documented `cached_tokens_by_modality` / `grounding_tool_count` fields were absent in both cases. Do not hard-code the documented field names; parse the `usage` object defensively. [field-observed, N=2 controlled re-GETs, 2026-07-19, superseding an earlier N=2 header-dependence reading]

Realized cost can run **far above the nominal band.** One Max-tier task realized roughly **$40** (~17M tokens), versus the $3–7 narrative estimate. Budget from measured end-to-end token usage, not the headline band. [field-observed, N=1]

**Tools.** When `tools` is omitted the defaults are `google_search`, `url_context`, and `code_execution`. Optional additions: `mcp_server` and `file_search`. Custom function-calling tools are not supported — remote MCP only.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

**Agent config.** `agent_config` fields, all vendor-documented: `type` (`"deep-research"`, required — selects the agent); `thinking_summaries` (`auto` / `none`, default `none`); `visualization` (`auto` / `off`, default `auto`); `collaborative_planning` (boolean, default `false`). Use `previous_interaction_id` to refine or follow up on a prior interaction.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-07-19]

**Streaming.** Streaming requires `stream=True` and `background=True` together. Set `thinking_summaries: "auto"` to receive thoughts. `step.delta` deltas carry `thought`, `text`, or `image` content. Reconnect a dropped stream with `?stream=true&last_event_id=` (distinct from `previous_interaction_id`, which chains a new interaction onto a prior completed one). The examples send an `Api-Revision` header of `2026-05-20`. Interactions API (GA) SDK floors: `google-genai >= 2.3.0`, `@google/genai >= 2.3.0`. Storage retention is **55 days on paid tiers and 1 day on the free tier**.
[source: https://ai.google.dev/gemini-api/docs/interactions-overview, retrieved 2026-07-19]
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

Do not assert an output-token cap for Gemini Deep Research. The 65,536-token cap that circulates for Gemini agents belongs to the separate Antigravity Agent, documented in `resources/agent-orchestration-surfaces.md`, not to Deep Research. See Gaps.

#### Field-observed behavior (Gemini Deep Research)

First-party observations of the public hosted agent, abstracted to model/API behavior. All are `[field-observed]`, N=1-2 unless noted; treat as ranges, not vendor-grade numbers. Naming the behavior is in scope here; detection/mitigation methodology routes to the `prompt-engineering-architect` skill.

- **Citation surface (dual schema).** Span-aware `url_citation` annotations `{start_index, end_index, url}` appear **only on the lead segment** of the report; the annotation objects carry **no `title`** (unlike the documented OpenAI shape). Every annotation `url` is an opaque `vertexaisearch.cloud.google.com/grounding-api-redirect/...` wrapper (observed 92/92 and 126/126 across two completions), never a direct source URL — it must be **dereferenced** before verification. The remaining ~73% of body length cites via inline `[cite: N]` markers with no annotation objects and no in-band N→source mapping. An annotations-only parser silently drops most citations. [field-observed, N=2]

- **Enumeration "bucket" failure is prompt-correctable.** On "list all X" prompts the agent names a few items and collapses the rest into an explicit "others / various / unlisted" bucket. One directive eliminated bucket language entirely in before/after runs: "a bucket is a failed answer; enumerate every item individually; if an item is confirmed to exist but cannot be detailed, still list it and flag it as undetailed." [field-observed, N=1-2 before/after]

- **Non-terminal failure modes (two known).** (1) *Silent worker death:* roughly 3 minutes after submission the interaction's GET begins returning an opaque `400 invalid_request` permanently; no terminal status is ever reachable, though `DELETE` still returns 200. Observed in 2 of 6 batched submissions on one day (0 of 7 solo); prompt-content-independent. Poll loops must treat a persistent 4xx on a previously-readable interaction as terminal-equivalent, or they will wait forever while holding a concurrency slot. (2) *Apparent stalls are a client-side misread:* the `steps` array exposes **no model steps until the job is terminal**, so "no model progress after T minutes" is not a liveness signal — a stall-kill heuristic with T below the job's natural runtime kills healthy jobs deterministically. An earlier observed correlation between bare domain/URL literals in exclusion lists and wedging did **not** survive controlled matched-pair testing (3 URL/prose pairs plus the original wedge-correlated prompt verbatim: 7/7 completed when submitted solo through an independent path) and is withdrawn. [field-observed, N=13 controlled + 6 historical, 2026-07-19]

- **~5-job account concurrency cap.** Roughly five concurrent jobs per account; over-cap jobs park rather than queueing cleanly, and neither `status` nor step content is a liveness signal (steps appear only at terminal state; a parked job reports non-terminal indefinitely). Batched submission *at* the cap showed no latency penalty versus solo (N=4 vs N=7). Issuing `DELETE` on an interaction frees a slot. [field-observed, N=1-2 for the cap itself]

- **Narrow source pool.** The agent deep-exploits a small set (~5 sources observed) where a competitor system reached ~17 on the same prompt — strong authority, weak recall. Do not treat a Gemini DR report as an exhaustive sweep; pair it with a broader-recall pass when coverage matters. [field-observed, N=1-2]

- **Adjacent/successor-entity substitution and unreachable-source padding.** Observed substituting an adjacent or successor entity for the asked-about one, and padding an unreachable citation with a different (wrong) source rather than reporting the gap. Verify entity identity and dereference every cited source before relying on a claim. [field-observed, N=1-2]

- **Latency is scope- and tier-dependent; measure with continuous polling.** Narrow single-table prompts on the standard agent completed in **3.6–8.9 minutes** end-to-end (N=11, continuous ~20 s polling), identical solo and batched four-wide at the ~5-job concurrency cap. A previously recorded 39–50 minute envelope (N=2, Max agent, broad multi-part prompts) was not reproduced and included polling-observation lag — completion timestamps were recorded at the next poll pass, not at service completion. Treat the vendor's "~20 min typical" as scope- and tier-dependent, not a bound in either direction. [field-observed, N=11, 2026-07-19]

- **Integration facts.** Auth via `x-goog-api-key` header; poll cadence ~20s; interaction ids are resumable; omitting `store` succeeds and defaults it to `true` (consistent with the documented `store` default); the `Api-Revision` header is accepted and documented as schema-pinning, but omitting it produced no observable difference at terminal GET (see the usage-object note above). Completed interactions remain retrievable within the documented retention window (55 days paid / 1 day free), which subsumes the earlier `>= 30-minute` post-completion observation. [field-observed, N=1-2, corroborated against the interactions-API docs]

- **Seeding pre-verified facts suppresses a recurring fabrication.** Supplying already-verified facts as fixed inputs suppressed a recurring fabrication and improved recall on the seeded entities. DR fabrications are **run-variable** — absence in one run is not a safety guarantee. (The generic verify / quote-or-abstain discipline is methodology and routes to `prompt-engineering-architect`; only the model-behavior half is recorded here.) [field-observed, N=1-2]

### OpenAI Deep Research

**Models and invocation.** `o3-deep-research` and `o4-mini-deep-research`, called through the Responses API in background mode. Configure a webhook to be notified on completion rather than polling indefinitely.
[source: https://developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]

**Imminent shutdown — flag this.** The `o3-deep-research-2025-06-26` / `o4-mini-deep-research-2025-06-26` snapshots (and their `o3-deep-research` / `o4-mini-deep-research` aliases) shut down **2026-07-23**, with `gpt-5.5-pro` named as the replacement — that is **4 days out as of 2026-07-19**, and the deep-research guide's own samples still reference the retiring IDs. This 2026-07-23 date is a distinct deprecation row from the base `o3-2025-04-16` snapshot (which retires 2026-12-11); do not conflate the two. Do not build against `o*-deep-research` now; verify the successor invocation (`gpt-5.5-pro` via the Responses API) against the current guide before writing any new integration.
[source: https://developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]
[source: https://developers.openai.com/api/docs/deprecations, retrieved 2026-07-19]

**Data source requirement.** At least one data source is required: web search, remote MCP servers, or file search backed by vector stores. Supported tools: web search, file search, remote MCP, and code interpreter. Function calling is not supported.
[source: https://developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]

**Citations.** Returned as `annotations` on the final message `output_text`, each with `url`, `title`, `start_index`, and `end_index`.
[source: https://developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]

**Prompt handling.** The API does not auto-clarify or rewrite prompts; it expects a fully-formed prompt. The docs suggest an optional separate `gpt-4.1` step to clarify or rewrite the prompt before submission.
[source: https://developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]

**Documented limits.** `max_tool_calls` caps cost and latency. Maximum of 2 vector stores. Remote MCP `require_approval` must be set to `never`.
[source: https://developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]

### Perplexity Sonar Deep Research

**Model.** `sonar-deep-research`.
[source: https://docs.perplexity.ai/docs/sonar/models/sonar-deep-research, retrieved 2026-06-01]

**Sync.** `POST https://api.perplexity.ai/v1/sonar` (messages API).
[source: https://docs.perplexity.ai/docs/sonar/models/sonar-deep-research, retrieved 2026-06-01]

**Async.** Submit with `POST https://api.perplexity.ai/v1/async/sonar`; retrieve with `GET https://api.perplexity.ai/v1/async/sonar/{request_id}`. Async statuses include `CREATED`. Note that these are uppercase, unlike Gemini's lowercase enum — do not share status-matching code across the two.
[source: https://docs.perplexity.ai/docs/cookbook/articles/async-deep-research/README, retrieved 2026-06-01]

**Reasoning effort.** `reasoning_effort`: `low` / `medium` (default) / `high`.
[source: https://docs.perplexity.ai/docs/sonar/models/sonar-deep-research, retrieved 2026-06-01]

**Citations.** Returns a `citations` array of URLs plus a structured `search_results` array with `title` and `url`. Context window is 128K.
[source: https://docs.perplexity.ai/docs/sonar/models/sonar-deep-research, retrieved 2026-06-01]

**Pricing.** $2 per 1M input tokens, $8 per 1M output tokens, $2 per 1M citation tokens, $5 per 1K search queries, and $3 per 1M reasoning tokens.
[source: https://docs.perplexity.ai/docs/sonar/models/sonar-deep-research, retrieved 2026-06-01]

**Throttling.** Keep concurrent deep-research requests to 3–5 to avoid throttling.
[source: https://docs.perplexity.ai/docs/cookbook/articles/async-deep-research/README, retrieved 2026-06-01]

**Agent API preset.** The Agent API also exposes a `deep-research` preset, called as `client.responses.create(preset="deep-research")`. This is distinct from calling `model="sonar-deep-research"` directly; they are two different entry points to the capability.
[source: https://docs.perplexity.ai/docs/sonar/models/sonar-deep-research, retrieved 2026-06-01]

**Realized cost runs well above the nominal output rate.** Third-party telemetry reports the *realized* output cost on `sonar-deep-research` far above the nominal $8 / 1M output rate — on the order of ~$120 / 1M output tokens (roughly 15x) — because reasoning, citation, and search-query billing stack on top of base output. Budget from measured end-to-end usage, not the headline output price. [community-reported]
[tier: 2, source: openrouter.ai/perplexity/sonar-deep-research, retrieved 2026-06-02]
[source: https://docs.perplexity.ai/docs/sonar/models/sonar-deep-research, retrieved 2026-06-01]

**Citations come back on the response payload, not through a tool-call channel.** The `citations` and `search_results` objects (above) ride on the response itself; an integration that does not explicitly read them off the response surfaces no sources. A reproducible third-party client bug turned on exactly this omission. [community-reported]
[tier: 2, source: github.com/danny-avila/LibreChat/issues/9005, retrieved 2026-06-02]

A GA / beta / preview lifecycle label is not stated on the model page. See Gaps.

## Shape B: Web-Search Tool-Use Loops (not managed async agents)

These are synchronous server-side tools, not async agent resources. The model iterates searches inside one request. There is nothing to poll.

### Anthropic

There is no separately branded async "Research" API agent. Agentic web research is delivered through the `web_search` server tool: Claude decides when to search, the API runs the searches, and the loop may iterate within a single Messages request. The optional code-execution tool enables dynamic filtering. For long-running managed work, Managed Agents include a built-in web search and fetch tool, documented in `resources/agent-orchestration-surfaces.md`.
[source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool, retrieved 2026-06-01]

**Tool types.** Current latest is `web_search_20260318` (adds a `response_inclusion` parameter); `web_search_20260209` (supports dynamic filtering; requires the code-execution tool) and `web_search_20250305` remain available. Citations are always on, returned as `web_search_result_location` blocks with `url`, `title`, and `cited_text` up to 150 characters. Pricing is $10 per 1,000 searches. An org admin must enable the tool in the Console before use.
[source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool, retrieved 2026-07-18]

### xAI Grok

"DeepSearch" is a consumer-app agent on grok.com and X, documented only in Grok-3 launch product copy. It is not a developer-API async agent.
[source: https://x.ai/news/grok-3, retrieved 2026-06-01]

The developer-API analog is the server-side Web Search / X Search tools (`web_search`), a synchronous tool-use loop that returns `response.citations`.
[source: https://docs.x.ai/developers/tools/web-search, retrieved 2026-06-01]

Do not assert "DeeperSearch" as a distinct named developer feature. It is not documented in any Tier 1 source and is omitted here.

## Comparison Framing

| Provider   | Shape | Entry point                                    | Completion model        | Citation surface                  |
|------------|-------|------------------------------------------------|-------------------------|-----------------------------------|
| Gemini     | A     | `POST /v1beta/interactions` (`background=true`)| Poll `.status`          | `url_citation` annotations (lead segment only; `grounding-api-redirect` wrapper URLs) + inline `[cite: N]` in the body — see field-observed |
| OpenAI     | A     | Responses API, background mode                 | Webhook on completion   | `annotations` on `output_text`    |
| Perplexity | A     | `/v1/async/sonar` (or sync `/v1/sonar`)        | Poll `{request_id}`     | `citations` + `search_results`    |
| Anthropic  | B     | `web_search` server tool in Messages request   | Synchronous, in-request | `web_search_result_location`      |
| xAI Grok   | B     | `web_search` server tool                       | Synchronous, in-request | `response.citations`              |

Shape A decouples submission from result: budget for minutes of latency, persist the identifier, and reconnect or webhook to the result. Shape B returns within one request: no identifier to persist, no terminal-state enum, latency bounded by the model's own search loop. Status-string conventions differ even within Shape A — Gemini uses lowercase (`completed`), Perplexity uses uppercase (`CREATED`) — so status-handling code does not port across providers.

## Cross-Agent Behavioral Observations

These behaviors recur across more than one hosted deep-research agent and are recorded here rather than duplicated into each provider section. All are Tier-2 (academic or community) and flagged `[community-reported]`; none is a vendor guarantee. This section names the behavior only — detection and mitigation methodology is out of scope and routes to the `prompt-engineering-architect` skill.

**Structured-repository blind spot.** Hosted deep-research and web agents traverse the open prose web (news, articles, vendor pages, search-result snippets) well, but reach *structured* authoritative data layers far less reliably: data behind public REST APIs, bulk datasets, and database- or JavaScript-backed query UIs. A controlled WebArena study found agents given API access scored more than 24 points higher (absolute success rate) than browsing-only agents on the same tasks, isolating how much of the structured layer browsing alone leaves behind. A finance-research benchmark that required SEC EDGAR access put even the best agent (OpenAI o3) at 46.8% accuracy. Operational consequence: when a question's primary evidence lives behind an API or database UI, the agent can silently omit it; pre-fetching that layer separately is more reliable than expecting the agent to reach it. [community-reported]
[tier: 2, source: arxiv.org/abs/2410.16464 (Beyond Browsing: API-Based Web Agents), retrieved 2026-06-02]
[tier: 2, source: arxiv.org/abs/2508.00828 (Finance Agent Benchmark), retrieved 2026-06-02]

**Agents diverge, and a single agent is not self-sufficient on citations.** Different hosted agents ground and cite the same question very differently — one public leaderboard records Gemini 2.5 Pro Deep Research at ~111 average effective citations, described as exceptional relative to other systems — so two agents on one prompt are not redundant runs of the same process. And no single agent is dependable on citation faithfulness alone: a long-standing citation benchmark found even the best models lack complete citation support roughly half the time. Treat a primary-source re-fetch and quote-match as load-bearing, not optional, before relying on any one agent's report. [community-reported]
[tier: 2, source: deepresearch-bench.github.io (DeepResearch Bench), retrieved 2026-06-02]
[tier: 2, source: aclanthology.org/2023.emnlp-main.398 (ALCE), retrieved 2026-06-02]

**Long reports collect more than they synthesize.** Benchmarks built specifically for long-form research generation find current agents far from saturated on synthesis quality. A live benchmark evaluating 17 frontier deep-research systems reports recurring synthesis-stage failure modes, and a generative-research-synthesis benchmark found no system exceeds a ~31% geometric mean across knowledge-synthesis, retrieval-quality, and verifiability metrics. A long, fluent report is not evidence that cross-source synthesis actually happened — verify the synthesis, not just the length. [community-reported]
[tier: 2, source: arxiv.org/abs/2510.14240 (LiveResearchBench), retrieved 2026-06-02]
[tier: 2, source: arxiv.org/abs/2508.20033 (DeepScholar-Bench), retrieved 2026-06-02]

## Gaps

- **Gemini Deep Research output-token cap** — not documented for this agent. The 65,536-token figure belongs to the separate Antigravity Agent (see `resources/agent-orchestration-surfaces.md`), not Deep Research.
- **Gemini `agent_config` runtime effects** — the field names and defaults are now documented (Tier-1), but the observable runtime effect of `visualization` and `collaborative_planning`, and how `thinking_summaries` interacts with streaming, are not detailed at `ai.google.dev/gemini-api/docs/interactions/deep-research`, checked 2026-07-19.
- **OpenAI deep-research post-migration shape** — how the replacement `gpt-5.5-pro` is invoked for deep research after the 2026-07-23 shutdown of the `o*-deep-research` IDs is not covered by the current guide.
- **Perplexity lifecycle label** — no GA / beta / preview status is stated on the `sonar-deep-research` model page.
- **Perplexity async polling cadence and timeout** — recommended poll interval and maximum job lifetime are not specified beyond the 3–5 concurrent-request guidance.
- **Anthropic / xAI managed async research** — neither provider exposes a documented async "Research" agent resource in the sources retrieved; this file covers only their synchronous web-search tools and (for Anthropic) cross-references Managed Agents.
- **xAI DeepSearch developer API** — the consumer-app DeepSearch agent has no documented developer-API equivalent beyond the `web_search` tool; "DeeperSearch" has no Tier 1 documentation and is omitted.
- **Authentication and rate-limit specifics** per provider are out of scope here; consult each provider's API reference.
