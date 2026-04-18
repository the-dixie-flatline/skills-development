# Reference File Schema

Normative schema for every family reference in `resources/`. When writing or auditing a reference file, this is the source of truth.

## Files Per Family

Each family has up to two references:

1. `{family}-prompt.md` — Prompt-layer guidance. Portable across web UI, third-party wrappers, and direct API use.
2. `{family}-prompt-api.md` — API-layer guidance. Relevant only when the caller controls inference configuration.

### What belongs in `{family}-prompt.md`

- Model selection (which variant for which task, with the selection rule stated)
- System-prompt conventions and role handling
- How the model responds to common prompting patterns: few-shot, CoT directives, role assignment, output-format instructions
- Context window practical guidance (where quality degrades, ordering effects)
- Multimodal input conventions (image placement, caption expectations, video/audio handling)
- Behavioral quirks (refusal patterns, verbosity defaults, hedging tendencies)
- Anti-patterns specific to this family
- Concrete examples where they clarify — not decorate

### What belongs in `{family}-prompt-api.md`

- Chat template raw format with real special tokens (open weights) or message structure (closed APIs)
- Sampling parameter defaults and failure modes
- Thinking / reasoning mode toggles (`thinking.budget_tokens`, `reasoning.effort`, `enable_thinking`, `thinkingConfig`, etc.)
- Tool use / function calling protocol and any parser flags required
- Structured output mechanisms (JSON mode, grammar constraints, schema fields)
- API surface compatibility (OpenAI-compatible, Anthropic-compatible, native endpoint)
- Caching, batch, and streaming specifics
- Inference stack flags (vLLM, SGLang, TGI for open weights; region / data residency for closed)
- Version-specific deprecations and breaking changes

### What belongs in neither

- Tutorial content ("what is an LLM," "what is a token"). Audience is practitioners.
- Benchmark rankings, buying guides, opinion pieces.
- Verbatim copies of provider documentation. Link to primary sources; vendored copies go silently wrong when the source updates.
- Second-person marketing voice. Write to the reader as a technical peer.

## Front Matter

Every reference file starts with YAML front matter.

```yaml
---
family: claude | openai | gemini | gemma | llama | qwen | grok | mistral | deepseek
scope: prompt | api
versions:
  - model-id-1
  - model-id-2
retrieved: YYYY-MM-DD
primary_sources:
  - https://...
  - https://...
maturity_note: |
  One or two sentences flagging age, frontier position, or major caveats
  about the state of the reference.
---
```

- `retrieved:` is the date primary sources were last checked. Content older than 90 days should be re-verified or flagged.
- `versions:` lists every model ID the file's bare claims apply to. Version-scoped claims within the file carry an inline `[applies-to: ...]` marker.
- `primary_sources:` are the Tier 1 URLs the file was grounded in. Enough that a reader can reproduce the sourcing pass.

## Section Order

Headings are stable across files in the same scope. Tooling and readers depend on consistent structure. If a section does not apply (e.g. multimodal for a text-only open-weights model), keep the heading and note the non-applicability in one line.

### `{family}-prompt.md`

1. Model Selection
2. Prompt Structure Conventions
3. Instruction Patterns
4. Context Window Practical Guidance
5. Multimodal Conventions
6. Behavioral Quirks
7. Anti-Patterns
8. Gaps

### `{family}-prompt-api.md`

1. API Surface
2. Chat Template / Message Structure
3. Sampling Parameters
4. Reasoning / Thinking Control
5. Tool Use / Function Calling
6. Structured Outputs
7. Caching, Batch, Streaming
8. Deployment Flags
9. Deprecations and Breaking Changes
10. Gaps

## Source Tiering

Every factual claim must trace to one of three tiers.

### Tier 1 — Primary

Provider's own documentation, model card, official GitHub repo, official blog post. Required to ground a bare factual claim.

### Tier 2 — Community-observed

Arxiv papers, well-known practitioners' published notes, framework maintainers' documentation, reproducible community experiments. Always flagged explicitly as community-reported in the reference text.

### Tier 3 — Excluded

SEO-optimized blogs, AI-generated review sites, secondary news aggregators, low-signal summaries. Do not cite.

Known examples of Tier 3 sources to exclude outright, based on prior sourcing passes: `buildfastwithai.com`, `lovableapp.org`, `lushbinary.com`, `stable-learn.com`, `agentconn.com`, `officechai.com`, `serenitiesai.com`, `financialcontent.com`, `labellerr.com`. Also: Medium posts that summarize a primary source without adding observation.

If a claim appears only in Tier 3 sources, either omit it or mark it explicitly as unverified rumor.

## Inline Provenance

Every non-trivial factual claim carries its source inline.

Single primary source:

```
The default chat template wraps thinking in `<think>...</think>` blocks.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
```

Multiple primary sources: list each. For community-reported behavior, include the tier:

```
Community reports suggest Qwen3.6-35B-A3B overtakes Llama 4 on agentic coding benchmarks.
[tier: 2, source: multiple; representative: example.org/qwen-agentic-coding-writeup]
```

A provenance tag is not required for structural statements obvious from the file's own schema (the section headings themselves, the front-matter fields). It is required for every empirical or version-specific claim.

## Inline Markers

Markers annotate claims for downstream tooling and readers.

- `[applies-to: <version-id>, ...]` — claim is scoped to specific model versions; overrides the file's `versions:` header.
- `[testable: id=<family>.<short-description>.v<N>, expected=<observable>]` — empirically validatable claim; the test harness extracts this.
- `[unverified]` — plausibly true but not documented by the provider.
- `[disputed: <summary>]` — primary sources disagree; state both positions inline.

### Testable claim IDs

Format: `{family}.{short-description}.v{N}`. Increment `N` when the expected behavior changes, so historical test runs stay interpretable.

Reserve the marker for claims where:

- The test is well-defined and deterministic enough to assert.
- Failure of the claim changes the guidance meaningfully.
- The test can be run with publicly available access to the model.

Do not tag claims about architecture, licensing, or provider business decisions. Those are not testable through model output.

## Handling Conflicts and Unknowns

- Conflicting primary sources → `[disputed: ...]`, present both positions.
- Plausible but undocumented behavior → `[unverified]`.
- Genuinely unknown → the file's "Gaps" section. "I do not know" is acceptable content.

Silent gaps are worse than declared gaps.

## Workflow To Write or Update a Reference

1. **Verify current state.** Search the provider's own domain and model cards for the current generation. Do not trust prior assumptions about version numbers, naming, or capability claims.
2. **Pull primary sources.** Fetch docs, model cards, official blog posts. Record the URLs under `primary_sources:` in the front matter.
3. **Draft the prompt-layer file.** Every claim gets provenance. Testable claims get IDs. Version-scoped claims get `[applies-to: ...]` markers.
4. **Draft the API-layer file.** Same discipline.
5. **Self-audit.** Every non-trivial claim has provenance. No Tier 3 sources. No slop language. Gaps section present and honest. Front matter accurate. Section order matches this schema.
6. **Update the SKILL.md coverage table.**

A human review of both files is required before anything lands on a public branch. Do not commit autonomously.

## Anti-Patterns

- Mixing prompt-layer and API-layer content in a single file.
- Sampling parameters, raw chat-template tokens, or SDK calls leaking into `{family}-prompt.md`.
- Claims without provenance.
- Tier 3 sources cited, even as "one example observed."
- Hedging language ("it appears," "may be," "seems to") on claims that are actually established by Tier 1 sources.
- Marketing voice, emojis, performative helpfulness.
- Instructions explaining what an LLM or a token is, rather than how to work with this specific family.
- Verbatim vendored provider documentation.
- Placeholder sections ("TODO," "coming soon," "to be filled in later").
- Reference files that depend on reading another reference file. Each must stand alone.
