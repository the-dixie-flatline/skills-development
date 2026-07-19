---
family: cross-family
scope: portable-baseline
families: [anthropic, openai, gemini, llama, qwen, gemma]
retrieved: 2026-07-19
primary_sources:
  - https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
  - https://developers.openai.com/api/docs/guides/prompt-engineering
  - https://developers.openai.com/api/docs/guides/reasoning-best-practices
  - https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-1_prompting_guide
  - https://ai.google.dev/gemini-api/docs/prompting-strategies
  - https://developer.meta.com/ai/docs/model-cards-and-prompt-formats/llama4/
  - https://huggingface.co/Qwen/Qwen3-32B
  - https://docs.litellm.ai/docs/completion/prompt_formatting
  - https://docs.litellm.ai/docs/completion/input
  - https://openrouter.ai/docs/guides/features/plugins/response-healing
maturity_note: |
  This file serves two routing cases the family references cannot: a prompt that
  must run unmodified across multiple families, and a target family this skill has
  no reference for. Its epistemic ceiling is lower than a family file's: no vendor
  documents cross-model claims, so every cross-family rule here is either labeled
  synthesis across self-scoped Tier 1 guidance or Tier 2 empirical literature.
  Where a family file exists for the target, it always outranks this file.
---

# Portable Baseline — Cross-Family Reference

The lead epistemic fact: **no vendor documents cross-model-family portability claims.** Anthropic's "improves performance across all models" enumerates Claude models only; OpenAI's guidance compares only its own model classes; Google's is scoped to Gemini generations. [source: the three vendor guides in front matter, scope checked per page, retrieved 2026-07-19] Everything cross-family in this file is therefore one of two things, and is labeled as which:

- **Convergence synthesis** — three or more vendors independently document the same practice for their own models. Strong portable default; still not a vendor guarantee for any other family.
- **Tier 2 empirical** — peer-reviewed cross-model measurement, flagged `[community-reported]`.

When this file and a family reference disagree about that family, the family reference wins.

## When this file applies

1. **Multi-family prompt.** One prompt must run unmodified across two or more families (router deployment, fallback chain, A/B across vendors). Load this file plus each covered target family's `{family}-prompt.md`.
2. **Uncovered family.** The target family has no reference in this skill. Load this file alone, tell the user the family is not covered, and deliver a baseline prompt labeled as best-practices baseline, not family-grounded guidance.
3. **Baseline scaffold.** A prompt is being drafted before the target model is chosen. Structure from this file ports; family tuning happens after the target is fixed.

## Portable prompt skeleton

Convergence synthesis across the Anthropic, OpenAI, and Google guides (all retrieved 2026-07-19). Each vendor claim below is self-scoped to that vendor's models; the skeleton is the intersection.

### Separate instructions, context, and examples with explicit delimiters

All three vendors require explicit structural delimiters for complex prompts. They diverge only on preference strength:

- Anthropic recommends XML tags specifically: named content-type tags (`<instructions>`, `<context>`, `<input>`), `<documents>`/`<document index="n">` nesting for multidocument inputs, `<example>`/`<examples>` around few-shot examples. "XML tags help Claude parse complex prompts unambiguously." [source: platform.claude.com claude-prompting-best-practices, retrieved 2026-07-19]
- OpenAI endorses Markdown headers/lists and XML tags, recommending a combination, privileging neither. [source: developers.openai.com prompt-engineering guide, retrieved 2026-07-19]
- Google publishes both an XML-style template (`<role>`, `<constraints>`, `<context>`, `<task>`) and a Markdown-header template (`# Identity`, `# Constraints`, `# Output format`); its only directive is "choose one format and use it consistently within a single prompt." [source: ai.google.dev/gemini-api/docs/prompting-strategies, page updated 2026-06-10, retrieved 2026-07-19]

**Portable rule:** delimit every section; pick one delimiter family per prompt and hold it consistently. XML-style named tags and Markdown headers are both accepted by all three majors; neither is documented as portable-superior. Consistency is the documented requirement, format choice is not.

### Long context first, task last

- Anthropic: place longform data (20k+ tokens) at the top, above query and instructions; end-placed queries "can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs." The figure is vendor-internal with unpublished methodology; treat as vendor-claimed, Claude-scoped. [source: platform.claude.com claude-prompting-best-practices, retrieved 2026-07-19]
- Google: "supply all the context first. Place your specific instructions or questions at the very end of the prompt," bridged by a transition phrase such as "Based on the information above...". [source: ai.google.dev/gemini-api/docs/prompting-strategies, retrieved 2026-07-19]
- OpenAI: developer-message ordering is identity → instructions → examples → context, with context "usually best positioned near the end of your prompt." Note the divergence in emphasis: OpenAI states standing instructions before context; Anthropic and Google put the specific query after the context. [source: developers.openai.com prompt-engineering guide, retrieved 2026-07-19]

**Portable rule:** keep bulky reference material in one contiguous delimited block; never interleave it with instructions; state the specific task after the context, at or near the end, with a short bridge phrase. Standing role/identity framing goes first.

### Few-shot examples: relevant, diverse, delimited

- Anthropic: examples are "one of the most reliable ways to steer Claude's output format, tone, and structure"; make them relevant and diverse, wrap in `<example>` tags, "include 3–5 examples for best results." [source: platform.claude.com claude-prompting-best-practices, retrieved 2026-07-19]
- Google: "always include few-shot examples"; too many cause overfitting; formatting across examples must be consistent, "especially paying attention to XML tags, white spaces, newlines, and example splitters." [source: ai.google.dev/gemini-api/docs/prompting-strategies, retrieved 2026-07-19] Google's Gemini 3 Developer Guide separately cautions that Gemini 3 "may over-analyze verbose or overly complex prompt engineering techniques used for older models" — a vendor-internal tension left unreconciled as of retrieval. [source: ai.google.dev/gemini-api/docs/gemini-3, retrieved 2026-07-19]
- OpenAI: Examples is a standard developer-message section for non-reasoning operation; for reasoning models the guidance inverts — see the next section. [source: developers.openai.com prompt-engineering guide + reasoning-best-practices, retrieved 2026-07-19]

**Portable rule:** when the target's reasoning state is off or unknown-classic, include a small set (roughly 3–5) of relevant, format-consistent examples in their own delimited section. Byte-identical formatting across examples matters; example-format inconsistency is a documented failure source (Google) and an empirically measured one (see Format sensitivity below).

### Explicit output contract

State the output format explicitly in its own section rather than implying it: Google's published templates carry a dedicated `# Output format` / `<task>` constraint section, and Anthropic documents examples as the most reliable output-format steering mechanism. [sources: ai.google.dev/gemini-api/docs/prompting-strategies; platform.claude.com claude-prompting-best-practices; both retrieved 2026-07-19] For a portable prompt this is load-bearing beyond quality: a schema stated in the prompt is the only output constraint that survives crossing families, because structured-output API mechanisms (JSON mode, grammar constraints, schema fields) are family-specific and do not port.

## Reasoning-era corrections

The best-documented correction to classic technique, stated by all three majors — each scoped to its own models:

- Anthropic (thinking-enabled Claude models): general instructions ("think thoroughly") "often produce better reasoning than a hand-written step-by-step plan"; manual CoT prompting is explicitly a fallback for when thinking is off. [source: platform.claude.com claude-prompting-best-practices, retrieved 2026-07-19]
- OpenAI: "Reasoning models will provide better results on tasks with only high-level guidance. This differs from GPT models, which benefit from very precise instructions"; the reasoning-best-practices page adds "Avoid chain-of-thought prompts" and "Try zero shot first" for reasoning models. [source: developers.openai.com prompt-engineering + reasoning-best-practices, retrieved 2026-07-19]
- Google: Gemini 2.5/3 generate internal thinking automatically, so instructing the model to outline, plan, or narrate reasoning steps in the response is "generally not necessary." Same-page exception: "Think very hard before answering" can raise the thinking budget on heavy problems. [source: ai.google.dev/gemini-api/docs/prompting-strategies, retrieved 2026-07-19]

Whether the correction applies is a property of the **API toggle state, not the prompt**: GPT-5.1's `reasoning_effort: "none"` (its literal API default) forces zero reasoning tokens, and in that mode OpenAI documents that non-reasoning-model guidance re-applies — few-shot prompting, and even explicit planning instructions ("prompting the model to think carefully about which functions it plans to invoke can improve accuracy"). [applies-to: gpt-5.1] [source: developers.openai.com/cookbook gpt-5-1_prompting_guide + api/docs/models/gpt-5.1, retrieved 2026-07-19]

**Portable rule:** a prompt that must run across families cannot know each target's reasoning state, so do not hard-code either regime's elicitation. Omit "think step by step" and prescriptive reasoning-step plans (unnecessary-to-harmful on thinking-enabled targets per all three vendors); carry task decomposition as *structure* — delimited sections, explicit output contract, acceptance criteria — which neither regime documents as harmful. Where the deployment controls the toggle per family, set technique per target instead and consult the family file.

## Router-portable exclusion rules

Family-specific affordances that must not appear in a prompt intended to run across families. Each is grounded in at least one vendor-documented instance; the generalized exclusion rule is synthesis.

- **No chat-template control tokens in message content.** Llama 4's template tokens — `<|begin_of_text|>`, `<|header_start|>`/`<|header_end|>`, `<|eot|>` — differ from ChatML, from Mistral's `[INST]`, and from Llama 3's own tokens, and Meta documents that they "are automatically populated when you run inference": the serving layer's chat template applies them. A control token embedded in prompt text is at best inert and at worst misparsed on every other family. [source: developer.meta.com/ai/docs/model-cards-and-prompt-formats/llama4/, retrieved 2026-07-19]
- **No family role-name assumptions.** Role vocabularies are family-scoped — Llama 4's raw template spells the tool-output role `ipython` where OpenAI-shaped APIs use `tool`. [source: developer.meta.com llama4 prompt-format page, retrieved 2026-07-19] Keep portable prompts to plain `system`/`user`/`assistant` semantics and let the integration layer map roles; per-family role divergence lives in the family files and the SKILL.md portability map.
- **No sampling assumptions — temperature=0 is not a safe universal default.** Alibaba's Qwen3 model cards instruct "DO NOT use greedy decoding" in thinking mode (the family default), citing performance degradation and endless repetition, and publish recommended values instead (Temperature=0.6, TopP=0.95, TopK=20, MinP=0). [applies-to: qwen3-32b and Qwen3-family cards] [source: huggingface.co/Qwen/Qwen3-32B, retrieved 2026-07-19] Take sampling from each target's model card; a portable prompt should not encode sampling expectations at all.
- **No reasoning-control vocabulary in prompt text.** Reasoning is controlled by mutually incompatible per-family API parameters, not by prompt content (see the SKILL.md Cross-Family Portability list for the full divergence). Prompt text that names another family's mechanism ("set reasoning_effort to high", "use extended thinking") is noise on every other target.
- **No API-mechanism dependencies.** Structured-output modes, tool-calling protocols, caching markers, and system-prompt precedence rules are all family-specific API surface. The portable prompt carries its constraints in text (explicit output contract, tool-use expectations in prose); the integration layer translates them per family.

### Gateway-layer normalization is documented — and it validates the exclusion rules

Router/gateway vendors document the *mechanism* layer that makes family-neutral prompts viable; they do not document prompt-content authoring (see Gaps). What they do document supports keeping family-specific affordances out of the prompt and letting the gateway translate:

- **LiteLLM** translates OpenAI-format requests per provider: it applies registered Hugging Face chat templates automatically (custom wrappers via `litellm.register_prompt_template()`; template mapping supported for Hugging Face, TogetherAI, Ollama, Petals), and its `drop_params=True` strips provider-unsupported OpenAI parameters instead of surfacing an upstream 400 (default behavior is to raise; only unsupported OpenAI params are dropped). [source: docs.litellm.ai/docs/completion/prompt_formatting + docs.litellm.ai/docs/completion/input, retrieved 2026-07-19]
- **OpenRouter** enforces `response_format` across routed models and ships a Response Healing plugin that validates and repairs malformed JSON output (non-streaming requests) — vendor-documented acknowledgment that structured-output fidelity varies per routed model and is patched at the gateway, not in the prompt. [source: openrouter.ai/docs/guides/features/plugins/response-healing + openrouter.ai/docs/guides/features/structured-outputs, retrieved 2026-07-19]

Consequence for authoring: do not compensate for one family's parameter set or JSON fidelity inside the portable prompt text; state the contract in text and configure normalization at the gateway.

### Format sensitivity is real and does not transfer

All claims in this subsection are Tier 2 peer-track measurements, `[community-reported]`.

2024 baseline, prior-generation models — the direction is the durable finding, not the numbers:

- FormatSpread (ICLR 2024): semantically equivalent formatting changes produced performance differences of up to 76 accuracy points (LLaMA-2-13B, few-shot); sensitivity persisted across model size and instruction tuning; and "format performance only weakly correlates between models." [community-reported] [source: arxiv.org/abs/2310.11324, retrieved 2026-07-19]
- "Does Prompt Formatting Have Any Impact on LLM Performance?" (2024): the same content in plain text, Markdown, JSON, and YAML swung GPT-3.5-turbo performance by up to 40% on a code-translation task; GPT-4 was more robust. No universally best format. [community-reported] [source: arxiv.org/abs/2411.10541, retrieved 2026-07-19]

2025–2026 follow-ups:

- Input-order sensitivity persists on API-accessed models: shuffled input arrangement produced measurable accuracy declines across paraphrasing, relevance-judgment, and multiple-choice tasks; few-shot prompting gave partial mitigation only. The abstract does not name the specific models tested. Phrasing-level sensitivity likewise persists and remains hard to predict (PromptSET benchmark). [community-reported] [sources: arxiv.org/abs/2502.04134 (v2, 2025-05-09); arxiv.org/abs/2502.06065; retrieved 2026-07-19]
- The format tax is **capacity-dependent, not generation-dependent** ("Capacity, Not Format", 2026-06): with information-matched prose controls across 4 models and 5 benchmarks, models with spare capacity absorbed JSON output constraints without degradation (Claude Sonnet: 88.7% JSON vs 89.3% CoT on MATH-Hard), while capacity-limited models degraded sharply under the same schema (Claude Haiku −36.2pp, largely truncation; GPT-4o-mini −28.0pp even with extended token budgets — pure capacity competition). Frontier models are not immune: Claude Opus 4.7 dropped 96.2% → 91.0% on AIME under JSON. The penalty scales with schema complexity. A delayed-structure ablation — reason freely first, apply the format afterward — recovered most of the lost accuracy. [community-reported] [source: arxiv.org/abs/2606.09410, retrieved 2026-07-19]
- **Input-side** format effects are also scale-dependent (single study, chatbot intent-classification, telecom-domain test set, 2024-era models): structured input formats (JSON/YAML) helped small models most — Gemini 1.5 Flash 8B rose from 64.65% (plain text) to 86.27% (JSON), a 21.62pp spread — while larger models showed narrower spreads, and no format won universally (Llama-8B scored best in plain text). [community-reported] [source: mdpi.com/2079-9292/14/5/888, Electronics 14(5):888, 2025-02-24, retrieved 2026-07-19]
- **Declaring the input format helps**: on legal-document extraction (CUAD excerpts), adding task details and an advisory that the input is structured (e.g., that the model receives structured Markdown) to the system prompt raised GPT-4.1 accuracy by roughly 10–13pp. Tested on GPT-4o/GPT-4.1 only. [community-reported] [source: arxiv.org/abs/2505.12837, retrieved 2026-07-19]
- **Mechanism evidence supports explicit structure**: a first-order Taylor analysis finds LLMs disperse (rather than cluster) semantically similar inputs through their layers, and that prompt templates typically exert greater influence on next-token logits than the questions themselves. [community-reported] [source: arxiv.org/abs/2604.18389, retrieved 2026-07-19] A companion line of work argues much measured prompt sensitivity is attributable to *underspecification*: minimally-instructed prompts show high variance and lower target-token logits, while prompts with specific instructions suffer less. [community-reported] [source: arxiv.org/abs/2602.04297, retrieved 2026-07-19] Both point the same direction as the vendor convergence: explicit, specific, delimited structure is the stabilizer.

**Portable consequence:** a format tuned on one family is not evidence it is optimal on another, and single-format evaluation of a multi-family prompt over-fits the evaluation. Keep structure simple and consistent, and evaluate the actual prompt on each actual target before shipping. Two additions from the capacity finding: on a multi-family prompt, expect the same output schema to be free on high-headroom targets and expensive on capacity-limited ones — evaluate the weakest target, not the flagship; and when a strict schema is required from an unknown or capacity-limited target, prefer the delayed-structure pattern (free-form reasoning first, formatting as a second pass) over constraining the entire generation. [community-reported, single study] [source: arxiv.org/abs/2606.09410, retrieved 2026-07-19]

## Uncovered-family procedure

When the target family has no reference in this skill, in order:

1. **Declare the gap first.** State plainly that this skill has no grounded reference for the family, so what follows is portable baseline plus the family's own primary documentation, not verified family guidance. Do not improvise a reference-shaped answer; that is this skill's core discipline.
2. **Pull the family's own primary sources before prompting.** The provider's model card and docs are the authority for: chat template and roles, recommended sampling, reasoning/thinking controls and their multi-turn round-trip rules, and structured-output/tool mechanisms. The Qwen greedy-decoding prohibition above is the standing example of a family-critical fact that only the model card states.
3. **Never hand-assemble the chat template.** For open weights, use the canonical encoder path (the tokenizer's shipped chat template via `apply_chat_template`, or the provider's own encoder); Meta's "automatically populated when you run inference" framing generalizes: template tokens belong to the serving layer. [source: developer.meta.com llama4 prompt-format page, retrieved 2026-07-19; encoder mechanism per huggingface.co/docs/transformers/chat_templating, framework-maintainer documentation, tier 2]
4. **Apply the portable skeleton** with the exclusion rules above: delimited sections held consistent, context-then-task ordering, explicit output contract in text, no control tokens, no family vocabulary, no sampling or reasoning assumptions.
5. **Omit reasoning elicitation** ("think step by step", prescriptive step plans) unless the family's own docs request it — the unknown target may be thinking-enabled, where all three majors document it as unnecessary-to-counterproductive for their models.
6. **Label the deliverable.** The output is a best-practices baseline to be validated against the family's documentation and behavior, not family-grounded guidance. Offer to draft a real reference per `SCHEMA.md` if the family will recur.

## Gaps

- **Cross-family validity of every rule here.** No vendor documents cross-model claims; the skeleton is convergence synthesis over self-scoped Tier 1 guidance. It is the best-documented available default, not a guarantee for any specific family.
- **Reasoning-toggle effect on format sensitivity.** No source in this file measures the same model's format sensitivity with thinking on vs off. The delayed-structure recovery (arxiv.org/abs/2606.09410) is a prompt-level ablation, not a reasoning-toggle comparison; whether enabling a family's thinking mode changes its format tax is unmeasured.
- **Breadth of the capacity-dependence finding.** The capacity-dependent format-tax result rests on one study (4 models, 5 benchmarks, heavily Claude/OpenAI). Independent replication, and coverage of open-weights families, is absent as of 2026-07-19.
- **Mistral and DeepSeek prompting guides.** Not verified into this file; their family references carry their guidance. Whether they align with the Anthropic/OpenAI/Google convergence on delimiters, few-shot, and context-last placement is unchecked.
- **Router-layer prompt-content guidance.** Router vendors document mechanism-layer normalization (parameter dropping, template mapping, output healing — see the gateway subsection), but no verified router-vendor documentation on prompt *content* authoring for heterogeneous targets exists in this file's sources. The content-side exclusion rules above remain built from per-family vendor documentation.
- **Few-shot on current Gemini.** Google's "always include few-shot examples" and its Gemini 3 "may over-analyze verbose or overly complex prompt engineering techniques" caution are in unresolved tension as of 2026-07-19; this file carries both rather than reconciling them.
- **Instruction-first vs instruction-last for short prompts.** The context-first/task-last convergence is documented for long-context inputs. No vendor quantifies ordering effects for short prompts; the skeleton's ordering is applied there as an unverified default. [unverified]
