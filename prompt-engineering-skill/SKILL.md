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

1. **Identify the target family** from the user's prompt, imported SDK (`anthropic`, `openai`, `google-genai`, transformers model ID, etc.), file naming, or an explicit question. If ambiguous, ask before loading anything.
2. **Decide scope.**
   - Prompt-layer only: load `resources/{family}-prompt.md`.
   - API-layer relevant: also load `resources/{family}-prompt-api.md`. Triggers include the user mentioning sampling parameters, tools, structured outputs, reasoning/thinking budgets, streaming, caching, chat template tokens, or specific SDK calls.
3. **Read the loaded file's front matter first.** The `versions:` field and `retrieved:` date decide whether the content still applies. If `retrieved:` is older than 90 days, say so before relying on the content.
4. **Answer using loaded content plus the reading discipline below.** Do not fill gaps from parametric memory; surface them as gaps.

## Coverage Status

| Family  | Prompt-layer | API-layer | Notes              |
|---------|--------------|-----------|--------------------|
| Claude  | published    | published | Covers Opus 4.7 + Sonnet 4.6 + Haiku 4.5 as of 2026-04-18. Flags Opus 4.7 breaking changes |
| OpenAI  | published    | published | Covers GPT-5.4 flagship + 5.4-pro/mini/nano + Codex + Realtime as of 2026-04-18. Responses API primary; Assistants API sunsetting 2026-08-26 |
| Gemini  | published    | published | Covers Gemini 3.1 Pro Preview + 3 Flash + 3.1 Flash-Lite Preview + 2.5 GA as of 2026-04-18 |
| Gemma   | published    | published | Open weights, Apache 2.0. Covers Gemma 4 E2B / E4B / 26B-A4B / 31B as of 2026-04-19. New chat template vs Gemma 3 |
| Llama   | published    | published | Open weights, Meta. Covers Llama 4 Scout + Maverick as of 2026-04-19. No Llama 5 released |
| Qwen    | published    | published | Pilot family. Covers Qwen3.6-Plus closed flagship + Qwen3.6-35B-A3B open weights as of 2026-04-18 |
| Grok    | published    | published | OpenAI-compatible API. Covers Grok 4.20 + 4.1 Fast pairs + multi-agent variant as of 2026-04-19 |
| Mistral | published    | published | Open weights + native API. Covers Mistral 3 family + Small 4 hybrid + Magistral as of 2026-04-19 |
| DeepSeek| published    | published | Covers DeepSeek-V3.2 flagship (chat + reasoner) + V3.2-Speciale as of 2026-04-19. Text-only, MIT license, OpenAI-compatible API |

"Planned" is an honest gap, not a placeholder. When the user asks about an uncovered family, tell them it is not yet covered rather than improvising a reference-shaped answer.

Update this table whenever a reference lands or is removed.

## Reading Discipline

Applies to every loaded reference.

- **Trust tiers.** Tier 1 (provider docs) claims are usable flat. Tier 2 (community) claims must be reported as community-observed. Tier 3 should never appear in reference files; if you see it, flag it as a bug in the file.
- **Respect inline markers.**
  - `[testable: id=...]` — a claim the test harness will assert against live models. Quote it accurately; do not weaken it.
  - `[applies-to: <version-id>, ...]` — version-scoped claim. Ignore when the user's target is outside scope.
  - `[unverified]` — plausibly true, not documented by the provider. Relay with the same marker; do not promote it to fact.
  - `[disputed: <summary>]` — primary sources conflict. Present both positions.
- **Honor "Gaps" sections.** When the user asks about something a reference explicitly lists as unknown, say "I do not know" plus the file's framing. Do not paper over the gap.
- **Do not export cross-family guidance.** A Qwen-specific chat-template quirk is not generalizable to Llama just because both use ChatML-derived formats. Stay inside the family the user is targeting.

## Cross-Family Portability

When porting a prompt or integration between families, these are the points where assumptions break. This list is the meta-map; each family file has the specifics. Consult the target family file before reusing a pattern.

- **Reasoning / thinking controls are not portable.** Parameter names, accepted values, and semantics all differ: `reasoning.effort` (OpenAI, some Grok); `thinking.type` + `output_config.effort` (Claude); `thinkingLevel` or `thinkingBudget` (Gemini); `enable_thinking` + `preserve_thinking` (Qwen); `<|think|>` token or `enable_thinking` kwarg (Gemma); `thinking: {type: "enabled"}` or model ID (DeepSeek); binary `reasoning_effort: "none" | "high"` (Mistral Small 4); `reasoning_effort` rejected on most Grok models but repurposed for **agent count** on Grok multi-agent; nothing at all (Llama). Treat `reasoning_effort` as a family-scoped name, not a universal knob.

- **Reasoning-artifact multi-turn handling actively contradicts between families.**
  - Claude: `thinking` blocks **must** be preserved unchanged in tool-use multi-turn.
  - OpenAI: reasoning items **must** be preserved (via `previous_response_id` or explicit inclusion).
  - Gemini: `thoughtSignature` must be passed back.
  - DeepSeek: `reasoning_content` **must not** be sent back — returns HTTP 400.
  - Gemma: strip thoughts from prior-turn history.
  - Qwen: opt-in via `preserve_thinking: True` (default `False`).

  There is no portable rule. Treat every family as bespoke.

- **Role names differ.**
  - `user` / `assistant` / `system` / `tool`: Claude, OpenAI, Qwen, Mistral, Grok, DeepSeek.
  - `user` / `model` (no `assistant`): Gemini, Gemma.
  - `ipython` role for tool output (not `tool`): Llama.
  - `developer` role with elevated precedence: OpenAI (general); DeepSeek (scoped to search-agent scenarios only).

  Prompts copied across families without role remapping will misparse.

- **"OpenAI-compatible" at the wire level is not "OpenAI-equivalent" semantically.** Several non-OpenAI families (Grok, DeepSeek, Mistral, Gemini via shim) accept OpenAI-shaped requests but diverge on behavior: Grok rejects `reasoning_effort` on most models and repurposes it on multi-agent; DeepSeek rejects `reasoning_content` on input; Mistral's `reasoning_effort` is binary. Compatible wire format is a migration convenience, not a behavior guarantee.

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
