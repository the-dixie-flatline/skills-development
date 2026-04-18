---
family: claude
scope: prompt
versions:
  - claude-opus-4-7
  - claude-sonnet-4-6
  - claude-haiku-4-5
  - claude-haiku-4-5-20251001
retrieved: 2026-04-18
primary_sources:
  - https://platform.claude.com/docs/en/about-claude/models/overview
  - https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7
  - https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
  - https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking
  - https://platform.claude.com/docs/en/build-with-claude/effort
  - https://www.anthropic.com/news/claude-opus-4-7
maturity_note: |
  Claude Opus 4.7 became generally available in early 2026 and introduces
  several breaking changes from Opus 4.6 that shift prompt-engineering
  practice: sampling parameters are rejected, manual thinking budgets are
  rejected, and thinking content is omitted from responses by default. Sonnet
  4.6 and Haiku 4.5 are stable mid-tier and fast-tier models for the 4.x
  generation.
---

# Claude — Prompt-Layer Reference

Portable prompting guidance for the current Claude 4.x generation. API-layer detail (parameters, content-block shapes, beta headers, platform deployment) lives in `claude-prompt-api.md`.

## 1. Model Selection

Three current-generation models. Pick by task axis, not brand.

| Target task                                                | Preferred model                              | Notes                                                                                      |
|------------------------------------------------------------|----------------------------------------------|--------------------------------------------------------------------------------------------|
| Long-horizon agentic coding, hardest reasoning, memory     | `claude-opus-4-7`                            | 1M context; 128K max output; adaptive thinking only; new tokenizer (1.0–1.35× prior count) |
| Balanced intelligence and speed; coding; computer use      | `claude-sonnet-4-6`                          | 1M context; 64K max output; adaptive + manual thinking                                     |
| High-throughput, low-latency, near-frontier at lowest cost | `claude-haiku-4-5` (`claude-haiku-4-5-20251001`) | 200K context; 64K max output; first Haiku with extended thinking                       |

[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

Legacy models (Opus 4.6, Sonnet 4.5, Opus 4.5, Opus 4.1) remain available for pinned workloads. Claude Sonnet 4 and Claude Opus 4 retire **2026-06-15**; Claude Haiku 3 retires **2026-04-19**. Migrate before the retirement dates.
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

Vision is supported on all three current models. Claude Opus 4.7 accepts images up to 2576 px / 3.75 MP — roughly 3× Opus 4.6's 1568 px / 1.15 MP ceiling — and maps model coordinates 1:1 to pixels (no scale-factor math for computer-use workflows).
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]

## 2. Prompt Structure Conventions

Claude uses a structured message protocol with a **top-level system prompt** separate from the user/assistant message list. A `system` prompt is not a role-based message; it is its own field on the request. (See `claude-prompt-api.md` for the exact JSON shape.)

Message roles are `user` and `assistant`, alternating. There is no `tool` role at the message level; tool results go inside user messages as `tool_result` content blocks, and tool calls go inside assistant messages as `tool_use` content blocks.

### XML tags are idiomatic, not required

Anthropic's own best-practices guide recommends XML tags to segment complex prompts. Conventional tags include `<instructions>`, `<context>`, `<input>`, `<example>`, `<examples>`, `<document>`, `<documents>`, `<document_content>`, `<source>`, `<thinking>`, `<answer>`, `<quotes>`, `<info>`. Tags are not a required protocol — they are parse hints. Use consistent names; nest when content has a natural hierarchy.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Ordering rule for long-context prompts

Put long documents and data **at the top** of the prompt; put the query and instructions **at the bottom**. Queries at the end can improve response quality by up to 30% in Anthropic's tests, especially with multi-document inputs.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Prefill migration

Prefilled assistant messages on the last turn are **deprecated on Claude 4.6 and later models, and rejected (400) on Claude Mythos Preview**. Migrate prefill-based patterns as follows:

- Forcing JSON/YAML structure → use the Structured Outputs feature (API-layer) or XML-tag instructions.
- Eliminating preambles → system-prompt instruction: "Respond directly without preamble."
- Steering around refusals → no longer generally needed; Claude's calibration has improved.
- Continuations → move continuation context into the user message explicitly.

[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 3. Instruction Patterns

### Be clear and direct

Claude follows explicit instructions well. Specify desired output format, constraints, and scope. Anthropic's "golden rule": show the prompt to a colleague with no task context — if they would be confused, Claude will be too.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

Prefer "do X" over "do not do Y." Telling Claude what to produce is more reliable than listing things to avoid.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Few-shot examples

3–5 examples is the recommended range. Wrap each in `<example>` tags (and the group in `<examples>`). Make examples relevant to the actual use case, diverse enough to cover edge cases, and structured consistently. Include `<thinking>` tags inside examples if you want Claude's extended-thinking output to follow a particular reasoning pattern.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Role assignment

Role-setting happens in the system prompt. Even a single sentence ("You are a Python coding assistant...") measurably focuses Claude's behavior and tone.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Chain-of-thought when thinking is disabled

When extended thinking is off, prompting for step-by-step reasoning still works. Use `<thinking>` and `<answer>` tags to separate reasoning from the final output. Append self-check directives ("Before finishing, verify your answer against [criteria]") — they reliably catch errors on coding and math.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

[applies-to: claude-opus-4-5] When extended thinking is disabled, Opus 4.5 is particularly sensitive to the word "think" and its variants. Alternatives like "consider," "evaluate," or "reason through" are recommended.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Long-document grounding

Ask Claude to quote relevant passages before carrying out a task. Place quotes in `<quotes>` tags, then derive analysis in a separate section. This cuts through long-document noise.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Literalism on Opus 4.7

[applies-to: claude-opus-4-7] Opus 4.7 interprets prompts more literally than Opus 4.6, particularly at lower `effort` levels. It will not silently generalize an instruction from one item to another. State the scope explicitly: "Apply this formatting to every section, not just the first one."
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]

## 4. Context Window Practical Guidance

- **Opus 4.7 and Sonnet 4.6**: 1M tokens at standard API pricing, no long-context premium.
- **Haiku 4.5**: 200K tokens.

[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]

[applies-to: claude-opus-4-7] Opus 4.7 uses a new tokenizer that may consume 1.0× to 1.35× as many tokens as Opus 4.6 on the same text. Budget `max_tokens` and compaction triggers with this headroom in mind.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]

[applies-to: claude-sonnet-4-6, claude-haiku-4-5] These models have **context awareness** — they track their remaining context-window budget during a conversation. In agent harnesses that compact context, tell Claude so in the system prompt so it does not prematurely wrap up work as the window fills.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

Long-context structural tips: wrap each document in `<document index="n">` with `<document_content>` and `<source>` subtags; put the query at the end.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 5. Multimodal Conventions

All three current models accept text and image input. Multi-image inputs benefit from the improved vision in Opus 4.5+ models; giving Claude a crop tool to zoom into image regions produces consistent uplift on vision evaluations.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

Image-dimension ceilings:

- Opus 4.7: up to **2576 px / 3.75 MP**.
- Opus 4.6 and earlier current-gen: **1568 px / 1.15 MP**.

[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]

For computer-use workflows, Anthropic recommends sending screenshots at 1080p as a performance/cost balance; 720p or 1366×768 for cost-sensitive workloads. Higher resolutions use more tokens without proportional accuracy gains for most tasks.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

Videos are processed as frame sequences; the current Claude generation does not natively consume video files.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 6. Behavioral Quirks

[applies-to: claude-opus-4-7]

- **Response length calibrates to task complexity** rather than defaulting to fixed verbosity. Simple lookups get short answers; open-ended analysis gets long ones. If your product depends on a specific verbosity, tune the prompt — do not rely on defaults.
- **Fewer tool calls by default; more reasoning.** Raising `effort` increases tool usage.
- **Fewer subagents by default.** Steerable via prompt; give explicit guidance on when subagents are warranted.
- **More direct, opinionated tone** with less validation-forward phrasing and fewer emoji than Opus 4.6's warmer default. Re-evaluate voice-sensitive prompts.
- **More regular user-facing progress updates** during long agentic traces. Scaffolding that forced interim summaries on earlier models is usually no longer needed.
- **Strict effort respect**, especially at `low` and `medium`. Shallow reasoning on a complex task means raise effort — not prompt harder.
- **Default design aesthetic**: cream/off-white background (~`#F4F1EA`), serif display type (Georgia, Fraunces, Playfair), italic accents, terracotta/amber accent. Persistent across runs; vague "don't use cream" instructions drift to a different fixed palette. Specify concrete palettes and typography, or ask for multiple proposed directions before building.
- **Thinking content omitted by default** — see API-layer reference for the opt-in field. For products that stream reasoning to users, the default causes a long pause before text output.

[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

[applies-to: claude-sonnet-4-6] Sonnet 4.6 defaults to `effort: "high"`. If you previously used Sonnet 4.5 without setting effort, you will see higher latency unless you set effort explicitly (`medium` for most applications, `low` for chat and classification).
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-04-18]

[applies-to: claude-opus-4-5, claude-opus-4-6] These models are more responsive to the system prompt than prior generations. Aggressive prompt language ("CRITICAL: You MUST use this tool when...") overtriggers. Use normal imperatives ("Use this tool when...").
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Tool-use triggering on code review

[applies-to: claude-opus-4-7] Review harnesses tuned for Opus 4.6 with language like "only report high-severity issues" or "don't nitpick" will see lower measured recall on Opus 4.7 — Opus 4.7 follows those filters more faithfully and suppresses findings it judges below the stated bar. Move filtering to a separate verification pass, or instruct the finding stage to report everything with confidence and severity scores for downstream ranking.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 7. Anti-Patterns

- **Do not prefill the last assistant turn** on Claude 4.6 or later models. It is deprecated; Mythos Preview rejects it with 400. Use Structured Outputs, XML-tag instructions, or system-prompt directives instead.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

- **Do not use CAPS-and-emphasis prompt language** ("CRITICAL: You MUST") on 4.5+ models. It causes overtriggering. Write normal imperatives.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

- **Do not carry over Opus 4.5 anti-laziness scaffolding** to 4.6+ models. It leads to overtriggering on tools and skills. Tune back aggressive "if in doubt, use X" guidance.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

- **Do not set `temperature`, `top_p`, or `top_k` on Opus 4.7** (API-layer rejection, 400). Previously-used `temperature=0` for determinism never produced identical outputs anyway; remove the parameter. See the API file for the migration.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]

- **Do not rely on vague negative design prompts** ("make it clean and minimal") to escape Opus 4.7's cream/serif default. They shift to a different fixed palette, not to variety. Specify a concrete palette and typography, or ask the model to propose options first.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

- **Do not force interim progress updates** on Opus 4.7 ("every 3 tool calls, summarize progress"). The model already provides regular updates; scaffolding interferes.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, retrieved 2026-04-18]

- **Do not use prompting to cap thinking cost** on Opus 4.6 / Sonnet 4.6 / Opus 4.7 when `effort` or `max_tokens` would do it more directly. Prompt-based steering of thinking is supported but is the less reliable lever.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 8. Gaps

- **Mythos Preview prompting differences** are noted briefly in several primary sources but not covered in depth here. Access is invitation-only; practitioner experience with Mythos is not broadly representative.
- **Haiku 4.5 extended-thinking patterns** are new as of this generation; the best-practices guide does not yet include Haiku-specific thinking recommendations. Use adaptive-thinking guidance from Sonnet 4.6 as a starting point and validate on Haiku workloads.
- **Language-specific quirks** across Claude's multilingual support are not covered in retrieved Tier 1 sources. Non-English prompting observations here should be treated as extensions of the English guidance until primary sources document otherwise.
- **Managed Agents vs direct Messages-API prompting differences**: Claude Managed Agents abstracts most of the adaptation covered here; this reference targets direct Messages-API usage. See Anthropic's Managed Agents documentation for the abstracted surface.
