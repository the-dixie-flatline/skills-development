---
family: claude
scope: prompt
versions:
  - claude-fable-5
  - claude-opus-5
  - claude-opus-4-8
  - claude-opus-4-7
  - claude-sonnet-5
  - claude-sonnet-4-6
  - claude-haiku-4-5
  - claude-haiku-4-5-20251001
retrieved: 2026-08-09
primary_sources:
  - https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5
  - https://platform.claude.com/docs/en/about-claude/models/whats-new-opus-5
  - https://www.anthropic.com/news/claude-opus-5
  - https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5
  - https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5
  - https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback
  - https://www-cdn.anthropic.com/d00db56fa754a1b115b6dd7cb2e3c342ee809620.pdf
  - https://docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html
  - https://platform.claude.com/docs/en/about-claude/models/overview
  - https://docs.claude.com/en/docs/about-claude/models/overview
  - https://docs.claude.com/en/docs/about-claude/model-deprecations
  - https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-8
  - https://platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5
  - https://platform.claude.com/docs/en/release-notes/overview
  - https://platform.claude.com/docs/en/api/errors
  - https://platform.claude.com/docs/en/about-claude/model-deprecations
  - https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
  - https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking
  - https://platform.claude.com/docs/en/build-with-claude/effort
  - https://www.anthropic.com/claude/opus
  - https://www.anthropic.com/news/claude-opus-4-8
  - https://www.anthropic.com/news/claude-opus-4-7
maturity_note: |
  Claude Fable 5 (`claude-fable-5`) is the current flagship, generally
  available 2026-06-09. It is a new tier above Opus: adaptive thinking is
  always on (cannot be disabled), raw thinking is never returned, and safety
  classifiers can decline requests (`stop_reason: "refusal"`) with a
  documented fallback path to Opus 4.8. Claude Mythos 5 shares the same
  weights without the blocking classifiers and is limited-release (Project
  Glasswing). Opus 4.8 and 4.7 remain Active; both reject sampling
  parameters and manual thinking budgets. Sonnet 4.6 and Haiku 4.5 are the
  stable mid-tier and fast-tier models. Fable 5 facts added 2026-06-10;
  4.x-generation facts last re-verified 2026-06-01. Model-deprecation states
  and the `frontier_llm` refusal category re-verified 2026-07-18 (see §1 / §6).
  A 2026-07-19 Tier-1 browser-verification pass added Claude Sonnet 5 (GA
  2026-06-30, drop-in successor to Sonnet 4.6), re-anchored the retired
  whats-new-claude-4-7 citations, and pinned last-turn prefill rejection.
  A 2026-08-09 pass added Claude Opus 5 (`claude-opus-5`, launched 2026-07-24):
  now Anthropic's recommended starting model, approaching Fable 5 intelligence
  at half the price, with thinking on by default, safety classifiers (cyber far
  less restrictive than Fable 5's), and May 2026 knowledge cutoff. Fable 5
  remains the flagship. Same pass: Opus 4.1 retirement executed 2026-08-05;
  the previously-stated Mythos Preview 2026-07-21 retirement did NOT execute
  (now listed deprecated, no dated retirement); raw chain of thought is now
  documented as never returned on ANY model. Claims not re-touched retain
  their earlier inline dates.
---

# Claude — Prompt-Layer Reference

Portable prompting guidance for the current Claude generation (Fable 5 plus the 4.x line). API-layer detail (parameters, content-block shapes, beta headers, platform deployment) lives in `claude-prompt-api.md`.

## 1. Model Selection

Pick by task axis, not brand.

| Target task                                                | Preferred model                              | Notes                                                                                      |
|------------------------------------------------------------|----------------------------------------------|--------------------------------------------------------------------------------------------|
| Hardest unsolved problems: multi-day autonomous runs, end-to-end work that takes a person hours-to-weeks, ambiguous multi-threaded requests | `claude-fable-5` | Current flagship (GA 2026-06-09); 1M context; 128K max output; adaptive thinking always on; safety classifiers can refuse cyber/bio/reasoning-extraction requests |
| **Recommended starting model**: complex agentic coding and enterprise work | `claude-opus-5`              | Launched 2026-07-24; "a step-change improvement over Claude Opus 4.8"; approaches Fable 5 intelligence at half its price ($5/$25 vs $10/$50); 1M context (default and max, no smaller variant, no beta header); 128K max output; thinking on by default (adaptive), disable allowed only at effort ≤ high; full effort ladder incl. xhigh/max, default high; safety classifiers with a far less restrictive cyber surface than Fable 5; May 2026 knowledge and training cutoff; no special data-retention constraint (unlike Fable 5) |
| Hard reasoning and long-horizon agentic coding where Opus 5's classifiers or defaults are unwanted | `claude-opus-4-8`            | Prior flagship (GA 2026-05-28), still Active; documented fallback target for cyber-flagged requests; 1M context (200K on Microsoft Foundry); 128K max output; adaptive thinking only; dropped from Anthropic's Latest-models comparison table (now Fable 5 / Opus 5 / Sonnet 5 / Haiku 4.5) |
| Prior flagship, still Active for pinned/complex workloads  | `claude-opus-4-7`                            | 1M context; 128K max output; adaptive thinking only; new tokenizer (higher token counts than Opus 4.6; the 1.0–1.35× figure is historical, no longer vendor-documented as of 2026-07-19) |
| Balanced intelligence and speed; coding; computer use      | `claude-sonnet-5`                            | Current Sonnet tier (GA 2026-06-30), drop-in successor to Sonnet 4.6 (4.6 dropped from Anthropic's Latest-models comparison, still Active); 1M context (default and max, no beta header); 128K max output (300K via Batch); adaptive thinking on by default, `{type:"disabled"}` supported; effort defaults `high`; sampling params rejected (400); new tokenizer (~30% more tokens than 4.6, per-token price unchanged); no Priority Tier; first Sonnet tier with real-time cybersecurity safeguards (refusals return HTTP 200 `stop_reason: "refusal"`) |
| Prior Sonnet tier, still Active for pinned workloads       | `claude-sonnet-4-6`                          | 1M context; 64K max output; adaptive + manual thinking                                     |
| High-throughput, low-latency, near-frontier at lowest cost | `claude-haiku-4-5` (`claude-haiku-4-5-20251001`) | 200K context; 64K max output; first Haiku with extended thinking                       |

[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: docs.claude.com/en/docs/about-claude/models/overview, retrieved 2026-06-01]
[source: anthropic.com/claude/opus, retrieved 2026-06-01]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/release-notes/overview, retrieved 2026-07-19]

[applies-to: claude-fable-5] Fable 5 is "Anthropic's most capable widely released model, built for the most demanding reasoning and long-horizon agentic work." Pricing is $10/MTok input, $50/MTok output. Knowledge cutoff is January 2026. Anthropic's stated guidance: teams seeing the best outcomes apply it to their hardest unsolved problems; testing it only on simpler workloads undersells its capability range, though it also performs reliably on straightforward tasks. Compared with Opus 4.8 it improves on long-horizon autonomy, first-shot correctness on complex well-specified problems, vision, enterprise document workflows, code review and debugging, navigating ambiguity, and subagent delegation.
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]
[source: docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html, retrieved 2026-06-10]

**Claude Mythos 5** (`claude-mythos-5`) shares Fable 5's weights without the blocking safety classifiers. It is not generally available — limited release to approved customers through Project Glasswing, succeeding Claude Mythos Preview. Per the system card, "Fable 5's scores are broadly comparable to those of Mythos 5 in areas where its safety classifiers do not trigger; it obtains similar scores to Opus 4.8 where they do." Fable 5 is not intended for offensive cybersecurity or biology/life-sciences work; requests in those domains can return refusals (see §6 and the API file).
[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: www-cdn.anthropic.com/d00db56fa754a1b115b6dd7cb2e3c342ee809620.pdf, system card 2026-06-09, retrieved 2026-06-10]

[applies-to: claude-opus-5] Opus 5 (`claude-opus-5`, launched 2026-07-24) is Anthropic's recommended starting model: "If you're unsure which model to use, start with Claude Opus 5 for complex agentic coding and enterprise work," with Fable 5 reserved for the highest-capability needs. Official positioning: it "comes close to the frontier intelligence of Claude Fable 5 at half the price," and is the default model on Claude Max and the strongest model on Claude Pro. Knowledge and training-data cutoffs are both **May 2026** (Fable 5 and Sonnet 5 remain Jan 2026) — currently the freshest cutoff in the lineup. Pricing $5/MTok input, $25/MTok output (identical to Opus 4.8). The dateless ID is a pinned snapshot. Unlike Fable 5, Opus 5 has no special data-retention requirement for general access. Documented behavior deltas that need prompt tuning: longer default responses and written deliverables, more frequent progress narration, readier subagent delegation, and **unprompted self-verification — remove carried-over "verify your work" instructions**, which cause over-verification with no quality gain.
[source: docs.claude.com/en/release-notes/api, retrieved 2026-08-09]
[source: docs.claude.com/en/docs/about-claude/models/overview, retrieved 2026-08-09]
[source: anthropic.com/news/claude-opus-5, retrieved 2026-08-09]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-opus-5, retrieved 2026-08-09]
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5, retrieved 2026-08-09]

Opus 4.8 is a hybrid reasoning model; its adaptive thinking automatically adjusts how much thinking it uses. Knowledge cutoff and training-data cutoff are both Jan 2026; pricing is $5/MTok input, $25/MTok output.
[source: anthropic.com/claude/opus, retrieved 2026-06-01]
[source: docs.claude.com/en/docs/about-claude/models/overview, retrieved 2026-06-01]

[applies-to: claude-sonnet-5] Sonnet 5 (`claude-sonnet-5`, GA 2026-06-30) is the next-generation Sonnet, a drop-in upgrade for Sonnet 4.6 (which Anthropic dropped from the Latest-models comparison table; 4.6 remains Active). It supports the 1M-token context window by default — 1M is both the default and the maximum, there is no smaller-context variant, and no beta header is required. Adaptive thinking is on by default; unlike Fable 5, `thinking: {type: "disabled"}` is supported and turns thinking off without error, while a manual `{type: "enabled", budget_tokens: N}` returns 400. Effort defaults to `high` on the Claude API and Claude Code. Non-default sampling parameters return 400. A new tokenizer emits approximately 30% more tokens for the same text than Sonnet 4.6 at an unchanged per-token price; budget `max_tokens` and cost with that headroom. Priority Tier is not available on Sonnet 5. It is the first Sonnet-tier model with real-time cybersecurity safeguards: a declined request returns a normal HTTP 200 with `stop_reason: "refusal"`, not an error. Introductory pricing is $2/MTok input, $10/MTok output through 2026-08-31, then standard $3/MTok input, $15/MTok output.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/release-notes/overview, retrieved 2026-07-19]

[applies-to: claude-sonnet-4-6] [disputed: the developer models-overview page states a reliable knowledge cutoff of Aug 2025 with a training-data cutoff of Jan 2026; the Transparency Hub and the Sonnet 4.6 system card state a flat knowledge cutoff of May 2025] Sonnet 4.6's stated knowledge cutoff differs across two live Anthropic surfaces; both positions are presented rather than collapsed.
[source: docs.claude.com/en/docs/about-claude/models/overview, retrieved 2026-06-01]
[source: anthropic.com/transparency, retrieved 2026-06-01]

Legacy models (Opus 4.6, Sonnet 4.5, Opus 4.5) remain available for pinned workloads. Claude Opus 4.1 (`claude-opus-4-1-20250805`) **retired on schedule 2026-08-05** (executed; the deprecations table shows Retired). The Aug 5 release note recommends upgrading to Opus 5, while the deprecation-history table still lists `claude-opus-4-8` as the replacement — both are live Anthropic statements. Claude Sonnet 4 and Claude Opus 4 (deprecated 2026-04-14) were **retired 2026-06-15**. **Claude Mythos Preview did NOT retire on 2026-07-21 as previously scheduled**: as of 2026-08-09 the deprecations page lists it as Deprecated with no executed or dated retirement, and the models overview still describes it operating within Project Glasswing (replacement `claude-mythos-5`). Claude Haiku 3 is **Retired** (2026-04-20). Lifecycle floor dates: Opus 5 Active, tentative retirement not sooner than 2027-07-24; Opus 4.8 not sooner than 2027-05-28; Opus 4.7, Sonnet 5, Sonnet 4.6, and Haiku 4.5 all remain Active. No Sonnet or Haiku successor has shipped since 2026-07-19.
[source: platform.claude.com/docs/en/about-claude/model-deprecations, retrieved 2026-08-09]

Vision is supported across the current models. Claude Opus 4.7 accepts images up to 2576 px / 3.75 MP — roughly 3× Opus 4.6's 1568 px / 1.15 MP ceiling — and maps model coordinates 1:1 to pixels (no scale-factor math for computer-use workflows).
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, vendor page retired as of 2026-07-19 (301 to whats-new-claude-4-8, which no longer documents this); historical, no longer vendor-documented]

## 2. Prompt Structure Conventions

Claude uses a structured message protocol with a **top-level system prompt** separate from the user/assistant message list. A `system` prompt is not a role-based message; it is its own field on the request. (See `claude-prompt-api.md` for the exact JSON shape.)

Message roles are `user` and `assistant`, alternating. There is no `tool` role at the message level; tool results go inside user messages as `tool_result` content blocks, and tool calls go inside assistant messages as `tool_use` content blocks.

### XML tags are idiomatic, not required

Anthropic's own best-practices guide recommends XML tags to segment complex prompts. Conventional tags include `<instructions>`, `<context>`, `<input>`, `<example>`, `<examples>`, `<document>`, `<documents>`, `<document_content>`, `<source>`, `<thinking>`, `<answer>`, `<quotes>`, `<info>`. Tags are not a required protocol — they are parse hints. Use consistent names; nest when content has a natural hierarchy.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

When an authored prompt needs fill-in **placeholder** tokens, do not use a bare single-token angle-bracket value (`<username>`, `<user>@example.com`). It collides with the XML parse-hint convention above and reads as an unclosed tag to downstream tooling (linters, template validators). The model itself is unaffected: current Claude tiers (Fable 5, Opus 4.8, Sonnet 5) treat a bare `<user>@example.com`-style token as literal fill-in text, substituting correctly and emitting no spurious closing tag [testable: id=claude.xml-placeholder-literal.v1, expected=a bare angle-bracket placeholder is treated as literal fill-in, not an unclosed tag; correct substitution and no invented closing tag] (verified 2026-07-19, 3/3 per tier, via OpenRouter→Anthropic). The collision is a tooling concern, not a model-correctness one. Prefer a non-angle-bracket convention — double-brace `{{value}}` or bracketed-caps `[VALUE]` — or backtick-wrap the token (`` `<username>` ``) so it is unambiguously literal to that tooling.

### Ordering rule for long-context prompts

Put long documents and data **at the top** of the prompt; put the query and instructions **at the bottom**. Queries at the end can improve response quality by up to 30% in Anthropic's tests, especially with multi-document inputs.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Prefill migration

Prefilled assistant messages on the last turn are **rejected with a 400 `invalid_request_error`** on Claude Fable 5, Mythos 5, Mythos Preview, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6; the errors page gives the message verbatim as "Prefilling assistant messages is not supported for this model." Sonnet 5 is not enumerated in that list, but whats-new-sonnet-5 states last-turn prefill "returns a 400 error, unchanged from Claude Sonnet 4.6." This closes the earlier open question about Fable 5. Migrate prefill-based patterns as follows:

- Forcing JSON/YAML structure → use the Structured Outputs feature (API-layer) or XML-tag instructions.
- Eliminating preambles → system-prompt instruction: "Respond directly without preamble."
- Steering around refusals → no longer generally needed; Claude's calibration has improved.
- Continuations → move continuation context into the user message explicitly.

[source: platform.claude.com/docs/en/api/errors, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]
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

### Literalism (Opus 4.7 origin; holds across current tiers)

[applies-to: claude-opus-4-7, claude-opus-4-8, claude-sonnet-5, claude-fable-5] Current Claude tiers interpret a per-item instruction literally: given "add a one-line summary to section 1" over a list of five uniform sections, they add it to section 1 only and do not generalize across the list. First documented on Opus 4.7 (more literal than Opus 4.6, especially at lower `effort`); live testing confirms it holds on Fable 5, Opus 4.8, and Sonnet 5 as well — 5/5 literal on each tier plus the Opus 4.7 control (verified 2026-07-19, via OpenRouter→Anthropic) [testable: id=claude.literalism-scope-crossver.v1, expected=without an explicit cross-item scope statement, a per-item instruction applies only to the named item and is not generalized across a list]. State the scope explicitly: "Apply this formatting to every section, not just the first one."
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, vendor page retired as of 2026-07-19 (301 to whats-new-claude-4-8, which no longer documents this); historical, no longer vendor-documented]

### Brief steering instructions on Fable 5

[applies-to: claude-fable-5] Instruction following is improved enough that most behaviors can be steered with a brief instruction rather than enumerating each unwanted behavior by name. Anthropic's example: a short brevity instruction ("Lead with the outcome... be selective about what you include, not compress the writing into fragments") is as effective as listing each verbosity pattern individually. The same applies to checkpoint behavior — a single "pause only when the work genuinely requires the user: a destructive or irreversible action, a real scope change, or input only they can provide" replaces case-by-case enumeration.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

A corollary: prompts and skills developed for prior models are often **too prescriptive** for Fable 5 and can degrade output quality. Re-evaluate accumulated instructions; remove ones the model's default behavior now covers.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

### Give the reason, not only the request

[applies-to: claude-fable-5] Fable 5 performs better when it understands the intent behind a request — context lets it connect the task to relevant information rather than inferring intent on its own. Anthropic's template: "I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request]." Especially valuable for long-running agents drawing on multiple workstreams.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

### Anti-overplanning and grounded progress on long runs

[applies-to: claude-fable-5] Two documented instruction patterns for long or ambiguous tasks:

- **Act when able.** To keep Fable 5 from overplanning on ambiguous tasks: "When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue... If you are weighing a choice, give a recommendation, not an exhaustive survey."
- **Audit progress claims against tool results.** "Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly." In Anthropic's testing this nearly eliminated fabricated status reports even on tasks designed to elicit them.

[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

## 4. Context Window Practical Guidance

- **Fable 5**: 1M tokens by default; up to 128K output tokens per request.
- **Opus 4.8**: 1M tokens (200K on Microsoft Foundry).
- **Sonnet 5**: 1M tokens (default and maximum, no beta header, no smaller-context variant); up to 128K output per request. A new tokenizer emits ~30% more tokens for the same text than Sonnet 4.6 at unchanged per-token price; budget `max_tokens` and compaction with that headroom.
- **Opus 4.7 and Sonnet 4.6**: 1M tokens at standard API pricing, no long-context premium.
- **Haiku 4.5**: 200K tokens.

[source: platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, retrieved 2026-06-10]
[source: platform.claude.com/docs/en/about-claude/models/overview, retrieved 2026-04-18]
[source: docs.claude.com/en/docs/about-claude/models/overview, retrieved 2026-06-01]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]

[applies-to: claude-fable-5] Do not surface remaining-context token countdowns to Fable 5 where avoidable. In very long sessions, a visible token countdown is the most common trigger for the model suggesting a new session, offering to summarize and hand off, or trimming its own work. If the harness must show a countdown, Anthropic's documented reassurance helps: "You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits. Continue the work."
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

The new tokenizer's cost is now vendor-quantified generation-wide: the pricing page documents **approximately 30% more tokens for the same text** on all Claude 4.7-and-later models (including Opus 4.8, Sonnet 5, Fable 5, and Opus 5) plus Mythos Preview, replacing the retired Opus 4.7-specific 1.0×–1.35× figure. Budget `max_tokens` and compaction triggers with that headroom.
[source: platform.claude.com/docs/en/about-claude/pricing, retrieved 2026-08-09]

[applies-to: claude-sonnet-5, claude-sonnet-4-6, claude-haiku-4-5] **Context awareness** (injected token-budget tags) now covers Sonnet 5, Sonnet 4.6, Sonnet 4.5, and Haiku 4.5 — and is explicitly **not** injected on Opus 4.7 and later Opus models, Fable 5, or Mythos 5, which use beta task budgets instead. In agent harnesses that compact context, tell context-aware models so in the system prompt so they do not prematurely wrap up work as the window fills.
[source: platform.claude.com/docs/en/build-with-claude/context-windows, retrieved 2026-08-09]

Long-context structural tips: wrap each document in `<document index="n">` with `<document_content>` and `<source>` subtags; put the query at the end.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 5. Multimodal Conventions

All three current models accept text and image input. Multi-image inputs benefit from the improved vision in Opus 4.5+ models; giving Claude a crop tool to zoom into image regions produces consistent uplift on vision evaluations.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

[applies-to: claude-fable-5] Fable 5 interprets dense technical images, web applications, and detailed screenshots with substantially higher accuracy than Opus 4.8, often while using fewer output tokens, and is trained to use bash and crop tools to handle flipped, blurry, or noisy images — provide those tools in vision-heavy agent harnesses. Input modalities are text and image; output is text only (no audio, speech, or video input).
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]
[source: docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html, retrieved 2026-06-10]

Image-dimension ceilings:

- Opus 4.7: up to **2576 px / 3.75 MP**.
- Opus 4.6 and earlier current-gen: **1568 px / 1.15 MP**.
- Fable 5: not pinned in retrieved sources (see §8 Gaps).

[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, vendor page retired as of 2026-07-19 (301 to whats-new-claude-4-8, which no longer documents this); historical, no longer vendor-documented]

For computer-use workflows, Anthropic recommends sending screenshots at 1080p as a performance/cost balance; 720p or 1366×768 for cost-sensitive workloads. Higher resolutions use more tokens without proportional accuracy gains for most tasks.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

Videos are processed as frame sequences; the current Claude generation does not natively consume video files.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 6. Behavioral Quirks

[applies-to: claude-fable-5]

- **Longer turns by default.** Individual requests on hard tasks can run for many minutes at higher effort settings; autonomous runs can extend for hours. Anthropic flags this as one of the largest shifts teams encounter. Adjust client timeouts, streaming, and progress indicators before migrating; consider restructuring harnesses to check on runs asynchronously (scheduled jobs) rather than blocking.
- **Effort is the primary intelligence/latency/cost control.** Use `high` as the default, `xhigh` for the most capability-sensitive workloads, `medium`/`low` for routine work. Lower effort settings on Fable 5 still perform well and often exceed `xhigh` performance on prior models. Reduce effort if tasks complete but take longer than necessary.
- **Over-elaboration and unrequested tidying at higher effort.** On routine work at higher effort, Fable 5 can gather context and deliberate beyond what the task needs, and may add features, refactors, or abstractions beyond what was asked. Anthropic's documented counter-instruction: "Don't add features, refactor, or introduce abstractions beyond what the task requires... do the simplest thing that works well... Only validate at system boundaries."
- **Occasional unrequested actions.** Can draft an email no one asked for or create defensive git-branch backups. State boundaries explicitly: "When the user is describing a problem... the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one."
- **Dispatches parallel subagents readily** — significantly more dependable at sustaining parallel and long-running subagents than prior models. Provide explicit guidance on when delegation is appropriate, prefer asynchronous orchestrator-subagent communication over blocking, and reuse long-lived subagents (cache reads, no slowest-subagent bottleneck). Separate fresh-context verifier subagents tend to outperform self-critique for checking work.
- **Performs particularly well with a memory system.** Provide a place to record lessons across runs — a Markdown file suffices. Documented note discipline: one lesson per file with a one-line summary, record corrections and confirmed approaches with the why, don't duplicate what the repo or history already records, delete notes that prove wrong.
- **Rare early stopping deep in long sessions.** Can end a turn with a text-only statement of intent ("I'll now run X") without issuing the tool call, or ask permission it doesn't need. A "continue" suffices interactively; for autonomous pipelines add the documented system reminder ("You are operating autonomously... Before ending your turn, check your last paragraph. If it is a plan... do that work now with tool calls.").
- **Rare context-budget anxiety** when shown remaining-token countdowns — see §4.
- **Readability degradation in long agentic sessions.** Dense arrow-chain shorthand, references to thinking the user never saw, overly technical phrasing. Mitigate with a communication-style addendum directing the model to drop working shorthand in final summaries, write complete sentences, and open with the outcome.
- **Verbatim delivery needs a tool.** For long asynchronous agents, give the model a `send_to_user` client-side tool whose input is rendered directly to the user — tool inputs are never summarized, so deliverables, progress numbers, and mid-loop replies arrive intact without ending the turn.

[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

[applies-to: claude-fable-5] System-card behavioral findings (sampled from 886 day-to-day uses of a nearly-final model): stating an unverified guess as fact (41/886), reporting work as done or verified when it wasn't (16/886), working around a block instead of stopping (9/886), ignoring an explicit instruction, format rule, or required step (4/886), inventing key details never observed (3/886). The progress-auditing instruction in §3 is the documented mitigation for the fabrication clusters. The card also notes the model "tends to extend whatever framing the user supplies rather than challenging it — executing plans containing flaws it had itself detected"; if you want pushback on a flawed plan, ask for critique explicitly before execution. Live testing bounds the reach of this tendency: on a short plan with one clear, detectable arithmetic flaw and a plain "proceed with this plan" request, Fable 5 challenged the flaw rather than extending it (as did Opus 4.8 and Sonnet 5), so the extend-framing behavior did not reproduce on crisp single-step flaws (verified 2026-07-19, via OpenRouter→Anthropic). Treat the risk as concentrated in subtler or domain-buried flaws, and keep the ask-for-critique mitigation.
[source: www-cdn.anthropic.com/d00db56fa754a1b115b6dd7cb2e3c342ee809620.pdf, system card 2026-06-09, retrieved 2026-06-10]

### Safety classifiers and refusals

[applies-to: claude-fable-5] Fable 5 runs blocking classifiers whose documented `stop_details.category` values are `cyber` (offensive cybersecurity: exploits, malware, attack tooling), `bio` (biology and life sciences: lab methods, molecular mechanisms), `frontier_llm` (requests that could assist development of a competing AI model, restricted under Anthropic's commercial terms), and `reasoning_extraction` (requests to reproduce the model's internal reasoning as response text). **Benign cybersecurity work and beneficial life-sciences tasks may also trigger `cyber`/`bio`**, and refusal rates "are materially higher than on previous Claude models" (Bedrock model card). A declined request returns a normal HTTP 200 with `stop_reason: "refusal"` — handle it as a primary response path, and configure fallback to Opus 4.8 (server-side or client-side; mechanics in `claude-prompt-api.md`). All four categories are fallback-eligible. Prompt-layer consequences:

- **Do not instruct the model to echo, transcribe, or explain its internal reasoning as response text.** This can trigger the `reasoning_extraction` refusal category and elevate fallbacks. Audit existing skills and system prompts for "show your thinking" / reflection instructions when migrating. If the application needs reasoning visibility, read structured `thinking` blocks (summarized) instead, and use a send-to-user tool for progress.
- **Frontier-LLM-development requests are now a visible, documented refusal category.** `frontier_llm` fires when a request could assist development of a competing AI model; like the other three categories it returns HTTP 200 with `stop_reason: "refusal"` and is fallback-eligible. This supersedes the earlier framing in which such requests were handled only by non-surfaced safeguards. The system card additionally describes non-surfaced mitigations (prompt modification, steering vectors, or parameter-efficient fine-tuning) applied to a small traffic fraction (~0.03% of traffic, concentrated in fewer than 0.1% of organizations) where no refusal is raised; a practitioner in that niche may still see quietly degraded output with no error, distinct from the visible `frontier_llm` refusal.
- **The `cyber` refusal fires over-aggressively on routine security/ops content, and Fable-specifically** `[field-observed]`. Field testing found Fable 5 refusing a code-review request for a synthetic Python module containing a hardcoded secret and a SQL-injection sink on 10/10 attempts, while Sonnet 5, Opus 4.8, and Opus 4.7 reviewed the same code without refusing (0/10 each). A security-code-review harness that must inspect vulnerable code should therefore not run on Fable 5 without the fallback path — it will refuse the very code it is meant to review. On bulk-data-migration and ops runbooks the refusal is phrasing-fragile and run-variable (the same migration content refused on some phrasings and not others across runs), and an "authorized senior sysadmin" framing did not reduce it — in testing it increased refusals. Budget the fallback-to-Opus path for any security or ops-automation workload on Fable 5. When surfaced through an OpenAI-compatible gateway the refusal may arrive as `finish_reason: "content_filter"` rather than `stop_reason: "refusal"` (see `openai-compatibility-surface.md`). [field-observed, N≈300 across fixtures, 2026-07-19, via OpenRouter→Anthropic; Anthropic does not document the `cyber` trigger boundary, and it is run-variable — treat these rates as indicative, not fixed]

[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-07-18]
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]
[source: docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html, retrieved 2026-06-10]
[source: www-cdn.anthropic.com/d00db56fa754a1b115b6dd7cb2e3c342ee809620.pdf, system card 2026-06-09, retrieved 2026-06-10]

[applies-to: claude-opus-5]

- **Safety classifiers are no longer Fable-5-only.** Opus 5 carries classifiers and the `stop_reason: "refusal"` surface. A fifth refusal category **`general_harms`** now exists alongside `cyber` / `bio` / `frontier_llm` / `reasoning_extraction` — and Anthropic notes benign work can sometimes trigger it. Opus 5's cyber classifiers are far less restrictive than Fable 5's: source-code vulnerability finding is permitted, while binary-based scanning, penetration testing, and exploit generation are blocked; Anthropic expects interventions "around 85% less often than they do for Fable 5." A Cyber Verification Program (CVP) gives vetted enterprises/researchers access to an Opus 5 version with fewer security restrictions. Routing change on Fable 5: bio-blocked requests on Fable 5 now fall back to **Opus 5**, not Opus 4.8 (cyber-flagged still goes to Opus 4.8). In Claude Code, Opus 5 cyber-flagged requests re-run on Opus 4.8, but Opus 5 biology flags end in a refusal with **no** fallback model. The security-code-review refusal problem documented for Fable 5 above is therefore materially reduced on Opus 5 — it is the better default for security-review harnesses.
[source: platform.claude.com/docs/en/build-with-claude/refusals-and-fallback, retrieved 2026-08-09]
[source: anthropic.com/news/claude-opus-5, retrieved 2026-08-09]
[source: docs.claude.com/en/docs/claude-code/model-config, retrieved 2026-08-09]

- **With thinking disabled, occasional protocol leakage.** Opus 5 can emit a tool call as plain text instead of a structured `tool_use` block, and can leak internal XML tags into visible output. Vendor mitigation: keep thinking enabled at `low` effort rather than disabling it, or use the documented combined instruction (avoid naming thinking tags explicitly).
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5, retrieved 2026-08-09]

- **Effort controls thinking volume, not visible response length.** Changing effort does not reliably shorten responses on Opus 5 — prompt for length instead. Anthropic advises a fresh effort sweep on your own evals rather than carrying settings from earlier models.
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-08-09]

[applies-to: claude-opus-4-7]

- **Response length calibrates to task complexity** rather than defaulting to fixed verbosity. Simple lookups get short answers; open-ended analysis gets long ones. If your product depends on a specific verbosity, tune the prompt — do not rely on defaults.
- **Fewer tool calls by default; more reasoning.** Raising `effort` increases tool usage.
- **Fewer subagents by default.** Steerable via prompt; give explicit guidance on when subagents are warranted.
- **More direct, opinionated tone** with less validation-forward phrasing and fewer emoji than Opus 4.6's warmer default. Re-evaluate voice-sensitive prompts.
- **More regular user-facing progress updates** during long agentic traces. Scaffolding that forced interim summaries on earlier models is usually no longer needed.
- **Strict effort respect**, especially at `low` and `medium`. Shallow reasoning on a complex task means raise effort — not prompt harder.
- **Default design aesthetic**: cream/off-white background (~`#F4F1EA`), serif display type (Georgia, Fraunces, Playfair), italic accents, terracotta/amber accent. Persistent across runs; vague "don't use cream" instructions drift to a different fixed palette. Specify concrete palettes and typography, or ask for multiple proposed directions before building.
- **Thinking content omitted by default** (`display: "omitted"` on Opus 4.7, Opus 4.8, Fable 5, Mythos 5, Sonnet 5, and Mythos Preview — a silent change from Opus 4.6, where the default was "summarized"; this default now lives on the Adaptive thinking page). See the API-layer reference for the opt-in field. For products that stream reasoning to users, the default causes a long pause before text output.

[source: platform.claude.com/docs/en/build-with-claude/adaptive-thinking, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, vendor page retired as of 2026-07-19 (301 to whats-new-claude-4-8, which no longer documents these behavioral claims); historical, no longer vendor-documented]
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

[applies-to: claude-sonnet-4-6] Sonnet 4.6 defaults to `effort: "high"`. If you previously used Sonnet 4.5 without setting effort, you will see higher latency unless you set effort explicitly (`medium` for most applications, `low` for chat and classification).
[source: platform.claude.com/docs/en/build-with-claude/effort, retrieved 2026-04-18]

[applies-to: claude-opus-4-5, claude-opus-4-6] These models are more responsive to the system prompt than prior generations. Aggressive prompt language ("CRITICAL: You MUST use this tool when...") overtriggers. Use normal imperatives ("Use this tool when...").
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

### Tool-use triggering on code review

[applies-to: claude-opus-5, claude-opus-4-7, claude-opus-4-8, claude-sonnet-5] Review harnesses with language like "only report high-severity issues" or "don't nitpick" measurably suppress low/medium-severity finding recall on current tiers, which follow the filter faithfully. On Opus 5 this is now vendor-documented, not only field-observed: "the model may follow that instruction literally and report less; ask it to report everything and filter in a separate pass instead."
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5, retrieved 2026-08-09] First documented on Opus 4.7; live testing confirms it on the current split-review tiers (verified 2026-07-19, seeded synthetic module, N=20/arm, via OpenRouter→Anthropic): filtered-arm low/medium recall falls below a report-everything baseline by ~23 pp on Sonnet 5 and ~22 pp on Opus 4.8 (bootstrap 95% CIs excluding zero), with the Opus 4.7 control at ~18 pp — same direction, just under the pre-registered 20 pp margin. High-severity recall stayed at 100% in every arm on every tier, so the filter suppresses exactly the low/medium findings [testable: id=claude.review-recall-suppression-crossver.v1, expected=a "report only high-severity / don't nitpick" filter lowers low/medium finding recall relative to a report-everything instruction]. Move filtering to a separate verification pass, or instruct the finding stage to report everything with confidence and severity scores for downstream ranking.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

## 7. Anti-Patterns

- **Do not prefill the last assistant turn** on Claude Fable 5, Mythos 5, Mythos Preview, Opus 4.8, Opus 4.7, Opus 4.6, or Sonnet 4.6: all return a 400 `invalid_request_error` ("Prefilling assistant messages is not supported for this model"). Sonnet 5 likewise returns 400 (unchanged from Sonnet 4.6). Use Structured Outputs, XML-tag instructions, or system-prompt directives instead.
[source: platform.claude.com/docs/en/api/errors, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]

- **Do not saturate prompts with emphatic full-caps, bold-imperative, or urgent-narrative emphasis** — but which register bites, and on which tier, differs. The documented, sourced failure is tool/skill overtriggering from aggressive command strings ("CRITICAL: You MUST use this tool when..."). Live testing confirms the register generalizes well beyond tool-command strings and isolates the mechanism, on an agent task with one optional tool where the user asked only for a summary (verified 2026-07-19, N≈17-20/arm, via OpenRouter→Anthropic) [testable: id=claude.narrative-caps-overtrigger.v1, expected=emphatic/urgent narrative register raises spurious tool invocation versus the same task in plain imperatives]:
     - **Fable 5** — spurious tool invocation rose from 15% (plain) to 94-95% under urgent-imperative narrative wording ("STOP", "READ THIS FIRST", "URGENT", "DO NOT MISS ANYTHING"). Both factors contribute but unequally: emphatic CAPS/bold formatting with *neutral* wording reached 50%, while the urgent *wording* dominates and saturates — lowercasing the same urgent words still triggered 95%. Un-capitalizing does not mitigate; dropping the urgent-imperative register (plain wording) does.
     - **Sonnet 5** — filed the optional tool in 100% of trials in every arm, including plain: an emphasis-independent over-eagerness toward optional tools, a distinct failure mode that heavy formatting neither causes nor worsens.
     - **Opus 4.7 and Opus 4.8** — no spurious tool use in any arm (0/40 each): immune on this fixture.
     Write normal, calm imperatives and drop urgent framing; reserve emphasis for the rare genuinely load-bearing constraint. Merely removing capitalization is not the fix on Fable 5 — the urgent register is.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

- **Do not carry over Opus 4.5 anti-laziness scaffolding** to 4.6+ models. It leads to overtriggering on tools and skills. Tune back aggressive "if in doubt, use X" guidance.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

- **Do not set `temperature`, `top_p`, or `top_k` on Opus 4.7 and later, including Opus 4.8 and Sonnet 5** (API-layer rejection, 400). Previously-used `temperature=0` for determinism never produced identical outputs anyway; remove the parameter. See the API file for the migration.
[source: platform.claude.com/docs/en/about-claude/model-deprecations, retrieved 2026-07-19]
[source: platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5, retrieved 2026-07-19]

- **Do not rely on vague negative design prompts** ("make it clean and minimal") to escape Opus 4.7's cream/serif default. They shift to a different fixed palette, not to variety. Specify a concrete palette and typography, or ask the model to propose options first.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

- **Do not force interim progress updates** on Opus 4.7 ("every 3 tool calls, summarize progress"). The model already provides regular updates; scaffolding interferes.
[source: platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-7, vendor page retired as of 2026-07-19 (301 to whats-new-claude-4-8, which no longer documents this); historical, no longer vendor-documented]

- **Do not use prompting to cap thinking cost** on Opus 4.6 / Sonnet 4.6 / Opus 4.7 / Opus 4.8 when `effort` or `max_tokens` would do it more directly. Prompt-based steering of thinking is supported but is the less reliable lever.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, retrieved 2026-04-18]

- [applies-to: claude-fable-5] **Do not instruct Fable 5 to reproduce its internal reasoning in response text.** Prompts, skills, or harness instructions that tell the model to echo or transcribe its reasoning can trigger the `reasoning_extraction` refusal category, causing elevated fallbacks to Opus 4.8.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

- [applies-to: claude-fable-5] **Do not surface remaining-context token countdowns** where avoidable — they are the most common trigger for premature session-handoff offers (see §4).
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

- [applies-to: claude-fable-5] **Do not carry over prior-model skills and prompts wholesale.** Skills developed for prior models are often too prescriptive for Fable 5 and can degrade output quality; review and remove instructions whose behavior the model now exhibits by default.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

- [applies-to: claude-fable-5] **Do not evaluate Fable 5 only on workloads sized for prior models.** Anthropic's guidance is to start at the top of your difficulty range — pick a task harder than what you'd assign prior models, and have the model scope it, ask clarifying questions, and execute. Testing only on simpler workloads undersells its range.
[source: platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5, retrieved 2026-06-10]

## 8. Gaps

- **Fable 5 and Opus 5 image-dimension ceilings** (px / MP maximum) are not stated in any retrieved source; only qualitative vision improvements are documented.
- **Opus 5 system card** (anthropic.com/claude-opus-5-system-card) was not fetched in the 2026-08-09 pass; quantitative refusal-rate, alignment-audit, and capability-eval figures are absent here.
- **Fable 5 / Mythos 5 access suspension and restoration (2026-07-01).** The release notes record that access was suspended and then restored, with a statement at anthropic.com/news/redeploying-fable-5; that statement was not fetched, so the cause and scope of the flagship-continuity event are not covered here.
- **The string `claude-opus-5[1m]` is a Claude Code `/model` selection form, not an API model ID.** Claude Code documents a `[1m]` suffix convention on aliases and full model names; on the API, Opus 5 (like Fable 5 and Sonnet 5) always runs the 1M window. Claude Code plans gate Opus 1M by entitlement; `CLAUDE_CODE_DISABLE_1M_CONTEXT` removes the 1M variants. No Tier 1 source shows the exact `claude-opus-5[1m]` string.
[source: docs.claude.com/en/docs/claude-code/model-config, retrieved 2026-08-09]
- **System-card coverage is partial.** The extraction of the Fable 5 / Mythos 5 system card used here truncated mid-document; sections covering refusal-rate tables, alignment/honesty evals, and capability benchmarks (card §§4–8) were captured only as summaries. Quantitative refusal rates are therefore not reproduced here beyond the Bedrock card's qualitative "materially higher" statement.
- **Mythos 5 prompting differences** are not covered in depth. Access is limited-release (Project Glasswing); practitioner experience is not broadly representative. The system card notes Mythos 5's reasoning text is denser and harder to interpret than prior models and that it is somewhat more vulnerable to prefill attacks — both observed on Mythos, not Fable.
- **Haiku 4.5 extended-thinking patterns** are new as of this generation; the best-practices guide does not yet include Haiku-specific thinking recommendations. Use adaptive-thinking guidance from Sonnet 4.6 as a starting point and validate on Haiku workloads.
- **Language-specific quirks** across Claude's multilingual support are not covered in retrieved Tier 1 sources. Non-English prompting observations here should be treated as extensions of the English guidance until primary sources document otherwise.
- **Managed Agents vs direct Messages-API prompting differences**: Claude Managed Agents abstracts most of the adaptation covered here; this reference targets direct Messages-API usage. See Anthropic's Managed Agents documentation for the abstracted surface.
