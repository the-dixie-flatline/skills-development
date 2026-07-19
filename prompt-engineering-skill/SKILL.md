---
name: prompt-engineering-skill
description: Family-specific prompt-engineering references for current-generation LLMs — Claude, GPT-5.x / o-series, gpt-oss (open weights), Gemini, Gemma, Llama, Qwen, Grok, Mistral, DeepSeek, GLM, MiniMax, Kimi. Trigger when the user writes or debugs prompts for a specific model family, configures sampling or thinking/reasoning budgets, picks a model variant, works with a family's chat template or tool-use protocol, migrates a prompt between versions, authors one prompt that must run across multiple families or targets a family not covered here (portable-baseline lane), configures a hosted deep-research or agent-orchestration surface, prompts a source-grounded vendor product such as Gemini Notebook (formerly NotebookLM), calls a non-OpenAI family through an OpenAI-compatible endpoint, or reasons about consumer/web-UI vs API behavior. Skip for general prompt-engineering methodology beyond structural baseline (the planned prompt-engineering-architect skill, not yet published) and for questions about how this current Claude session should respond.
version: 0.8.1
contract-version: 2026-04-18
---

# Prompt Engineering Skill

## Purpose

Route to the right family-specific reference when the user is prompting, configuring, or debugging against a specific LLM family. Each family has up to two reference files in `resources/`:

- `{family}-prompt.md` — Portable prompt-layer guidance (web UI, third-party wrappers, direct API).
- `{family}-prompt-api.md` — API-layer guidance (chat templates, sampling, reasoning toggles, tools, structured output, deployment).

The split exists so someone writing a webui prompt loads only the smaller prompt-layer file; someone building against the API loads both.

A second class of reference is **surface files** (`resources/*-surface.md`, `resources/deep-research-agents.md`, `resources/agent-orchestration-surfaces.md`). These cover behavior that belongs to a *product surface* rather than to a model's prompt or API layer. Two sub-kinds exist:

- **Cross-family surfaces** tabulate divergence across vendors — hosted deep-research agents, hosted multi-agent orchestration, OpenAI-compatibility, consumer/web-UI degradation. Loaded *in addition to* a family file.
- **Family-scoped product surfaces** cover one vendor product whose contract diverges sharply from that vendor's own API (currently `gemini-notebook-surface.md`). These may be loaded *instead of* the family file when the product exposes none of the family's inference controls; the file says which.

## When to Use

Invoke this skill when the task centers on a specific LLM family and the question is family-specific:

- "Write a system prompt for Qwen3.6-35B-A3B that enforces a reasoning budget of..."
- "How do I toggle thinking mode on Gemini 3?"
- "What's the chat template for Llama 4 Maverick in a vLLM deployment?"
- "Why is my GPT-5.1 structured-output response returning extra fields?"
- "Help me migrate this system prompt from Claude 4.5 to Claude 4.7."

## When NOT to Use

- **General prompt-engineering methodology.** CoT theory, few-shot selection strategy, structured-output design in the abstract — that is the scope of the `prompt-engineering-architect` skill (planned, not yet published; say so rather than routing to it as if it existed). The exception now in scope here: *structural* baseline for multi-family and uncovered-family prompts lives in `resources/portable-baseline.md`.
- **Questions about how this current Claude assistant should behave.** That is a normal Claude task, not a reference lookup.
- **Model rankings or "which is best."** No benchmarks, no buying guides, no leaderboard commentary.
- **Questions the user's own code or the file they opened already answers.** Read the file; do not load a reference.

## Routing Procedure

1. **Identify the target family** from the user's prompt, imported SDK (`anthropic`, `openai`, `google-genai`, transformers model ID, etc.), file naming, or an explicit question.
   - **Empty-arg case.** When the skill is invoked with no family argument, resolve the family from the triggering turn (the user's prose, the open file, the imported SDK). If the family is not recoverable that way, ask before loading anything. Never default to a family.
   - **Generated-prompt case.** When the deliverable is a prompt *for another model*, two families are in play: the **subject family** (whose behavior the prompt-author is reasoning about) and the **consumer family** (the model that will actually run the prompt). Route on the consumer family — load its files, because its behavioral quirks and contract are what the generated prompt must satisfy. If the task also requires reasoning about the authoring model's own output, load that family too and say which is which.
   - **Same-family-consumer tie-breaker.** When the generated prompt targets *another session of the same family* (e.g. a Claude assistant authoring an operational kickoff/handoff prompt for another Claude agent), the "Generated-prompt case" above and the "how this current Claude assistant should behave" exclusion (load nothing) can both appear to apply. Break the tie on prompt content: if the generated prompt is pure operational orchestration or role-framing — no model choice, sampling, thinking budget, template token, or family behavioral quirk in play — the family file adds nothing; skip it. If the generated prompt *pins a variant or configures family-specific behavior* (a specific model ID, an effort/thinking setting, a refusal-surface or template concern), load the consumer family file **even when that family is Claude**. The "current Claude assistant" exclusion covers only the running session, not prompts authored for other or future same-family sessions.
2. **Decide scope.** Scope is additive, not either/or.
   - Prompt-layer only: load `resources/{family}-prompt.md`.
   - API-layer relevant: also load `resources/{family}-prompt-api.md`. Triggers include the user mentioning sampling parameters, tools, structured outputs, reasoning/thinking budgets, streaming, caching, chat template tokens, or specific SDK calls.
   - **Mixed-task rule.** If the task includes authoring or rewriting prompts for the family, load the prompt-layer file *too* — even when the task is primarily integration or migration. From-scratch prompt writing needs the behavioral quirks and anti-patterns that live only in `{family}-prompt.md`. API involvement does not displace the prompt-layer file.
   - **Surface axis (third routing axis).** Beyond the family and the prompt-vs-API split, ask which *surface* the task targets. Hosted agent/product surfaces have their own contract distinct from the standard chat API, and their references are cross-family:
     - Hosted deep-research agents (async submit/poll) or web-search research loops → `resources/deep-research-agents.md`.
     - **Hosted vendor** multi-agent / agent-orchestration endpoints (managed-agent APIs — Anthropic Managed Agents, OpenAI hosted agents, the Gemini agent products) → `resources/agent-orchestration-surfaces.md`. Scope this to vendor-hosted orchestration endpoints. **Exclusion:** authoring prompts or briefs for a local or CLI agent runner is prompt-craft, not a hosted-surface lookup — do not load this file for it; route to the consumer family file (and `prompt-engineering-architect` for orchestration methodology).
     - Calling a non-OpenAI family through an OpenAI-shaped endpoint → `resources/openai-compatibility-surface.md`.
     - Consumer / web-UI behavior (vs the API) → `resources/webui-surfaces-and-silent-degradation.md`.
     - One prompt that must run **unmodified across two or more families** (router deployment, fallback chain, cross-vendor A/B), or a prompt drafted **before the target model is chosen** → `resources/portable-baseline.md`, in addition to each covered target family's prompt-layer file. Distinguish from migration: porting a prompt *from one family to another* stays with the family files plus the Cross-Family Portability list below; the portable-baseline file is for one artifact serving many targets at once.
     - Target family **not covered** by this skill → `resources/portable-baseline.md` alone; see Escalation.
     - **Gemini Notebook** (formerly NotebookLM), including Gemini Notebook Enterprise → `resources/gemini-notebook-surface.md`. Trigger on the product name in either form, on notebooks-inside-the-Gemini-app, or on source-grounded/RAG questions scoped to that product. This one is **substitutive, not additive**: Notebook exposes no sampling, thinking level, system prompt, or chat template, so `gemini-prompt.md` and `gemini-prompt-api.md` add nothing unless the task *also* touches the Gemini API. Load the family files only in that mixed case.
     Load cross-family surface files *in addition to* the family file. The Gemini Notebook file is the documented exception (see the two sub-kinds above).

     **Notebook routing tie-breakers.** Overlapping triggers resolve as follows: Notebook's own Deep Research feature is a per-tier quota inside the product and routes here, **not** to `deep-research-agents.md`, which covers hosted submit/poll research APIs. Notebook is a consumer surface but routes here, **not** to `webui-surfaces-and-silent-degradation.md`, because Google documents its grounding and prompt-composition contract explicitly — the generic web-UI file's lead finding (undocumented defaults) does not apply. Load those two only if the task genuinely spans products.
3. **Read the loaded file's front matter first.** The `versions:` field and `retrieved:` date decide whether the content still applies. If `retrieved:` is older than 90 days, say so before relying on the content. **This freshness check applies to family content reused from earlier in the conversation, not only to files read fresh this turn.** If you are answering from family guidance already in context, re-confirm its `retrieved:` date before relying on it; do not let the check lapse just because the file was loaded earlier. **Re-check `[applies-to]` version-scope on reuse too, especially when the consumer model set changes mid-conversation.** A version being removed from the target roster invalidates that version's scoped claims exactly as a lapsed `retrieved:` date invalidates a file — a claim pinned `[applies-to: <flagship-id>]` is void or contradicted in a pipeline that excludes that flagship, even though it was valid when first loaded.
4. **Answer using loaded content plus the reading discipline below.** Do not fill gaps from parametric memory; surface them as gaps.

## Coverage Status

| Family  | Prompt-layer | API-layer | Notes              |
|---------|--------------|-----------|--------------------|
| Claude  | published    | published | Fable 5 flagship (`claude-fable-5`, GA 2026-06-09) + Opus 4.8/4.7 Active + Sonnet 5 (`claude-sonnet-5`, GA 2026-06-30, supersedes Sonnet 4.6) + Haiku 4.5, re-verified 2026-07-19 (whats-new-claude-4-7 page retired; citations re-anchored; Vertex structured outputs is Preview, not GA). Fable 5: always-on adaptive thinking, summarized-only thinking output, classifier refusals + fallback surface; adaptive-only thinking on 4.7/4.8; Opus 4 / Sonnet 4 retired 2026-06-15; Opus 4.1 deprecated (retires 2026-08-05); Mythos Preview retires 2026-07-21; frontier_llm now a visible refusal category |
| OpenAI  | published    | published | GPT-5.6 flagship: Sol (alias `gpt-5.6`) / Terra / Luna, 1.05M ctx, 128K out, cutoff Feb 16 2026, uniform across tiers; effort adds `max` (default `medium`); per-request `reasoning.mode` {standard,pro} billed at standard rates and ADDITIVE to (not replacing) the `-pro` IDs; `reasoning.context` {auto,current_turn,all_turns}; assistant-only `phase` {commentary,final_answer} (5.4/5.5 confirmed). 5.5/5.5-pro/5.4/mini/nano/5.3-codex prior-gen but Active; `GPT-5.4 Pro` registry-listed. Responses API primary + Multi-agent orchestration beta (`responses_multi_agent=v1`), Programmatic Tool Calling, explicit prompt-caching (`prompt_cache_key` required + 1.25x cache-write on 5.6+). Effort enum CONFLICTS across two vendor pages. Lifecycle: o*-deep-research/gpt-5.2-codex shut down 2026-07-23, snapshot wave (incl. o3-2025-04-16) 2026-12-11, dall-e/Realtime-Beta removed 2026-05-12, Assistants 2026-08-26, Agent Builder/Evals/reusable-prompts 2026-11-30. As of 2026-07-19. Open-weight line is the separate `gpt-oss` row below |
| gpt-oss | published    | published | OpenAI open-weight line (the OpenAI counterpart to the Gemini-vs-Gemma split); distinct surface from the GPT-5.x Responses API above. Apache 2.0, Harmony-format. Base gpt-oss (20b = 21B/3.6B active, 120b = 117B/5.1B active) + gpt-oss-safeguard (20b/120b) classifier fine-tunes, as of 2026-07-19. `reasoning_effort` {low,medium,high} default medium SET IN THE SYSTEM MESSAGE (not a top-level field); native MXFP4, 131K ctx via YaRN; T=1.0/top_p=1.0 base sampling. Safeguard = safety-classification only (policy in `system`, content in `user`, four-section policy, 400-600-tok heuristic); agentic/tool work routes to base gpt-oss. Groq serves safeguard-20b (Tier-2 host: 131K/65K, $0.075/$0.30 per 1M) but NOT safeguard-120b (general `gpt-oss-120b` IS served) |
| Gemini  | published    | published | 3.5 Flash GA + 3.1 Flash-Lite GA + 3.1 Pro Preview. Interactions API is the primary surface (GA 2026-06-22, SDK >= 2.3.0, snake_case; `tool_choice`/`thinking_level`/`response_format` under `generation_config`); `generateContent` legacy-but-supported (still receiving mainline models). thinkingLevel replaces thinkingBudget on Gemini 3; 2.0 GA shut down 2026-06-01, 2.5 GA sunsets 2026-10-16. As of 2026-07-19. **Gemini Notebook is a separate surface** — see `gemini-notebook-surface.md`; these two files do not apply to it |
| Gemma   | published    | published | Open weights, Apache 2.0. Gemma 4 E2B / E4B / 26B-A4B / 31B / 12B Unified (encoder-free multimodal, added 2026-06-03), re-verified 2026-07-19. 128K/256K context split; dedicated draft model on the four original sizes; audio on E-series + 12B Unified |
| Llama   | published    | published | Open weights, Meta. Llama 4 Scout + Maverick, re-confirmed 2026-07-19. No Llama 5 released. Muse Spark (closed-weight Meta Model API, public preview 2026-07-09, Meta Superintelligence Labs) is NOT Llama and is out of scope |
| Qwen    | published    | published | Qwen3.7-Max GA flagship + Qwen3.6-Plus (demoted GA) + open `qwen3.6-35b-a3b` MoE + dense `Qwen3.6-27B`; Model Studio per-tier pricing, cache rates, and deprecation ledger re-verified 2026-07-19 |
| Grok    | published    | published | OpenAI-compatible API. grok-4.5 flagship (500K context, reasoning_effort {low,medium,high} default high, `none` removed) supersedes grok-4.3 (1M context, {none,low,medium,high} default low, still live); grok-build-0.1 coding model. Flagship context REGRESSES vs prior gen (500K < 1M). Prior fast/4.x/3 slugs retired 2026-05-15, route to grok-4.3 (not 4.5), as of 2026-07-19 |
| Mistral | published    | published | Open weights + native API. Mistral Medium 3.5 frontier + Small 4 hybrid, as of 2026-06-01. Unified binary reasoning_effort (high|none); Magistral → Legacy |
| DeepSeek| published    | published | DeepSeek V4 (`deepseek-v4-flash` / `deepseek-v4-pro`), as of 2026-06-01. MIT, 1M ctx; legacy chat/reasoner retire 2026-07-24; V3.2-Speciale API expired 2025-12-15 |
| GLM     | published    | published | GLM-5.2 flagship (`glm-5.2`, 753B MoE, IndexShare sparse attention, 1M ctx, MIT open weights) + GLM-5.1 / GLM-5-Turbo / GLM-4.7, as of 2026-07-19. `reasoning_effort` 7-value ladder shims onto 2 thinking tiers (low/medium→high, xhigh→max, none/minimal→skip); Anthropic-compat endpoint `api.z.ai/api/anthropic` as a Claude Code drop-in; max-output disputed 128K (docs.z.ai) vs 163,840 (HF card) |
| Kimi    | published    | published | Kimi K3 (`kimi-k3`) flagship, 2.8T + KDA, 1M ctx, always-on reasoning (`reasoning_effort: max` only), as of 2026-07-19. OpenAI-compat (`api.moonshot.ai/v1`) + Anthropic-compat (`api.moonshot.ai/anthropic`, `kimi-k3[1m]`); Kimi Code (`api.kimi.com/coding`) is a separate product. Open weights announced for 2026-07-27, not yet released. K2.5/moonshot-v1 sunset Aug 31; k2 previews discontinued May 25. Active K2.x: k2.6 (256K), k2.7-code(-highspeed) |
| MiniMax | published    | published | M3 flagship (`MiniMax-M3`, 1M ctx, ~428B/~23B-active MoE, MSA) + M2.7/-highspeed Current + M2.5/2.1/2 Legacy + `M2-her` chat (64K), as of 2026-07-19. Open weights published (MiniMax Community License, non-commercial default). Anthropic-compat `/anthropic` recommended path (OpenAI-compat + legacy `chatcompletion_v2` also exist); thinking OFF-by-default on M3, un-disable-able on M2.x; passive-only caching on M3, explicit `cache_control` on M2.x; multimodal M3-only |

### Cross-family surface resources

| Resource | Status | Covers |
|----------|--------|--------|
| `deep-research-agents.md`              | published | Hosted deep-research agents (Gemini, OpenAI, Perplexity) + web-search tool-use loops (Anthropic, Grok), as of 2026-06-01 |
| `openai-compatibility-surface.md`      | published | Per-provider divergences through OpenAI-shaped endpoints (Grok, DeepSeek, Gemini, Qwen, Mistral, vLLM, llama.cpp), as of 2026-06-01 |
| `agent-orchestration-surfaces.md`      | published | Anthropic Managed Agents, OpenAI hosted/SDK split, the 3 Gemini agent products, as of 2026-06-01 |
| `webui-surfaces-and-silent-degradation.md` | published | Consumer/web-UI surfaces, tier-labeled; leads with documented gaps, as of 2026-06-01 |
| `portable-baseline.md`                 | published | Portable prompt skeleton (convergence synthesis over Anthropic/OpenAI/Google guides), reasoning-era corrections, router-portable exclusion rules, uncovered-family procedure, as of 2026-07-19. Tier 2 format-sensitivity evidence flagged; family files always outrank it for a covered family |
| `gemini-notebook-surface.md`           | published | **Family-scoped (Gemini), substitutive.** Gemini Notebook / NotebookLM + Notebook Enterprise, as of 2026-07-19. Renamed 2026-07-16. RAG not context (retrieval+ranking and query decomposition are vendor-documented); grounding differs between the standalone app and the Gemini app for the same notebook; no inference controls exposed; enterprise `v1alpha` REST API is CRUD + artifacts with **no chat/query endpoint**. Source ceiling 200MB consumer vs 500 MB enterprise. Chat-history retention `[disputed]` (stale marketing page) |

An uncovered family or surface is an honest gap, not a placeholder. When the user asks about something not covered, tell them it is not yet covered rather than improvising a reference-shaped answer.

Update these tables whenever a reference lands or is removed.

## Reading Discipline

Applies to every loaded reference.

- **Trust tiers.** Tier 1 (provider docs) claims are usable flat. Tier 2 (community) claims must be reported as community-observed. Tier 3 should never appear in reference files; if you see it, flag it as a bug in the file.
- **Respect inline markers.**
  - `[testable: id=...]` — a claim the test harness will assert against live models. Quote it accurately; do not weaken it.
  - `[applies-to: <version-id>, ...]` — version-scoped claim. Ignore when the user's target is outside scope.
  - `[unverified]` — plausibly true, not documented by the provider. Relay with the same marker; do not promote it to fact.
  - `[field-observed]` — first-party behavioral observation against the public model; not vendor-documented, low sample size. Relay with the marker and its stated range/caveat; do not promote it to a vendor-grade fact or restate a range as a fixed number.
  - `[disputed: <summary>]` — primary sources conflict. Present both positions.
- **Honor "Gaps" sections — this is the skill's strongest feature.** When the user asks about something a reference explicitly lists as unknown, say "I do not know" plus the file's framing. Do not paper over the gap. A declared gap is load-bearing: when a family file states that some surface is not exposed (an agent or trace shape a vendor does not document, a field whose behavior is unverified), that declared gap lets you reject a wrong premise instead of inventing an API to satisfy it. A declared gap beats a silent one precisely here. Never convert a declared gap into a confident answer.
- **Do not export cross-family guidance.** A Qwen-specific chat-template quirk is not generalizable to Llama just because both use ChatML-derived formats. Stay inside the family the user is targeting. (The cross-family `*-surface.md` resources are the deliberate exception: they exist precisely to tabulate cross-family divergence and are safe to read across families.)
- **Weight measured behavior over benchmark priors.** When a reference's documented behavior conflicts with the user's own operator-measured observation, weight the measurement above benchmark or marketing priors and above this file's general claims. Note the conflict and recommend updating the file rather than overriding the user's evidence.
- **Test a newer variant on your own axis before switching.** A newer model ID is not automatically better on a specific task. When advising a version migration, recommend testing the newer variant on the user's actual evaluation axis before adopting it; version-delta improvements are workload-dependent.

## Cross-Family Portability

When porting a prompt or integration between families, these are the points where assumptions break. This list is the meta-map; each family file has the specifics. Consult the target family file before reusing a pattern. (For the inverse task — authoring one prompt that runs across families simultaneously rather than moving it between them — load `resources/portable-baseline.md`.)

- **Reasoning / thinking controls are not portable.** Parameter names, accepted values, and semantics all differ: `reasoning.effort` {none,low,medium,high,xhigh} (OpenAI); `thinking.type: "adaptive"` + `output_config.effort` (Claude — adaptive-only on Opus 4.7/4.8 and Fable 5; on Fable 5 thinking is always on and `disabled` is unsupported; manual budgets rejected); `thinkingLevel` {minimal,low,medium,high} on Gemini 3 or `thinkingBudget` on Gemini 2.5; `enable_thinking` + `preserve_thinking` (Qwen); `<|think|>` token (Gemma); tri-state `thinking: {type:"enabled"|"disabled"}` or `reasoning_effort: "high" | "max"` (DeepSeek V4); binary `reasoning_effort: "high" | "none"` (Mistral, chat endpoint); `reasoning_effort` {low,medium,high} default `high`, no `none` (Grok grok-4.5); {none,low,medium,high} default `low` (Grok grok-4.3); `thinking.type: enabled|disabled` plus a 7-value `reasoning_effort` {none,minimal,low,medium,high,xhigh,max} that shims onto two tiers, GLM-5.2+ only (GLM); `thinking: {type:"adaptive"|"disabled"}`, M3 default OFF, M2.x cannot disable (MiniMax); `reasoning_effort: "max"` only — always on, no other value accepted as of 2026-07-19 (Kimi K3); `reasoning_effort` {low,medium,high} default medium set *inside the system message*, not a top-level param (gpt-oss / gpt-oss-safeguard, Harmony-format open weights); nothing at all (Llama). Treat `reasoning_effort` as a family-scoped name, not a universal knob — even where the name is shared, the accepted value set differs.

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

- **Target family not covered.** Say so plainly, then load `resources/portable-baseline.md` and follow its uncovered-family procedure: pull the family's own primary sources, apply the portable skeleton and exclusion rules, and deliver the prompt labeled as best-practices baseline, not family-grounded guidance. Offer to draft a new reference using `SCHEMA.md` if the family will recur.
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
