# LLM Family Prompt-Engineering References

A public reference library for prompting current-generation LLMs across providers. Organized by model family, split by scope.

## What This Is

Evidence-tiered prompting references for the production LLM families practitioners actually ship against. Each family has up to two files:

- `resources/{family}-prompt.md` — Portable prompt-layer guidance. Usable from any interface (web UI, third-party wrapper, direct API).
- `resources/{family}-prompt-api.md` — API-layer guidance (chat templates, sampling defaults, reasoning toggles, tool-use protocols, structured output, deployment flags).

The split is deliberate. Someone writing a Gemini webui prompt loads only `gemini-prompt.md`; someone building against the Claude API loads both. Each file is self-contained.

## Intended Audience

- Practitioners, consultants, and framework authors writing prompts against specific LLM families.
- Evaluators who would rather read one cross-provider reference than seven bookmark folders.
- A Claude skill routing layer (`SKILL.md` in this directory) that loads the right family reference on demand.
- A test harness (separate, planned) that will extract `[testable: ...]`-tagged claims and assert them against live models.

## Files

- `SKILL.md` — Routing layer. Describes when this skill applies and how to pick which resource to load.
- `SCHEMA.md` — Normative schema for every reference file: front-matter shape, section order, source tiers, inline markers, required workflow.
- `README.md` — This file.
- `resources/` — Per-family reference files, named `{family}-prompt.md` and `{family}-prompt-api.md`.

## Coverage

At 0.1.0, nine families are covered: Claude, OpenAI (GPT-5.x / o-series), Gemini, Gemma, Llama, Qwen, Grok, Mistral, DeepSeek. `SKILL.md` carries the authoritative coverage table and the "published" vs "planned" status of each family.

A family listed as "planned" is an honest gap, not a placeholder. If the family you need has not been written, it has not been written. Additional families (Cohere Command, Kimi, Phi, Granite, Jamba, etc.) may be added in future releases; see `CHANGELOG.md` at the repository root for release history.

## Source Policy (summary)

Full policy lives in `SCHEMA.md`. Short version:

- **Tier 1 — Primary.** Provider's own docs, model cards, official GitHub, official blog posts. Required for bare factual claims.
- **Tier 2 — Community.** Arxiv, well-known practitioner notes, framework maintainers' docs. Explicitly flagged as community-reported in the text.
- **Tier 3 — Excluded.** SEO-optimized blogs, AI-generated review sites, secondary aggregators. Not cited.

Every non-trivial claim carries inline provenance and a retrieval date. Every file's front matter declares which model versions its bare claims apply to. Empirically validatable claims carry `[testable: ...]` markers for the future test harness.

## How To Use

**As a human reader.** Load the two files for your target family; ignore the rest. Start with the prompt-layer file. Open the API-layer file only if you control the inference call.

**As a Claude skill consumer.** The `SKILL.md` routing layer selects and loads the right files on demand based on the user's target family and scope.

## Known Limitations

- **Point-in-time snapshots.** Model providers change behavior between releases. Files carry `retrieved:` dates. Content older than 90 days without re-verification should be treated as stale.
- **No test-harness validation yet.** `[testable: ...]` claims are tagged but not yet automatically asserted. When the harness ships, per-family validation status will be tracked.
- **Uneven depth across families.** Providers with denser public documentation (Anthropic, Qwen) support deeper references than providers with thinner coverage (Grok). This is a sourcing limit, not a design choice.
- **No benchmark tables, no model rankings, no buying guides.** Scope is how to prompt a given family, not which family is "best."
- **No pre-current-generation coverage.** Retired model families (Claude 3.x, GPT-4.x, Gemini 2.x, Llama 2/3.x) are not covered except where migration guidance is warranted.

## Contributing

Contributions welcome. Before submitting a PR:

1. Read the repository-level `CLAUDE.md` and this skill's `SCHEMA.md`.
2. Confirm every claim traces to a Tier 1 or Tier 2 source with inline provenance.
3. Confirm no client, employer, or other non-public data has leaked into examples or fixtures.
4. Update `retrieved:` dates where the file was touched.
5. Declare AI assistance in the PR description if substantial.

Critique-oriented PRs — pointing out stale claims, bad sourcing, missing provenance, or drift from the schema — are valued equally with additive PRs.

## License

Inherits from the parent repository. See the repository root for the applicable `LICENSE`.
