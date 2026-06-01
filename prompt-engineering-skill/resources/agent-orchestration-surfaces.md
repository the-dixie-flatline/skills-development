---
family: cross-family
scope: agent-orchestration
families: [anthropic, openai, gemini]
retrieved: 2026-06-01
primary_sources:
  - https://platform.claude.com/docs/en/managed-agents/overview
  - https://platform.claude.com/docs/en/api/beta/agents/create
  - https://developers.openai.com/api/docs/guides/agents/orchestration
  - https://developers.openai.com/api/docs/models/gpt-5.5
  - https://developers.openai.com/api/docs/guides/deep-research
  - https://ai.google.dev/gemini-api/docs/antigravity-agent
  - https://ai.google.dev/gemini-api/docs/models/gemini-2.5-computer-use-preview-10-2025
maturity_note: |
  The one true hosted multi-agent surface documented here is Anthropic Managed
  Agents: a coordinator spawns 1-20 named agents as session threads. OpenAI's
  hosted surface is stateful single-agent (Responses + Conversations + Background
  mode); its multi-agent handoffs are Agents-SDK-only with no hosted endpoint.
  Gemini ships three distinct agent products that prior research conflated —
  Antigravity Agent (managed sandbox), Deep Research, and Computer Use — and only
  Antigravity is a general-purpose managed agent. All model IDs, beta/revision
  headers, and limits below are volatile and date-stamped; re-verify before use.
---

# Agent Orchestration Surfaces — Cross-Family Reference

Hosted agent and multi-agent orchestration surfaces across Anthropic, OpenAI, and
Gemini. Scope is the vendor's own server-side surface: what runs on the provider's
infrastructure versus what is SDK-only client orchestration. For the web-search
tool-use research loop and for Gemini Deep Research, see `resources/deep-research-agents.md`.

This file does not duplicate single-family API detail. For Claude Messages-API
sampling, thinking, and tool-use mechanics, see `resources/claude-prompt-api.md`.

## 1. Anthropic Managed Agents (hosted multi-agent)

The most fully documented hosted multi-agent surface. A coordinator agent spawns
referenced agents as session threads; the platform manages session and environment
state server-side.
[source: platform.claude.com/docs/en/managed-agents/overview, retrieved 2026-06-01]

### Beta gating

The beta header `managed-agents-2026-04-01` is **required** on all Managed Agents
endpoints. Requests without it do not reach the surface.
[source: platform.claude.com/docs/en/managed-agents/overview, retrieved 2026-06-01]
[applies-to: managed-agents-2026-04-01]

### Multi-agent topology

- **Depth limit is 1.** A coordinator may reference other agents, but those
  referenced agents must exist, must not be archived, and must not themselves
  have `multiagent` set. There is no documented recursion past one level.
- **Coordinator roster: 1-20 entries.** The roster names the agents the
  coordinator may spawn as session threads.

[source: platform.claude.com/docs/en/api/beta/agents/create, retrieved 2026-06-01]

### Agent definition limits

| Field              | Limit                                  |
|--------------------|----------------------------------------|
| System prompt      | up to 100,000 characters               |
| Agent name         | 1-256 characters                       |
| Agent description  | up to 2,048 characters                 |
| Tools (total)      | maximum 128 across all toolsets        |
| Tool name          | 1-128 characters                       |
| Tool description   | 1-1,024 characters                     |

[source: platform.claude.com/docs/en/api/beta/agents/create, retrieved 2026-06-01]

### Built-in tools

Bash, file operations, web search and fetch, and MCP servers are available as
built-in tooling.
[source: platform.claude.com/docs/en/managed-agents/overview, retrieved 2026-06-01]

### Rate limits (per organization)

- **Create endpoints** (agents, sessions, environments, and similar):
  300 requests/min.
- **Read endpoints** (retrieve, list, stream, and similar): 600 requests/min.

[source: platform.claude.com/docs/en/managed-agents/overview, retrieved 2026-06-01]

### Compliance posture

Managed Agents is **not** eligible for Zero Data Retention (ZDR) or for HIPAA
Business Associate Agreement (BAA) coverage. The surface is stateful by design,
which is the stated reason it sits outside those programs. Route regulated
workloads accordingly.
[source: platform.claude.com/docs/en/managed-agents/overview, retrieved 2026-06-01]

## 2. OpenAI (hosted single-agent; multi-agent is SDK-only)

OpenAI's hosted surface is a **stateful single-agent** stack. Multi-agent
orchestration exists, but only in the Agents SDK on the client — there is no
hosted multi-agent endpoint.

### Hosted / server-side surface

The server-side surface is the Responses API plus the Conversations API for state
management, plus Background mode, plus hosted tools running on `gpt-5.5`. Agent
Builder / ChatKit is a hosted build surface layered on top.
[source: developers.openai.com/api/docs/guides/agents/orchestration, retrieved 2026-06-01]
[applies-to: gpt-5.5]

Hosted tools on `gpt-5.5`: web search, file search, code interpreter, hosted shell,
computer use, MCP, and tool search.
[source: developers.openai.com/api/docs/models/gpt-5.5, retrieved 2026-06-01]
[applies-to: gpt-5.5]

### Multi-agent orchestration is SDK-only

Multi-agent handoffs and orchestration live exclusively in the Agents SDK —
Python `agents` or JS `@openai/agents`. Two patterns:

- **Handoffs** — transfer control to a specialist agent.
- **Agents as tools** — keep the manager agent in control while calling another
  agent as a tool.

There is **no hosted multi-agent handoff API endpoint**. The orchestration runs in
the caller's process.
[source: developers.openai.com/api/docs/guides/agents/orchestration, retrieved 2026-06-01]

### MCP approval default

MCP tool calls default to requiring **per-call approval**. This is configurable via
`require_approval`.
[source: developers.openai.com/api/docs/guides/agents/orchestration, retrieved 2026-06-01]

### Deep Research

OpenAI's Deep Research surface is out of scope here; see
`resources/deep-research-agents.md`.
[source: developers.openai.com/api/docs/guides/deep-research, retrieved 2026-06-01]

## 3. Gemini — three distinct agent products

Gemini does not ship one agent product. It ships three, all on the Interactions
API, and they are not interchangeable. Prior research conflated them; treat each
separately.

### 3a. Antigravity Agent (general-purpose managed sandbox)

A general-purpose managed agent: it "reasons, executes code, manages files, and
browses the web inside your own secure Linux sandbox, hosted by Google." It is
powered by Gemini 3.5 Flash and uses the same harness as the Antigravity IDE.
[source: ai.google.dev/gemini-api/docs/antigravity-agent, retrieved 2026-06-01]
[applies-to: antigravity-preview-05-2026]

**Surface and headers.** Invoked through the Interactions API
(`client.interactions.create`). Setting `environment="remote"` provisions the
sandbox. Api-Revision header `2026-05-20`.
[source: ai.google.dev/gemini-api/docs/antigravity-agent, retrieved 2026-06-01]
[applies-to: antigravity-preview-05-2026]

**Default tools.** `code_execution` (bash / Python / Node), `google_search`, and
`url_context`. The filesystem tool is auto-enabled when `environment` is set.
[source: ai.google.dev/gemini-api/docs/antigravity-agent, retrieved 2026-06-01]

**Token limits.** Input context 1,048,576 tokens (compacted at roughly 135k);
output 65,536 tokens. The 65,536 output cap is the **Antigravity Agent's**, not
Gemini Deep Research's — do not transfer it across products.
[source: ai.google.dev/gemini-api/docs/antigravity-agent, retrieved 2026-06-01]

**Environments.** Three forms for the `environment` argument:

- `"remote"` — fresh sandbox.
- a reuse handle such as `"env_abc123"` — state persists across calls.
- a full `EnvironmentConfig` — Git / GCS / inline sources plus network rules.

[source: ai.google.dev/gemini-api/docs/antigravity-agent, retrieved 2026-06-01]

**Pricing.** Pay-as-you-go on the underlying Gemini tokens plus tools; environment
compute is **not billed during preview**. Typical task $0.25-3.25; complex tasks up
to roughly $5. Around 50-70% of input tokens are typically cached.
[source: ai.google.dev/gemini-api/docs/antigravity-agent, retrieved 2026-06-01]
[applies-to: antigravity-preview-05-2026]

**Documented limitations.**

- `temperature`, `top_p`, `top_k`, `stop_sequences`, and `max_output_tokens`
  return HTTP 400.
- No structured outputs.
- `file_search`, `computer_use`, `google_maps`, `function_calling`, and `mcp` are
  not yet supported.
- No `background=True`.
- Requires `store=True`.
- Multimodal input is limited to text and image.

[source: ai.google.dev/gemini-api/docs/antigravity-agent, retrieved 2026-06-01]
[applies-to: antigravity-preview-05-2026]

### 3b. Gemini Deep Research

A separate product on the Interactions API
(`deep-research-preview-04-2026` / `deep-research-max-preview-04-2026`). Detail,
including its own token and output behavior, lives in
`resources/deep-research-agents.md`; do not assume Antigravity's limits apply to it.
[source: ai.google.dev/gemini-api/docs/antigravity-agent, retrieved 2026-06-01]

### 3c. Computer Use

`gemini-2.5-computer-use-preview-10-2025` perceives a screen and performs UI actions
(click, type, navigate). It is **not** a hosted sandbox — the caller supplies the
environment. The Gemini 3.x text models do not support computer use.
[source: ai.google.dev/gemini-api/docs/models/gemini-2.5-computer-use-preview-10-2025, retrieved 2026-06-01]
[applies-to: gemini-2.5-computer-use-preview-10-2025]

## 4. Gaps

- **xAI / Grok.** No documented vendor multi-agent orchestration surface. The only
  agentic developer surface is the web-search tool-use loop; see
  `resources/deep-research-agents.md`.
- **DeepSeek, Mistral, Qwen, Llama, Gemma.** No first-party hosted managed
  multi-agent orchestration surface is documented. Orchestration over these models
  is left to third-party frameworks running in the caller's infrastructure.
- **Anthropic Managed Agents concurrent-threads cap.** A separate numeric cap on
  concurrent session threads, beyond the 1-20 coordinator roster, is not documented
  in the retrieved sources.
- **Gemini agent parity on Vertex AI.** Only the ai.google.dev Gemini Developer API
  surface was confirmed. Vertex AI parity and wording for the three Gemini agent
  products were not verified.
- **OpenAI hosted multi-agent.** No hosted multi-agent handoff endpoint exists as of
  retrieval; if one ships later, it supersedes the SDK-only statement in §2.
