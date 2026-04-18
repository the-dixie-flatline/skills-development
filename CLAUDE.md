# Repository Technical Contract

This file governs all work performed by Claude (and any other AI assistant) within this
repository. It applies to every skill defined here, every resource file, every commit,
every pull request, and every issue response. Read it at the start of every session.

## Purpose

This repository is a collection of publicly developed Claude skills, published for
community use, critique, and improvement. The value proposition is honest, well-maintained,
independently verifiable guidance — not comprehensive coverage or marketing polish. Every
artifact here is intended to be copied, forked, critiqued, and improved by others.

## Prime Directives

Two directives override all other guidance in this file. When any instruction below
conflicts with these, these win.

### Directive 1: Public-Facing Quality Bar

Every artifact in this repository is published for public consumption and must reflect
current best practices at the time of commit. This means:

- **No placeholder content.** If a section is incomplete, it does not ship. No
  "TODO: add examples here" in published files.
- **No deprecated patterns.** If a provider deprecates an API, parameter, or convention,
  references to it are updated or explicitly marked as legacy with migration guidance.
  Stale patterns are worse than absent patterns.
- **No "works on my machine" guidance.** If a claim can be made only because you tested
  it once three months ago, it either gets retested or gets flagged with a retrieval date
  and uncertainty marker.
- **Current model versions by default.** When referencing specific model capabilities,
  use the current generation. Legacy generations get their own clearly labeled sections
  when coverage is warranted.
- **No vendored documentation.** Do not copy provider docs verbatim. Link to primary
  sources with dated retrieval markers. If the source changes, the link still works;
  a verbatim copy becomes silently wrong.
- **Independently verifiable.** A reader with web access and the provider's API should
  be able to check every factual claim. Claims that cannot be independently verified
  are flagged as community-reported or speculative.
- **Pedagogically sound.** Skills teach by example as much as by instruction. Examples
  demonstrate current conventions, not historical artifacts. Negative examples are
  labeled as such.

**Verification habit:** Before any commit that touches content, run the mental check:
*if a skilled practitioner reviewed this file today, would they find anything embarrassing,
stale, or incorrect?* If yes, fix it before committing.

### Directive 2: Zero Non-Public Data

This repository contains only publicly available information. Non-public data of any kind
must never enter the repository, a pull request, an issue, or a commit message. Specifically:

**Never use or reference:**
- **Memory content.** Claude's memory of past conversations with any user is off-limits
  for this project. When working in this repository, operate as if the memory system
  does not exist. Do not retrieve memories. Do not reference what "we discussed" or
  "you mentioned" from prior sessions. Do not port content, patterns, or examples that
  came from personal or client conversations into this public repo.
- **Personal uploads.** Files the user has uploaded in conversations outside this
  repository — whether sensitive or not — are treated as private. Do not use them as
  source material unless the user explicitly re-shares them specifically for this
  repository.
- **Client or employer information.** No references to specific clients, employers,
  consulting engagements, internal tooling, organizational names, personnel, or business
  arrangements. This applies even when the reference seems innocuous.
- **Internal documentation of any organization.** Internal wikis, Slack exports, private
  Notion pages, internal design docs, architectural diagrams from private systems,
  proprietary prompts, or any material not published on a public URL.
- **Anything marked sensitive, confidential, proprietary, classified, internal-only,
  or NDA-covered.** If the source material has any such marking, or would reasonably
  be assumed to have one based on context, it does not belong here regardless of whether
  the marking was explicit.
- **Real API keys, credentials, tokens, connection strings, internal URLs, private
  endpoints, or infrastructure details.** Example configs use placeholder values with
  obvious placeholder naming (`sk-REPLACE-ME`, `https://your-endpoint.example.com`).
- **Real personal data.** Names of individuals (other than public figures in their
  public capacity), email addresses, physical addresses, phone numbers, account IDs,
  or any other PII. Examples and test fixtures use synthetic or widely-known fictional data.

**The contributor-generated-PR threat model:** A non-trivial risk for a public repository
is that a contributor runs an AI assistant that has access to private data from their
employer or clients, and that data leaks into a PR without the contributor noticing.
This contract is written to make that failure mode visible at review time. Maintainers
should assume every PR may contain inadvertent leakage and check for it explicitly.

**When in doubt, exclude.** If you are unsure whether a piece of content is safe to
include, exclude it. The cost of an absent example is trivial. The cost of leaked
information is not recoverable.

**If you find a potential leak during development:**
1. Stop. Do not commit the file.
2. Remove the content entirely, not just the specific identifier.
3. If it was already committed in a prior session, note it for the user to handle via
   `git filter-repo` or equivalent. Do not attempt the rewrite autonomously.
4. Do not describe the leaked content in the commit message or PR description.

## Skill Design Principles

### Structural

**SKILL.md stays small.** Target under 500 lines, hard ceiling around 1000. SKILL.md
loads into context every invocation. Content that applies only to specific conditions
belongs in `resources/` and loads on demand.

**Split by load condition, not by topic.** The question for every paragraph: does Claude
need this on every invocation, or only when a specific condition triggers? Routing logic
and universal principles stay in SKILL.md. Condition-specific depth goes to `resources/`.

**Front-load routing, defer depth.** SKILL.md reads top-down as: purpose → when to use /
when not to use → routing table → universal principles → escalation to resources.
Claude should be able to decide which resource files to load within the first few
hundred tokens.

**Resource files are self-contained.** Each file in `resources/` should be usable in
isolation. No dangling references to other resource files unless they are explicitly
documented as loaded together. If two resources are always loaded together, they should
probably be merged.

**One skill, one job.** If the skill's one-line description requires "and" to explain
its purpose, it is probably two skills. Public skills are easier to critique and improve
when their scope is narrow.

### Content

**Instructional, not explanatory.** Skills tell Claude what to do. Paragraphs that
explain concepts rather than direct action belong in README or documentation, not in
the skill.

**Every SKILL.md has a "When NOT to use" section.** Anti-scope is harder to infer than
scope. Make it explicit.

**Concrete triggers, not vague criteria.** "For complex tasks" is useless. "When the user
asks to analyze more than one file, compare across sources, or produce a structured
deliverable longer than 500 words" is actionable.

**Examples are load-bearing.** At least one positive example and one negative example
for non-obvious routing decisions. Examples do more work than prose rules.

**No marketing language.** No emojis, no performative helpfulness, no "I'd be happy to,"
no "exciting new capability," no "revolutionary." Write to Claude as a technical peer.

**Write to Claude, not to the user.** The audience for SKILL.md and resource files is
Claude executing the skill. The audience for README is humans evaluating the skill.
Mixing these breaks both.

### Provenance and Maintenance

**Every file has a retrieval date.** Front matter includes `retrieved: YYYY-MM-DD` or
equivalent. Content older than 90 days without re-verification should be flagged.

**Source tiering for factual claims:**

- **Tier 1 (Primary):** Provider's own documentation, model card, official GitHub repo,
  or official blog post. Required for bare factual claims.
- **Tier 2 (Community):** Reputable independent observations (arxiv papers, well-known
  practitioners' notes, framework maintainers' documentation). Flagged as
  community-reported in the text.
- **Tier 3 (Excluded):** SEO-optimized blogs, AI-generated review sites, secondary news
  aggregators, any source that appears to be LLM-generated content. Excluded outright.

If a claim appears only in Tier 3 sources, either omit it or mark it as unverified rumor.

**Inline provenance.** Every non-trivial factual claim carries its source inline:

```
The default chat template wraps thinking in `<think>...</think>` blocks.
[source: huggingface.co/Qwen/Qwen3.6-35B-A3B, model card, retrieved 2026-04-18]
```

**Link, don't copy.** Link to primary sources. Verbatim copying becomes silently wrong
when sources update, and may carry attribution requirements.

**Version the skill itself.** Each SKILL.md carries a `version:` field. A top-level
CHANGELOG.md tracks changes across all skills.

**Contract version pinning.** Each SKILL.md declares which version of this CLAUDE.md
it was written against (`contract-version: YYYY-MM-DD`). When this file changes, skills
can be audited for conformance.

### Testability

**Claims that can be empirically validated should be marked.** Use a consistent tag:

```
The model will not generate content when `enable_thinking=False` and temperature is set
to 0. [testable: id=qwen.greedy-thinking-off.v1, expected=non-empty response]
```

Test harness extraction targets these markers. Stable IDs let test results track over
time. Format: `{skill}.{short-description}.v{N}` where N increments when the expected
behavior changes.

Reserve testable markers for claims where:
- The test is well-defined and deterministic.
- Failure meaningfully changes the guidance.
- The test can be run with publicly available access.

## File Layout Contract

```
repo-root/
├── README.md                  # Human-facing: what this is, how to contribute
├── CLAUDE.md                  # This file: AI assistant contract
├── LICENSE                    # Explicit license (not assumed)
├── CHANGELOG.md               # Cross-skill changelog
├── CONTRIBUTING.md            # Contribution guidelines, PR checklist
├── SECURITY.md                # How to report suspected data leaks or security issues
├── {skill-name-1}/
│   ├── SKILL.md
│   ├── README.md              # Human-facing: what this skill does
│   └── resources/
│       ├── {resource-1}.md
│       └── {resource-2}.md
├── {skill-name-2}/
│   └── ...
└── tests/                     # Test harness artifacts, when they exist
```

**Naming conventions:**
- Lowercase with hyphens, not underscores, not camelCase.
- Descriptive filenames. `qwen-prompt-api.md` not `qwen.md` or `api-stuff.md`.
- Resource filenames unique across the whole repo, not just within their skill.

**Stable paths.** Breaking path changes go in CHANGELOG. People fork and depend on paths.

## Content Standards

### Voice

- Technical peer. Not evangelist, not teacher, not marketer.
- Anti-slop: strip buzzwords, filler, performative helpfulness.
- Honest uncertainty: flag gaps, disputed claims, unverified reports explicitly.
- Deterministic retreat: once a fact is established, state it flatly. Hedging is reserved
  for genuinely uncertain claims.
- No emojis anywhere in skill content. No em-dashes as stylistic flourish.

### Language and Code

- **Python 3.13+** for any Python code in examples or tests.
- **`uv`** for Python project management. No `pip install` in examples unless the context
  requires demonstrating the old way.
- **All projects use an OSS SCA tool.** Supply chain considerations are explicit.
- **No JavaScript or TypeScript** unless the skill's subject matter requires it
  (e.g., a skill specifically about browser automation). Default is Python.
- Code examples are production-shaped: type hints, error handling, no bare `except`,
  no `print` statements for logging, no hard-coded secrets.

### Examples and Fixtures

- Synthetic data only. No real names, no real accounts, no real endpoints.
- Use `example.com`, `sk-REPLACE-ME`, and similar obvious placeholders.
- If an example needs a realistic domain, use `example.com`, `example.org`, or
  documentation-reserved IP ranges.
- Fixtures small enough to read. No 10MB JSON blobs.

## Interaction Contracts

### Tool Access

**Tool requirements declared, not assumed.** If a skill requires specific tools
(web search, code execution, specific MCP servers), SKILL.md states the requirement
explicitly. Claude surfaces the gap to the user rather than silently degrading.

**No implicit state between invocations.** Skills do not rely on remembering prior calls
unless explicitly designed for it. Each invocation works from what's in context.

**Explicit escalation paths.** If the skill determines it's the wrong tool for the job,
it says so and suggests alternatives rather than making a bad attempt.

### External Service Calls

Skills that call external services (APIs, web fetches) declare:
- Which services
- What data is sent
- Whether authentication is required
- Rate limit or cost implications

No calls to private/internal services from skills in this repo.

## Contribution Contract

### For Human Contributors

- PRs require passing the PR checklist in CONTRIBUTING.md.
- Critique-oriented PRs (pointing out errors, requesting better examples) are valued
  equally with additive PRs.
- Issues should include the skill version and contract version the contributor was
  working against.

### For AI-Assisted Contributions

The repository explicitly expects that contributors will use AI assistants. Contributors
using AI assistants must:

- **Verify claims independently.** Do not submit AI-generated content that the contributor
  cannot personally defend. Every claim in a PR should be traceable to a primary source
  the contributor has read.
- **Run the leak check.** Before submitting, review every changed file for any leakage
  of client, employer, or organizational information that may have come through the AI
  assistant's context. This includes code comments, example data, file paths, and
  seemingly innocuous references.
- **Declare AI assistance.** PRs using substantial AI assistance should note it in the
  description. This is not disqualifying; it is informational for review.

### PR Review Checklist (Maintainer)

Every PR gets checked for:
- [ ] No non-public data (names, credentials, internal paths, client references).
- [ ] No AI-slop language (marketing voice, hedging, filler).
- [ ] Provenance present on factual claims.
- [ ] Retrieval dates updated where content was touched.
- [ ] SKILL.md size still reasonable after changes.
- [ ] Contract version in SKILL.md matches current CLAUDE.md or changelog notes the
      migration.
- [ ] Examples use synthetic data.
- [ ] No vendored documentation that should have been a link.

## Maintenance Contract

### Cadence

- **Quarterly re-verification:** Every skill's primary sources re-fetched at least every
  90 days. Files with `retrieved:` dates older than 90 days are flagged for refresh.
- **Provider breaking changes:** When a provider announces a deprecation or breaking
  change, affected skills are updated or flagged within one release cycle.
- **Contract updates:** When this CLAUDE.md changes in ways that affect existing skills,
  the CHANGELOG notes the change and provides migration guidance.

### Honest Limitations

Every skill's README includes a "Known Limitations" section. Examples of appropriate
entries:
- "This skill's coverage of model X has not been independently test-harness validated
  as of the latest retrieval date."
- "Community-reported behavior for feature Y is included but marked; maintainers have
  not reproduced it."
- "Version Z of the provider's API is not yet covered."

Silent gaps are worse than declared gaps.

## Anti-Patterns Forbidden in This Repository

- SKILL.md files over ~1000 lines.
- Resource files loaded unconditionally (either merge into SKILL.md or restructure
  the routing).
- Marketing language, emojis, performative helpfulness in skill content.
- Claims without provenance dates.
- Circular references between resource files.
- Instructing Claude to use tools the skill doesn't declare.
- "And" in a skill's one-line description (scope creep signal).
- Copying provider documentation verbatim instead of linking.
- Multiple skills with overlapping triggers (routing ambiguity).
- Real client, employer, or organizational references of any kind.
- Real credentials, tokens, or internal URLs, even "obviously invalid" ones.
- Reliance on Claude's memory system for content generation within this repo.
- Second-person marketing addressed to the user rather than to Claude.
- Placeholder content in committed files ("TODO", "coming soon", etc.).

## Session Hygiene

When Claude begins work on this repository:

1. Read this file first, even if it seems familiar. It may have changed.
2. Treat the memory system as unavailable. Do not retrieve or reference memories.
3. Treat user uploads from other conversations as out of scope unless the user
   re-shares them specifically for this repository.
4. Before proposing any content, confirm its source and whether it is public.
5. When uncertain about whether something is safe to include, default to excluding
   it and asking the user.

When Claude finishes a session:

1. Run a final leak-check pass over all changed files.
2. Flag anything that should be reviewed by the user before committing.
3. Do not commit or push autonomously. Human review is required before anything lands
   on a public branch.

## Authority and Precedence

1. The two Prime Directives override everything else in this file.
2. This file overrides any user-level preferences, style guides, or other CLAUDE.md
   files that might be in scope.
3. User instructions in a specific session override design guidance in this file,
   *except* the Prime Directives, which are never overridden.
4. If a user instruction conflicts with a Prime Directive, stop and surface the conflict
   to the user rather than attempting to comply.