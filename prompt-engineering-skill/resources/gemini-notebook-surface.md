---
family: gemini
scope: product-surface
surface: gemini-notebook
retrieved: 2026-07-19
contract-version: 2026-04-18
primary_sources:
  - https://blog.google/innovation-and-ai/products/gemini-notebook/notebooklm-gemini-notebook/
  - https://blog.google/innovation-and-ai/products/notebooklm/better-research-notebooklm/
  - https://blog.google/innovation-and-ai/models-and-research/google-labs/notebooklm-custom-personas-engine-upgrade/
  - https://support.google.com/gemininotebook/answer/16269187
  - https://support.google.com/gemininotebook/answer/16179559
  - https://support.google.com/gemininotebook/answer/17003757
  - https://support.google.com/gemininotebook/answer/16213268
  - https://docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/overview
  - https://docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/api-notebooks
  - https://docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/api-notebooks-sources
  - https://workspace.google.com/products/gemini-notebook/
  - https://knowledge.workspace.google.com/admin/generative-ai/generative-ai-in-google-workspace-privacy-hub
maturity_note: |
  Gemini Notebook is a retrieval product, not a long-context chat surface. The
  distinction is vendor-documented, not inferred: Google publishes both a
  retrieval-and-ranking description and a RAG process diagram for the chat path.
  Prompt craft here is therefore closer to search-query craft than to
  context-window craft, and almost nothing in `gemini-prompt.md` or
  `gemini-prompt-api.md` about sampling, thinking levels, or template tokens
  applies — none of those controls are exposed.

  Product naming is unstable as of this retrieval. The rename to Gemini Notebook
  landed 2026-07-16, three days before this file was written, and Google has not
  rewritten its own back catalog: blog posts, help articles, Cloud documentation
  paths, and the Workspace product page still carry NotebookLM naming in varying
  degrees. Treat both names as referring to one product.
---

# Gemini Notebook — Product Surface Reference

Gemini Notebook (formerly NotebookLM) is a source-grounded research product. Its
chat path retrieves from an indexed corpus of user-supplied sources rather than
loading them into a context window, and it exposes no inference controls at all.
Load this file when the task targets Notebook specifically. Load
`gemini-prompt.md` alongside it only if the task also touches the Gemini API;
the two surfaces share a model family and essentially nothing else.

## 1. Naming and lifecycle

Google renamed NotebookLM to Gemini Notebook on 2026-07-16, announced by Josh
Woodward (VP, Google Labs, Gemini app & AI Studio). The announcement states it
is "the same standalone product, now doing more across the Google ecosystem,"
and in body text: "It remains a standalone product focused on being your premier
research tool."
[source: blog.google/innovation-and-ai/products/gemini-notebook/notebooklm-gemini-notebook/, retrieved 2026-07-19]

Reporting that describes Gemini Notebook and NotebookLM as two complementary
products is not supported by Google's own statements. There is one product and
one notebook store; what differs is which interface you open it in (section 2).

**Namespace state.** Both `support.google.com/notebooklm/...` and
`support.google.com/gemininotebook/...` are live and serve the same articles;
Google's own current pages still link to the older path in places. The Cloud
documentation path retains `notebooklm-enterprise` in its URL while the page
titles say Gemini Notebook Enterprise. No deprecation plan is published for
either legacy path. Do not treat a `notebooklm` URL as stale on that basis alone.

## 2. The grounding split — the highest-value prompt fact

The same notebook answers differently depending on which app you open it in.

- **Gemini Notebook (standalone):** responses are grounded *exclusively* in
  notebook sources.
- **Gemini app:** responses are grounded in notebook sources *and* may also use
  web search and other tools.

Custom instructions are shared across both; changes in one propagate to the
other. Studio artifacts (Audio Overviews, Video Overviews, Infographics, Slide
Decks) are generated only in the standalone app.
[source: support.google.com/gemininotebook/answer/17003757, retrieved 2026-07-19]

The practical consequence: a prompt that depends on strict source confinement is
not portable between the two surfaces, and nothing in the prompt text can enforce
confinement in the Gemini app. Surface selection is the control, not wording.

**The confinement claim is tier-scoped.** On Pro and Ultra desktop, chat gains
agentic capability — web search, code execution, file generation — and Google
states it works "with or without sources."
[source: support.google.com/gemininotebook/answer/16179559, retrieved 2026-07-19]
"Grounded exclusively in your sources" therefore describes the Standard tier of
the standalone app, not the product as a whole. Do not state it unqualified.

## 3. What actually enters the prompt

Google publishes the composition contract, which is unusual for a consumer
surface and worth relying on:

- **Sources:** always, either the full set or the checkbox-selected subset.
- **Notes:** only when explicitly selected.
- **Conversation history:** always.

[source: support.google.com/gemininotebook/answer/16269187, retrieved 2026-07-19]

Conversation history is saved automatically and persists across sessions; a
session can be closed and resumed. Users can delete chat history at any time, and
in shared notebooks a user's chat is visible only to them.
[source: blog.google/innovation-and-ai/models-and-research/google-labs/notebooklm-custom-personas-engine-upgrade/, retrieved 2026-07-19]

## 4. Retrieval mechanics

Retrieval, not context loading, is the documented mechanism:

> When your notebook contains many sources, Gemini Notebook retrieves the most
> relevant information based on your question, then builds a response from it.

[source: support.google.com/gemininotebook/answer/16269187, retrieved 2026-07-19]

The October 2025 engine post documents three further mechanisms that bear
directly on prompt craft:

1. **Query decomposition.** The system "automatically explores your sources from
   multiple angles, going beyond your initial prompt to synthesize findings into
   a single, more nuanced response." The post's process diagram labels the stages
   as an original query producing intermediate questions, then a retrieval and
   ranking step, then output.
2. **Ranking exists.** The same diagram names "Retrieval and Ranking" as a
   discrete stage. Reranking is therefore not wholly undocumented, though no
   algorithm, ordering, or cutoff is published.
3. **Context engineering at scale.** Google states that for very large notebooks
   "careful context engineering is critical in delivering a high quality and
   trustworthy answer, grounded on the most relevant information in your sources"
   — an explicit acknowledgment that not all source material reaches the model.

The post also states Google enabled "the full 1 million token context window of
Gemini in NotebookLM chat across all plans" and increased multiturn conversation
capacity "more than sixfold."
[source: blog.google/innovation-and-ai/models-and-research/google-labs/notebooklm-custom-personas-engine-upgrade/, retrieved 2026-07-19]

**Do not read the 1M context window as "your sources are in context."** A notebook
at the Standard ceiling (50 sources x 500,000 words) exceeds 1M tokens by more
than an order of magnitude. The context window bounds what retrieval can hand to
the model per turn; it does not bound what the notebook can hold. Any guidance
that treats the notebook as a context window is wrong.

### Prompt-craft consequences

- **Specific beats broad.** Google's own troubleshooting names unclear phrasing
  as a failure cause and advises making the question "more precise and
  understandable," explicitly because retrieval must locate the right passages.
  [source: support.google.com/gemininotebook/answer/16269187, retrieved 2026-07-19]
- **Source checkboxes are a retrieval-scoping control** and are more reliable
  than asking the model in prose to ignore a source. Whether unchecking narrows
  the index or filters post-retrieval is undocumented (section 9).
- **Expect decomposition.** A single broad question is expanded into intermediate
  questions automatically. Manually decomposing a question into a numbered list
  of sub-questions duplicates work the system already does; it is not a documented
  improvement.

## 5. Limits that bind prompting

Consumer tiers, per user unless noted:
[source: support.google.com/gemininotebook/answer/16213268, retrieved 2026-07-19]

| | Standard | Plus | Pro | Ultra 20TB | Ultra 30TB |
|---|---|---|---|---|---|
| Notebooks | 100 | 200 | 500 | 500 | 500 |
| Sources / notebook | 50 | 100 | 300 | 500 | 600 |
| Chats / day | 50 | 200 | 500 | 2.5K | 5K |
| Deep Research | 10/month | 3/day | 20/day | 75/day | 200/day |
| Audio Overviews / day | 3 | 6 | 20 | 100 | 200 |

Per source: **500,000 words, or 200MB for local uploads**, no page limit. Import
fails on word-count overage, size overage, or a copy-protected PDF.
[source: support.google.com/gemininotebook/answer/16269187, retrieved 2026-07-19]

**The size ceiling differs on enterprise: 500 MB or 500,000 words per source**,
not 200MB. Do not quote 200MB as a product-wide figure.
[source: docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/overview, retrieved 2026-07-19]

**There is no published limit on chat query length or on the custom-instructions
field.** This is a confirmed absence, not an unchecked one — the binding limits
are per-source size, sources per notebook, and chats per day. Anyone reasoning
about "prompt size limits" on this surface is reasoning about the wrong constraint.

**XLSX word counting is not what it looks like.** Conversion inserts structural
characters for rows and cell boundaries which count toward the 500,000-word
limit, so a spreadsheet with a visible word count well under the ceiling can
still fail import. Roughly 150,000 active cells per sheet; multi-sheet workbooks
are processed one sheet at a time.
[source: docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/overview, retrieved 2026-07-19]

## 6. Chat configuration

Configure Chat exposes a conversational style (Default, Learning Guide, Custom)
and a response length (Default, Longer, Shorter). Custom accepts a goal, voice,
or role.
[source: support.google.com/gemininotebook/answer/16179559, retrieved 2026-07-19]

This is the only steering control on the surface. There is no temperature, no
thinking level, no system prompt, no template access. Guidance written for the
Gemini API's `generation_config` has no analog here.

Google publishes worked examples of custom goals, including a research-advisor
framing that challenges assumptions, a marketing-strategist framing constrained
to action plans, and a multi-perspective framing that analyzes material as
academic, creative strategist, and skeptical reviewer in one pass. See the
October 2025 post for the full text rather than paraphrasing it.
[source: blog.google/innovation-and-ai/models-and-research/google-labs/notebooklm-custom-personas-engine-upgrade/, retrieved 2026-07-19]

## 7. Refusal and citation behavior

**Out-of-scope refusal is documented, with a worked example.** Creative or
generative requests that go beyond the sources may return "Gemini Notebook can't
answer this question." Google's own example is "rewrite the end of my short
story." Refusal is expected behavior on this surface, not a failure to be
prompted around.
[source: support.google.com/gemininotebook/answer/16179559, retrieved 2026-07-19]

**Safety flags fire on source content.** Sources containing violence, sexuality,
or obscenity can block answers "even in historical contexts." A refusal may
therefore originate in the corpus rather than the prompt — relevant when working
with primary historical material.
[source: support.google.com/gemininotebook/answer/16269187, retrieved 2026-07-19]

**Citation granularity degrades on short sources.** If source content is too
short, the response references the whole document instead of a quoted span. Do
not read a missing inline citation as a grounding failure.
[source: support.google.com/gemininotebook/answer/16269187, retrieved 2026-07-19]

This is a *citation* behavior. It is not evidence that short notebooks bypass
retrieval and load wholesale into context; Google documents no such threshold
(section 9).

## 8. Enterprise surface

Gemini Notebook Enterprise is delivered through Google Cloud, standalone or
within Gemini Enterprise, at $9 USD per license per month with a yearly discount.
[source: cloud.google.com/resources/notebooklm-enterprise, retrieved 2026-07-19]

Published enterprise limits: 500 notebooks/user, 300 sources/notebook, 500
queries/user/day, 500 MB or 500,000 words per source, 20/day each for audio
overviews, video overviews, mind maps, and reports, 15/day for slide decks and
infographics. Enterprise adds DOCX, PPTX, and XLSX source types over consumer.
Five predefined IAM roles; VPC-SC, CMEK, and data residency (US or EU
multi-region); notebooks shareable only within the same Cloud project, never
publicly.
[source: docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/overview, retrieved 2026-07-19]

**Not integrated with Workspace DLP.** Google's mitigation guidance is to use
Information Rights Management (disabling download, copy, or print) on sensitive
Drive files to prevent ingestion.
[source: knowledge.workspace.google.com/admin/generative-ai/generative-ai-in-google-workspace-privacy-hub, retrieved 2026-07-19]

### The API is CRUD and artifacts, not inference

A REST API exists at `{us|eu|global}-discoveryengine.googleapis.com/v1alpha/`
covering `notebooks.create`, `.get`, `.listRecentlyViewed`, `.batchDelete`,
`.share`; `notebooks.sources.batchCreate` (Drive docs/slides, raw text, web URLs,
YouTube), `.uploadFile`, `.get`, `.batchDelete`; and a documented audio-overview
endpoint.
[source: docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/api-notebooks, retrieved 2026-07-19]
[source: docs.cloud.google.com/gemini/enterprise/notebooklm-enterprise/docs/api-notebooks-sources, retrieved 2026-07-19]

**No endpoint accepts a chat query or returns a chat response.** Across the
notebook, source, and audio-overview API pages, there is no inference path. You
can programmatically build and manage a notebook and generate an audio overview;
you cannot programmatically ask it a question. Note also `v1alpha` — this is not
a stable surface. If a user's plan depends on scripted Q&A against a notebook,
say plainly that the documented API does not support it.

`sources.get` returns per-source `metadata.wordCount` and `metadata.tokenCount`.
This is the only documented way to measure a source's token cost, and it is
available only on the enterprise API.

## 9. Disputed

**Chat history retention.** The Workspace product page FAQ states: "Gemini
Notebook does not currently keep a history of questions and responses in chat.
You can save notes by pinning the response."
[source: workspace.google.com/products/gemini-notebook/, retrieved 2026-07-19]
This contradicts both the help center ("Chat history is retained and kept private
to you," with a Delete Chat History control) and the October 2025 engine post
("your conversations will now be automatically saved").

Resolution: the dated blog post and the help center agree and postdate the
undated marketing FAQ, which appears stale by roughly nine months. Rely on
retention being real. Flag the contradiction rather than silently picking, since
the marketing page is still live.

**Enterprise "5x" marketing.** The Cloud page advertises "5x more Audio
Overviews, notebooks, and sources per notebook." Against Standard, actual ratios
are 5x notebooks (100 to 500) and 6x sources (50 to 300), while audio overviews
go 3/day to 20/day. Treat "5x" as approximate marketing, not a computable
multiplier.

**A five-tier Workspace access ladder** (Standard / More / Higher / Expanded /
Highest), including an "Expanded" tier at 400 sources and 1,000 chats/day, has
been reported but is not corroborated by any page verified here. The consumer
table (section 5) and the enterprise table (section 8) are verified; the ladder
is not. Do not cite it. [unverified]

## 10. Anti-patterns

- **Treating the notebook as a context window.** It is a retrieval index. Sizing
  advice derived from the 1M token window is wrong (section 4).
- **Porting API guidance here.** No sampling, thinking level, system prompt,
  chat template, or structured-output control is exposed.
- **Prompting for source exclusion in prose** instead of using the checkboxes.
- **Treating a refusal as a prompt defect.** Out-of-scope refusal is designed
  behavior, and safety refusals can originate in the source corpus.
- **Quoting 200MB as the universal source ceiling.** Consumer is 200MB;
  enterprise is 500 MB.
- **Assuming strict source grounding on paid tiers or in the Gemini app.**
  Both break the confinement invariant (section 2).
- **Promising programmatic Q&A.** The enterprise API has no inference endpoint.

## 11. Gaps

Undocumented across every Google primary source checked as of 2026-07-19, and
corroborated as absent by two independent research passes:

- **Model variant per tier.** Google names the family — Notebook runs on "Gemini
  3.5 and Antigravity"
  [source: blog.google/innovation-and-ai/products/notebooklm/better-research-notebooklm/, retrieved 2026-07-19]
  — but never maps Flash versus Pro to a subscription tier. Tier documentation
  says only "Access," "Higher access," "Highest access" to Gemini models. Do not
  infer a variant.
- **Retrieval internals.** Chunking strategy and size, passages retrieved per
  query, dense versus sparse versus hybrid, ranking algorithm, embedding model.
  Ranking is named as a stage (section 4); nothing about how it works is published.
- **Source-selection semantics.** Whether unchecking narrows the index, filters
  post-retrieval, or masks attention.
- **Any retrieval-versus-wholesale-loading threshold.** No source states that
  small notebooks bypass retrieval. Claims of a documented threshold are wrong.
- **Chat query length and custom-instructions length limits.**
- **Conversation-history truncation.** Capacity increased "more than sixfold";
  how history is truncated or summarized past that is unpublished.
- **Enterprise-versus-consumer divergence in grounding logic or refusal
  behavior.** Google documents control, compliance, and data-handling differences
  only.

When a user asks about any of the above, say it is not documented and name what
Google does publish instead. Do not reason by analogy from Vertex AI Search, the
Gemini API File Search tool, or RAG Engine — those describe different systems,
and Google has not stated that Notebook shares their pipelines.
