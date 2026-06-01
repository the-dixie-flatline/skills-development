---
name: prompt-engineering-skill
description: Family-specific prompt-engineering references for current-generation LLMs — Claude, GPT-5.x / o-series, Gemini, Gemma, Llama, Qwen, Grok, Mistral, DeepSeek. Trigger when the user writes or debugs prompts for a specific model family, configures sampling or thinking/reasoning budgets, picks a model variant, works with a family's chat template or tool-use protocol, or migrates a prompt between versions. Skip for general prompt-engineering methodology (use prompt-engineering-architect) and for questions about how this current Claude session should respond.
version: 0.1.0
contract-version: 2026-04-18
---

# Prompt Engineering Skill

## Purpose

Route to the right family-specific reference when the user is prompting, configuring, or debugging against a specific LLM family. Each family has up to two reference files in `resources/`:

- `{family}-prompt.md` — Portable prompt-layer guidance (web UI, third-party wrappers, direct API).
- `{family}-prompt-api.md` — API-layer guidance (chat templates, sampling, reasoning toggles, tools, structured output, deployment).

The split exists so someone writing a webui prompt loads only the smaller prompt-layer file; someone building against the API loads both.

A second class of reference is **cross-family surface files** (`resources/*-surface.md`, `resources/deep-research-agents.md`, `resources/agent-orchestration-surfaces.md`). These tabulate behavior that is inherently cross-family — hosted deep-research agents, hosted multi-agent orchestration, OpenAI-compatibility divergences, and consumer/web-UI degradation. They are loaded *in addition to* a family file when the task targets one of those surfaces (see the surface axis in Routing).

## When to Use

Invoke this skill when the task centers on a specific LLM family and the question is family-specific:

- "Write a system prompt for Qwen3.6-35B-A3B that enforces a reasoning budget of..."
- "How do I toggle thinking mode on Gemini 3?"
- "What's the chat template for Llama 4 Maverick in a vLLM deployment?"
- "Why is my GPT-5.1 structured-output response returning extra fields?"
- "Help me migrate this system prompt from Claude 4.5 to Claude 4.7."

## When NOT to Use

- **General prompt-engineering methodology.** CoT theory, few-shot selection strategy, structured-output design in the abstract — that is the scope of the `prompt-engineering-architect` skill.
- **Questions about how this current Claude assistant should behave.** That is a normal Claude task, not a reference lookup.
- **Model rankings or "which is best."** No benchmarks, no buying guides, no leaderboard commentary.
- **Questions the user's own code or the file they opened already answers.** Read the file; do not load a reference.

## Routing Procedure

1. **Identify the target family** from the user's prompt, imported SDK (`anthropic`, `openai`, `google-genai`, transformers model ID, etc.), file naming, or an explicit question.
   - **Empty-arg case.** When the skill is invoked with no family argument, resolve the family from the triggering turn (the user's prose, the open file, the imported SDK). If the family is not recoverable that way, ask before loading anything. Never default to a family.
   - **Generated-prompt case.** When the deliverable is a prompt *for another model*, two families are in play: the **subject family** (whose behavior the prompt-author is reasoning about) and the **consumer family** (the model that will actually run the prompt). Route on the consumer family — load its files, because its behavioral quirks and contract are what the generated prompt must satisfy. If the task also requires reasoning about the authoring model's own output, load that family too and say which is which.
2. **Decide scope.** Scope is additive, not either/or.
   - Prompt-layer only: load `resources/{family}-prompt.md`.
   - API-layer relevant: also load `resources/{family}-prompt-api.md`. Triggers include the user mentioning sampling parameters, tools, structured outputs, reasoning/thinking budgets, streaming, caching, chat template tokens, or specific SDK calls.
   - **Mixed-task rule.** If the task includes authoring or rewriting prompts for the family, load the prompt-layer file *too* — even when the task is primarily integration or migration. From-scratch prompt writing needs the behavioral quirks and anti-patterns that live only in `{family}-prompt.md`. API involvement does not displace the prompt-layer file.
   - **Surface axis (third routing axis).** Beyond the family and the prompt-vs-API split, ask which *surface* the task targets. Hosted agent/product surfaces have their own contract distinct from the standard chat API, and their references are cross-family:
     - Hosted deep-research agents (async submit/poll) or web-search research loops → `resources/deep-research-agents.md`.
     - Hosted multi-agent / agent-orchestration surfaces → `resources/agent-orchestration-surfaces.md`.
     - Calling a non-OpenAI family through an OpenAI-shaped endpoint → `resources/openai-compatibility-surface.md`.
     - Consumer / web-UI behavior (vs the API) → `resources/webui-surfaces-and-silent-degradation.md`.
     Load the cross-family surface file *in addition to* the family file, not instead of it.
3. **Read the loaded file's front matter first.** The `versions:` field and `retrieved:` date decide whether the content still applies. If `retrieved:` is older than 90 days, say so before relying on the content. **This freshness check applies to family content reused from earlier in the conversation, not only to files read fresh this turn.** If you are answering from family guidance already in context, re-confirm its `retrieved:` date before relying on it; do not let the check lapse just because the file was loaded earlier.
4. **Answer using loaded content plus the reading discipline below.** Do not fill gaps from parametric memory; surface them as gaps.

## Coverage Status

| Family  | Prompt-layer | API-layer | Notes              |
|---------|--------------|-----------|--------------------|
| Claude  | published    | published | Opus 4.8 flagship (GA 2026-05-28) + Opus 4.7 still Active + Sonnet 4.6 + Haiku 4.5, as of 2026-06-01. Adaptive-only thinking on 4.7/4.8; Opus 4 / Sonnet 4 retire 2026-06-15 |
| OpenAI  | published    | published | GPT-5.5 flagship + 5.5-pro + 5.4 (cheaper tier) /mini/nano + 5.3-codex, as of 2026-06-01. Responses API primary; Assistants removed 2026-08-26 (→ Responses + Conversations); o*-deep-research IDs shut down 2026-07-23 |
| Gemini  | published    | published | 3.5 Flash GA + 3.1 Flash-Lite GA + 3.1 Pro Preview, as of 2026-06-01. thinkingLevel replaces thinkingBudget on Gemini 3; 2.0 GA shut down 2026-06-01, 2.5 GA sunsets 2026-10-16 |
| Gemma   | published    | published | Open weights, Apache 2.0. Gemma 4 E2B / E4B / 26B-A4B / 31B, re-verified 2026-06-01. 128K/256K context split; per-size dedicated draft model |
| Llama   | published    | published | Open weights, Meta. Llama 4 Scout + Maverick, re-confirmed 2026-06-01. No Llama 5 released |
| Qwen    | published    | published | Qwen3.7-Max GA flagship + Qwen3.6-Plus (demoted GA) + open `qwen3.6-35b-a3b` MoE + dense `Qwen3.6-27B`, as of 2026-06-01 |
| Grok    | published    | published | OpenAI-compatible API. grok-4.3 flagship + grok-build-0.1, as of 2026-06-01. reasoning_effort {none,low,medium,high}; prior fast/4.x/3 slugs retired 2026-05-15 |
| Mistral | published    | published | Open weights + native API. Mistral Medium 3.5 frontier + Small 4 hybrid, as of 2026-06-01. Unified binary reasoning_effort (high|none); Magistral → Legacy |
| DeepSeek| published    | published | DeepSeek V4 (`deepseek-v4-flash` / `deepseek-v4-pro`), as of 2026-06-01. MIT, 1M ctx; legacy chat/reasoner retire 2026-07-24; V3.2-Speciale API expired 2025-12-15 |

### Cross-family surface resources

| Resource | Status | Covers |
|----------|--------|--------|
| `deep-research-agents.md`              | published | Hosted deep-research agents (Gemini, OpenAI, Perplexity) + web-search tool-use loops (Anthropic, Grok), as of 2026-06-01 |
| `openai-compatibility-surface.md`      | published | Per-provider divergences through OpenAI-shaped endpoints (Grok, DeepSeek, Gemini, Qwen, Mistral, vLLM, llama.cpp), as of 2026-06-01 |
| `agent-orchestration-surfaces.md`      | published | Anthropic Managed Agents, OpenAI hosted/SDK split, the 3 Gemini agent products, as of 2026-06-01 |
| `webui-surfaces-and-silent-degradation.md` | published | Consumer/web-UI surfaces, tier-labeled; leads with documented gaps, as of 2026-06-01 |

An uncovered family or surface is an honest gap, not a placeholder. When the user asks about something not covered, tell them it is not yet covered rather than improvising a reference-shaped answer.

Update these tables whenever a reference lands or is removed.

## Reading Discipline

Applies to every loaded reference.

- **Trust tiers.** Tier 1 (provider docs) claims are usable flat. Tier 2 (community) claims must be reported as community-observed. Tier 3 should never appear in reference files; if you see it, flag it as a bug in the file.
- **Respect inline markers.**
  - `[testable: id=...]` — a claim the test harness will assert against live models. Quote it accurately; do not weaken it.
  - `[applies-to: <version-id>, ...]` — version-scoped claim. Ignore when the user's target is outside scope.
  - `[unverified]` — plausibly true, not documented by the provider. Relay with the same marker; do not promote it to fact.
  - `[disputed: <summary>]` — primary sources conflict. Present both positions.
- **Honor "Gaps" sections — this is the skill's strongest feature.** When the user asks about something a reference explicitly lists as unknown, say "I do not know" plus the file's framing. Do not paper over the gap. A declared gap is load-bearing: when a family file states that some surface is not exposed (an agent or trace shape a vendor does not document, a field whose behavior is unverified), that declared gap lets you reject a wrong premise instead of inventing an API to satisfy it. A declared gap beats a silent one precisely here. Never convert a declared gap into a confident answer.
- **Do not export cross-family guidance.** A Qwen-specific chat-template quirk is not generalizable to Llama just because both use ChatML-derived formats. Stay inside the family the user is targeting. (The cross-family `*-surface.md` resources are the deliberate exception: they exist precisely to tabulate cross-family divergence and are safe to read across families.)
- **Weight measured behavior over benchmark priors.** When a reference's documented behavior conflicts with the user's own operator-measured observation, weight the measurement above benchmark or marketing priors and above this file's general claims. Note the conflict and recommend updating the file rather than overriding the user's evidence.
- **Test a newer variant on your own axis before switching.** A newer model ID is not automatically better on a specific task. When advising a version migration, recommend testing the newer variant on the user's actual evaluation axis before adopting it; version-delta improvements are workload-dependent.

## Cross-Family Portability

When porting a prompt or integration between families, these are the points where assumptions break. This list is the meta-map; each family file has the specifics. Consult the target family file before reusing a pattern.

- **Reasoning / thinking controls are not portable.** Parameter names, accepted values, and semantics all differ: `reasoning.effort` {none,low,medium,high,xhigh} (OpenAI); `thinking.type: "adaptive"` + `output_config.effort` (Claude — adaptive-only on Opus 4.7/4.8, manual budgets rejected); `thinkingLevel` {minimal,low,medium,high} on Gemini 3 or `thinkingBudget` on Gemini 2.5; `enable_thinking` + `preserve_thinking` (Qwen); `<|think|>` token (Gemma); tri-state `thinking: {type:"enabled"|"disabled"}` or `reasoning_effort: "high" | "max"` (DeepSeek V4); binary `reasoning_effort: "high" | "none"` (Mistral, chat endpoint); `reasoning_effort` {none,low,medium,high}, default `low` (Grok grok-4.3); nothing at all (Llama). Treat `reasoning_effort` as a family-scoped name, not a universal knob — even where the name is shared, the accepted value set differs.

- **Reasoning-artifact multi-turn handling actively contradicts between families.**
  - Claude: `thinking` blocks **must** be preserved unchanged in tool-use multi-turn.
  - OpenAI: reasoning items **must** be preserved (via `previous_response_id` or explicit inclusion).
  - Gemini: `thoughtSignature` must be passed back.
  - DeepSeek (V4): **conditional** — on non-tool-call turns `reasoning_content` is ignored if sent back; on tool-call turns it **must** be passed back or the API returns HTTP 400. (The legacy `deepseek-reasoner` rule was: any `reasoning_content` on input returns 400.)
  - Gemma: strip thoughts from prior-turn history.
  - Qwen: opt-in via `preserve_thinking: True` (default `False`).

  There is no portable rule. Treat every family as bespoke.

- **Role names differ.**
  - `user` / `assistant` / `system` / `tool`: Claude, OpenAI, Qwen, Mistral, Grok, DeepSeek.
  - `user` / `model` (no `assistant`): Gemini, Gemma.
  - `ipython` role for tool output (not `tool`): Llama.
  - `developer` role with elevated precedence: OpenAI (general); DeepSeek (scoped to search-agent scenarios only).

  Prompts copied across families without role remapping will misparse.

- **"OpenAI-compatible" at the wire level is not "OpenAI-equivalent" semantically.** Several non-OpenAI families (Grok, DeepSeek, Mistral, Qwen, Gemini via shim) accept OpenAI-shaped requests but diverge on behavior: the reasoning field name, the multi-turn round-trip rule, rejected/no-op params, and cache-token field paths all differ per provider (e.g. Grok errors on `presence_penalty`/`frequency_penalty`/`stop` for reasoning models; DeepSeek's reasoning round-trip is conditional on tool calls; Mistral's `reasoning_effort` is binary and uses `max_tokens` not `max_completion_tokens`). Compatible wire format is a migration convenience, not a behavior guarantee. The full per-provider matrix is in `resources/openai-compatibility-surface.md`.

- **Open-weights chat templates need the canonical encoder.** Hand-assembling template strings is a landmine across open-weights families — Gemma's asymmetric `<|turn>` / `<turn|>`, DeepSeek's full-width `｜` and `▁` characters, Llama's token migration from 3.x, Mistral's version-dependent whitespace rules, Qwen's ChatML-derived structure. Use the provider's canonical encoder (`mistral-common`, DeepSeek's `encode_messages`, HuggingFace `apply_chat_template` where a Jinja template ships) rather than constructing the token stream manually.

- **Pin dated model IDs for production.** Closed-API families and most open-weights variants publish dated snapshots alongside rolling aliases. Aliases rotate without notice (Mistral's YYMM suffix is the clearest example; Google shut down the original `gemini-3-pro-preview` on 2026-03-09; OpenAI adds dated snapshots per release). Pin dated IDs in production; use aliases only for experimentation.

## Escalation

- **Target family not covered.** Say so plainly. Offer to help at a general prompt-engineering level (via the `prompt-engineering-architect` skill) or to draft a new reference using `SCHEMA.md`.
- **Reference is stale.** Flag to the user and offer to re-verify against primary sources before relying on the content for production work.
- **User's current observation contradicts the reference.** Do not silently concede. Note the conflict, ask for the user's source if they have one, and recommend updating the file.

## Adding a New Family

Follow `SCHEMA.md`. Order of operations:

1. Web-verify current generation, flagship IDs, and licensing against the provider's own documentation.
2. Draft `resources/{family}-prompt.md` (prompt-layer only).
3. Draft `resources/{family}-prompt-api.md` (API-layer only).
4. Self-audit: every claim has provenance; no Tier 3 leaks; no slop language; gaps declared.
5. Update this file's coverage table.

Do not commit. Human review is required before a reference lands on a public branch.
