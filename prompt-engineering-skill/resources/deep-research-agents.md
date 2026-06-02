---
family: cross-family
scope: deep-research-agents
families: [gemini, openai, perplexity, anthropic, grok]
retrieved: 2026-06-01
primary_sources:
  - https://ai.google.dev/gemini-api/docs/interactions/deep-research
  - https://ai.google.dev/gemini-api/docs/interactions
  - https://ai.google.dev/api/interactions-api
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
  before pinning.
---

# Deep Research Agents — Cross-Family Reference

Hosted research agents do not share one shape. Two are documented here, and they are not interchangeable.

- **Shape A — async hosted agents.** Submit a request, get an identifier, poll for a terminal state, read the result. The provider runs a multi-step research loop server-side, sometimes for tens of minutes. Gemini Deep Research, OpenAI Deep Research, and Perplexity Sonar Deep Research are in this shape.
- **Shape B — web-search tool-use loops.** A single synchronous model request in which the model decides to search, the API executes searches inline, and the loop iterates within that one request. There is no separate agent resource to submit to or poll. Anthropic and xAI Grok deliver agentic web research this way through server-side tools.

Pick the shape first. The integration code, latency profile, and citation surface differ between them. Do not template a Shape B tool call as if it were a Shape A submit/poll job.

## Shape A: Async Hosted Agents (submit -> poll)

### Gemini Deep Research

**Agent IDs.** `deep-research-preview-04-2026` (fast tier) and `deep-research-max-preview-04-2026` (max tier), both Public Preview. The overview still lists `deep-research-pro-preview-12-2025`. All three are preview IDs and rotate.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]
[source: https://ai.google.dev/gemini-api/docs/interactions, retrieved 2026-06-01]

**Submit.** `POST /v1beta/interactions` (SDK `client.interactions.create`). Required body fields: `input`, `agent`. `background: true` is mandatory — the docs state "Agents are required to use background=True". `store: true` is required; `store=false` is incompatible with `background=true`.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]
[source: https://ai.google.dev/api/interactions-api, retrieved 2026-06-01]

**Poll.** `GET /v1beta/interactions/{id}` (SDK `interactions.get(id)`). Loop on `.status`.
[source: https://ai.google.dev/api/interactions-api, retrieved 2026-06-01]

**Status enum.** Full set: `in_progress`, `requires_action`, `completed`, `failed`, `cancelled`, `incomplete`, `budget_exceeded`. Terminal success/failure values are lowercase `completed` / `failed` — not `SUCCEEDED`. Do not match on uppercase or on Perplexity-style status strings.
[source: https://ai.google.dev/api/interactions-api, retrieved 2026-06-01]
[testable: id=gemini.deep-research-status-lowercase.v1, expected=a completed interaction returns status string "completed", not "SUCCEEDED"]

**Time budget.** Maximum research time is 60 minutes; most tasks finish within roughly 20 minutes.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

**Structured outputs.** Not supported — "The Deep Research Agent currently doesn't support structured outputs." Steer output shape with formatting instructions in the prompt, not a schema.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

**Usage and cost.** The response carries a `usage` object with token statistics: `total_tokens`, `total_input_tokens`, `total_output_tokens`, `total_thought_tokens`, `total_tool_use_tokens`, `cached_tokens_by_modality`, and `grounding_tool_count`. There is no monetary cost field in the response; cost is a narrative estimate only — roughly $1–3 per task for Deep Research and roughly $3–7 per task for Deep Research Max.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

**Tools.** When `tools` is omitted the defaults are `google_search`, `url_context`, and `code_execution`. Optional additions: `mcp_server` and `file_search`. Custom function-calling tools are not supported — remote MCP only.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

**Agent config.** `agent_config` fields: `thinking_summaries` (`none` / `auto`), `visualization` (`auto` / `off`), `collaborative_planning` (boolean). Use `previous_interaction_id` to refine or follow up on a prior interaction.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]

**Streaming.** Streaming requires `stream=True` and `background=True` together. Set `thinking_summaries: "auto"` to receive thoughts. `step.delta` deltas carry `thought`, `text`, or `image` content. Reconnect a dropped stream with `?stream=true&last_event_id=`. The examples send an `Api-Revision` header of `2026-05-20`. SDK floor versions: `google-genai >= 1.55.0`, `@google/genai >= 1.33.0`. Storage retention is 55 days on paid tiers and 1 day on the free tier.
[source: https://ai.google.dev/gemini-api/docs/interactions/deep-research, retrieved 2026-06-01]
[source: https://ai.google.dev/gemini-api/docs/interactions, retrieved 2026-06-01]

Do not assert an output-token cap for Gemini Deep Research. The 65,536-token cap that circulates for Gemini agents belongs to the separate Antigravity Agent, documented in `resources/agent-orchestration-surfaces.md`, not to Deep Research. See Gaps.

### OpenAI Deep Research

**Models and invocation.** `o3-deep-research` and `o4-mini-deep-research`, called through the Responses API in background mode. Configure a webhook to be notified on completion rather than polling indefinitely.
[source: https://developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]

**Near-term ID mismatch — flag this.** The `o3-deep-research` / `o4-mini-deep-research` IDs are scheduled to shut down 2026-07-23, with `gpt-5.5-pro` named as the replacement. As of 2026-06-01 the deep-research guide still documents the `o*-deep-research` IDs as the invocation. Code written today against the documented IDs will need migration before the shutdown; verify the current guide before building.
[source: https://developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]
[source: https://developers.openai.com/api/docs/deprecations, retrieved 2026-06-01]

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

**Tool types.** `web_search_20260209` (supports dynamic filtering; requires the code-execution tool) and `web_search_20250305`. Citations are always on, returned as `web_search_result_location` blocks with `url`, `title`, and `cited_text` up to 150 characters. Pricing is $10 per 1,000 searches. An org admin must enable the tool in the Console before use.
[source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool, retrieved 2026-06-01]

### xAI Grok

"DeepSearch" is a consumer-app agent on grok.com and X, documented only in Grok-3 launch product copy. It is not a developer-API async agent.
[source: https://x.ai/news/grok-3, retrieved 2026-06-01]

The developer-API analog is the server-side Web Search / X Search tools (`web_search`), a synchronous tool-use loop that returns `response.citations`.
[source: https://docs.x.ai/developers/tools/web-search, retrieved 2026-06-01]

Do not assert "DeeperSearch" as a distinct named developer feature. It is not documented in any Tier 1 source and is omitted here.

## Comparison Framing

| Provider   | Shape | Entry point                                    | Completion model        | Citation surface                  |
|------------|-------|------------------------------------------------|-------------------------|-----------------------------------|
| Gemini     | A     | `POST /v1beta/interactions` (`background=true`)| Poll `.status`          | grounding via default tools       |
| OpenAI     | A     | Responses API, background mode                 | Webhook on completion   | `annotations` on `output_text`    |
| Perplexity | A     | `/v1/async/sonar` (or sync `/v1/sonar`)        | Poll `{request_id}`     | `citations` + `search_results`    |
| Anthropic  | B     | `web_search` server tool in Messages request   | Synchronous, in-request | `web_search_result_location`      |
| xAI Grok   | B     | `web_search` server tool                       | Synchronous, in-request | `response.citations`              |

Shape A decouples submission from result: budget for minutes of latency, persist the identifier, and reconnect or webhook to the result. Shape B returns within one request: no identifier to persist, no terminal-state enum, latency bounded by the model's own search loop. Status-string conventions differ even within Shape A — Gemini uses lowercase (`completed`), Perplexity uses uppercase (`CREATED`) — so status-handling code does not port across providers.

## Gaps

- **Gemini Deep Research output-token cap** — not documented for this agent. The 65,536-token figure belongs to the separate Antigravity Agent (see `resources/agent-orchestration-surfaces.md`), not Deep Research.
- **Gemini `agent_config` semantics** — the runtime effect of `visualization`, `collaborative_planning`, and the interaction between `thinking_summaries` and streaming are not detailed beyond the field names.
- **OpenAI deep-research post-migration shape** — how the replacement `gpt-5.5-pro` is invoked for deep research after the 2026-07-23 shutdown of the `o*-deep-research` IDs is not covered by the current guide.
- **Perplexity lifecycle label** — no GA / beta / preview status is stated on the `sonar-deep-research` model page.
- **Perplexity async polling cadence and timeout** — recommended poll interval and maximum job lifetime are not specified beyond the 3–5 concurrent-request guidance.
- **Anthropic / xAI managed async research** — neither provider exposes a documented async "Research" agent resource in the sources retrieved; this file covers only their synchronous web-search tools and (for Anthropic) cross-references Managed Agents.
- **xAI DeepSearch developer API** — the consumer-app DeepSearch agent has no documented developer-API equivalent beyond the `web_search` tool; "DeeperSearch" has no Tier 1 documentation and is omitted.
- **Authentication and rate-limit specifics** per provider are out of scope here; consult each provider's API reference.
