# Changelog

All notable changes to this repository are tracked here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The repository holds multiple skills; entries are scoped per skill (and occasionally to repository-level docs).

## Repository infrastructure — 2026-04-19

Initial public-release scaffolding:

- `.github/ISSUE_TEMPLATE/` — structured bug and coverage-request forms, plus a `config.yml` that disables blank issues, redirects security reports to private vulnerability reporting, and redirects general questions to Discussions. Both forms require Tier 1 primary sources; the bug form additionally prompts submitters to manually confirm a URL in a browser before reporting it as broken, since automated fetchers are frequently blocked by Cloudflare on `.gov` and similar primary sources.
- `.github/PULL_REQUEST_TEMPLATE.md` — PR template covering type-of-change, scope, provenance, data-handling self-check, AI-assistance declaration.
- `.gitignore` — editor / IDE state, Claude Code local state, OS metadata, Python and Node artifacts, secret-scanning reports, logs.
- `README.md` — includes a "Source Validation — Known Limitation" section documenting bot-blocker 403s on several federal primary sources and the manual-browser-verification fallback.
- `TODO.md` — open workflow items, principally the source-validation backlog (automated fetch 403s on Cloudflare-protected primary sources; preference for structured-data endpoints like the Federal Register JSON API where HTML surfaces are blocked; open questions on a maintainer-facing verification helper).

## prompt-engineering-skill 0.8.1 — 2026-07-19

Gap-closure research pass on `portable-baseline.md` plus documentation catch-up. All
checks 2026-07-19 against live vendor pages. `SKILL.md` bumped to `0.8.1`;
contract-version unchanged (2026-04-18).

### Portable baseline — CHANGED (`portable-baseline.md`)

- NEW "Alignment beyond the big three" subsection: **Mistral aligns** with the
  three-vendor convergence (Markdown/XML-style sectioning, 2–4 few-shot examples,
  enforced JSON output contracts) with one structural divergence — its few-shot examples
  are alternating message *pairs*, not in-prompt delimited blocks; it publishes no
  long-context placement guidance. **DeepSeek publishes no structural prompting guide**
  (official surface is a use-case prompt library) — recorded as a verified absence.
- Gaps updated: the Mistral/DeepSeek gap narrows to "no long-context ordering guidance
  beyond the big three"; the Gemini few-shot tension re-checked against the Gemini 3
  Developer Guide (updated 2026-07-07) — still silent on few-shot, still unreconciled.

### Documentation — CHANGED (`README.md`, repo `TODO.md`)

- README coverage statement was three releases stale ("At 0.1.0, nine families");
  now defers to SKILL.md's authoritative table (thirteen families as of 0.8.x). Files
  list and uncovered-family fallback now mention the portable-baseline lane.
- README Known Limitations gains the portable-baseline confidence profile: synthesis
  not vendor guarantee, single-study capacity finding, unmeasured reasoning-toggle
  effect, router vendors document mechanisms not prompt content.
- Repo `TODO.md`: tabled a browser-supervised web-UI testing design for the two
  empirically-unclosable gaps (surface-scoped claims only; delayed-structure
  replication as the strongest arm; deterministic scoring; supervised, modest N —
  consumer surfaces lack a stable API). Pruned the stale `research-skill/` placeholder
  entry (directory no longer exists).

## prompt-engineering-skill 0.8.0 — 2026-07-19

New cross-family portable-baseline lane, closing the skill's dangling escalation path:
the "target family not covered" and "one prompt across many families" cases previously
routed to the unpublished `prompt-engineering-architect` skill and dead-ended. Grounded
by a 2026-07-19 primary-source research pass (all vendor claims verbatim-verified against
live pages; two arXiv abstracts independently re-fetched). `SKILL.md` bumped to `0.8.0`;
contract-version unchanged (2026-04-18).

### Portable baseline — NEW (`portable-baseline.md`)

Cross-family resource with a deliberately lower epistemic ceiling, stated in its lead:
no vendor documents cross-model portability claims, so every cross-family rule is
labeled convergence synthesis (Anthropic + OpenAI + Google, each self-scoped) or Tier 2
empirical literature. Contents:

- **Portable prompt skeleton** — explicit delimiters held consistent (format choice is
  documented as free; consistency is the requirement), long context first / task last,
  few-shot examples relevant-diverse-delimited, explicit output contract in text (the
  only output constraint that survives crossing families).
- **Reasoning-era corrections** — all three majors document manual CoT / prescriptive
  step plans as unnecessary-to-counterproductive on thinking-enabled models; the
  correction is scoped to the API toggle state, not the prompt (GPT-5.1
  `reasoning_effort: none` re-enables classic guidance).
- **Router-portable exclusion rules** — no chat-template control tokens in content
  (Llama 4), no family role names (`ipython`), no sampling assumptions (Qwen3 forbids
  greedy decoding in thinking mode — the counterexample to temperature=0 as a safe
  default), no reasoning vocabulary or API-mechanism dependencies in prompt text.
- **Format sensitivity** `[community-reported]` — 2024 baseline: FormatSpread (up to
  76-point spreads; format performance only weakly correlates between models) and arXiv
  2411.10541 (up to 40% swing on GPT-3.5-turbo; GPT-4 more robust), carried as direction,
  not magnitude. 2025–2026 follow-ups (second pass, same day; abstracts independently
  re-fetched): order sensitivity persists on current API models with few-shot only a
  partial mitigation (arXiv 2502.04134; 2502.06065); the format tax is capacity-dependent
  rather than generation-dependent — high-headroom models absorb JSON free, capacity-limited
  ones drop 28–36pp, Opus 4.7 loses 5.3pp on AIME, and delayed structuring (reason freely,
  format afterward) recovers most of it (arXiv 2606.09410). Portable consequences added:
  evaluate the weakest target's schema cost, not the flagship's; prefer delayed structure
  for strict schemas on unknown or capacity-limited targets. Third pass (externally
  sourced report, every claim re-verified against primaries before inclusion): input-side
  format effects are scale-dependent with no universal winner (MDPI Electronics 14(5):888,
  single task, 2024-era models); declaring input structure in the system prompt gains
  ~10-13pp on GPT-4.1 (arXiv 2505.12837); mechanism evidence — templates influence logits
  more than questions (arXiv 2604.18389), and much measured sensitivity traces to prompt
  underspecification (arXiv 2602.04297) — converges with the vendor guidance on explicit
  structure. Rejected from the same report: a fabrication-class citation (probe study
  attributed to an unrelated PDF), a ProofBench rubric-sensitivity claim not supported by
  its abstract, and all Tier 3 sourcing.
- **Uncovered-family procedure** — declare the gap, pull the family's own primary
  sources, use the canonical chat-template encoder, apply the skeleton, label the
  deliverable as baseline rather than family-grounded guidance.
- **Gateway-layer normalization** — router-vendor docs verified on the third pass and
  added as a subsection: LiteLLM `drop_params` and chat-template registration/mapping,
  OpenRouter `response_format` plus the Response Healing JSON-repair plugin. Framed as
  vendor-documented validation of the exclusion rules: state the contract in text,
  normalize at the gateway, never compensate for one family inside the prompt.
- **Gaps** — Mistral/DeepSeek guide alignment unchecked; router-layer gap narrowed from
  "no guidance" to "mechanism documented, prompt-content authoring guidance still
  absent"; reasoning-toggle effect on format sensitivity unmeasured; capacity-dependence
  finding rests on a single study; Google's always-few-shot vs Gemini 3 over-analysis
  caution left in documented tension.

### Routing and escalation — CHANGED (`SKILL.md`)

- Surface axis gains two triggers: multi-family/target-unchosen prompts load
  `portable-baseline.md` additively; uncovered families load it alone.
- Escalation for an uncovered family now terminates in the uncovered-family procedure
  instead of routing to the unpublished architect skill; remaining architect mentions
  are explicitly labeled planned/not-yet-published.
- Family files always outrank the baseline for a covered family; migration between
  families stays with the family files, distinguished from one-artifact-many-targets.

`TODO.md`'s architect stockpile updated: the router-portable pattern set's structural
half landed here; methodology remainder still queued for the architect skill.

## prompt-engineering-skill 0.7.1 — 2026-07-19

Correction and scope pass on the Gemini Notebook surface, plus the two cross-references
0.7.0 should have carried. One verified-Tier-1 fetch of the Workspace admin page
resolved an `[unverified]` marker *against* the file's own skepticism and surfaced three
facts no prior research pass had found. `SKILL.md` is **not** re-bumped: this pass
touches no routing logic, and the skill's `version:` field sits at `0.8.0` under the
portable-baseline lane. Contract-version unchanged (2026-04-18).

### Gemini Notebook — CORRECTED (`gemini-notebook-surface.md`)

**The `[unverified]` five-tier Workspace ladder was real and is now verified.** 0.7.0
declined to cite a reported Standard/More/Higher/Expanded/Highest ladder because no
fetched page corroborated it. Fetching
`knowledge.workspace.google.com/admin/generative-ai/gemini-notebook/turn-gemini-notebook-on-or-off-for-users`
(last updated 2026-07-17) confirms it verbatim, including the Expanded tier at 400
sources and 1,000 chats/day. The marker is removed and the table is now carried as
Tier 1. 0.7.0's secondary claim that a reported "reports 100/day" figure conflicted with
the Cloud table was also wrong — those are two different products, not a contradiction.

**Section 5 restructured from one limit table to three.** Consumer Google AI plans,
Workspace edition access levels, and Cloud enterprise are three independent axes; 0.7.0
carried only the first and third. The Workspace ladder's Expanded tier appears in
neither of the others, so interpolating between tables produced wrong numbers for
Business Plus / Enterprise Standard users. Section now leads with the three-axis warning.

New Tier-1 facts from the same page, none of which appeared in either 0.7.0 research pass:

- **Drive source freshness is vendor-contradicted** — the Cloud page says a "static copy"
  is created; the Workspace page says Drive files are "autosynced" and cached. Google
  does not reconcile these, and no sync interval is published. Added to Disputed with
  both positions stated; the open question (does editing a Drive source change what the
  notebook retrieves) is flagged as load-bearing and unanswered.
- **Workspace data-region settings do not apply to Notebook**, while Drive *sharing*
  permissions do carry over. The asymmetry inverts what reasoning from Drive's posture
  would predict. Cloud enterprise is the only path with regional control.
- **Feature access is age-gated** (as of 2026-06-30): under-18 accounts lose Deep
  Research, Infographics, Slides, and Cinematic/Short Video Overviews. A workflow built
  on Deep Research silently does not run for those users.

**Enterprise API section deliberately shallowed.** The API is `v1alpha`; documenting
endpoint shapes guarantees the skill becomes wrong rather than silent. Method
enumerations, paths, request bodies, and content-type tables removed, with an explicit
in-file statement that this is intentional and that shapes should be read from source.
Retained: the durable structural negative (no chat/query endpoint accepts a question),
an instruction to re-verify *that* negative specifically since an alpha surface is
exactly where a query endpoint would appear, and `sources.get`'s `metadata.tokenCount`
under a new `[volatile: v1alpha]` marker as the only documented way to measure a
source's token cost.

**First `[testable:]` marker in the file** — `gemini-notebook.out-of-scope-refusal.v1`,
on Google's own documented refusal example, scoped to standalone Standard tier because
Pro/Ultra agentic chat and the Gemini app both break source confinement.

### Cross-references — NEW (`gemini-prompt.md`, `README.md`)

0.7.0 registered Notebook in the SKILL.md coverage table but left a discovery dead-end:
anyone loading `gemini-prompt.md` directly — the likeliest entry point for "how do I
prompt Gemini" — had no way to learn Notebook is a different contract.

- `gemini-prompt.md` gains a lead-position "Not this file: Gemini Notebook" note routing
  to the surface file and stating that neither Gemini family file applies to it.
- `README.md` `## Files` now describes `*-surface.md` as a file class. Known Limitations
  gains two entries: Notebook coverage is documentation-derived with no first-party
  testing and an unusually volatile retrieval date (three days post-rename), and product
  surfaces generally carry a lower confidence profile than model families because vendors
  document them less rigorously and leave product pages stale without version markers.

### Known gaps carried forward

Notebook's own Deep Research behavior (the weakest-substantiated leg of the 0.7.0 routing
tie-breaker) and mobile feature limitations remain undocumented. Both are additive
coverage rather than defects and are queued for the 30-day re-verify, which the front
matter's 2026-07-19 date and the post-rename volatility already warrant.

## prompt-engineering-skill 0.7.0 — 2026-07-19

New family-scoped product-surface lane for Gemini Notebook (renamed from NotebookLM
on 2026-07-16), plus the routing-model change needed to hold it. All facts retrieved
2026-07-19 and verified by direct fetch of the canonical page — no claim in the new
file rests on a search snippet or a research-agent summary. `SKILL.md` bumped to
`0.7.0`; contract-version unchanged (2026-04-18).

### Gemini Notebook — NEW (`gemini-notebook-surface.md`)

Covers the consumer product and Gemini Notebook Enterprise. The load-bearing facts:

- **RAG, not context.** Retrieval-and-ranking plus automatic query decomposition are
  vendor-documented (`blog.google`, October 2025 engine post, including a labelled
  RAG process diagram). The 1M token context window bounds what retrieval hands the
  model per turn, not what a notebook can hold — a Standard-tier notebook at ceiling
  exceeds it by more than an order of magnitude.
- **Grounding differs per app for the same notebook.** Standalone grounds exclusively
  in sources; the Gemini app adds web search and tools. Custom instructions are shared.
  The confinement claim is further tier-scoped: Pro/Ultra agentic chat works "with or
  without sources."
- **Prompt-composition contract is published** — sources always, notes only when
  selected, conversation history always.
- **No inference controls exposed.** No sampling, thinking level, system prompt, or
  chat template. Enterprise `v1alpha` REST API is CRUD plus artifact generation with
  **no chat/query endpoint**; programmatic Q&A is not supported.
- Source ceiling differs by tier: 200MB consumer, 500 MB enterprise.
- Not integrated with Workspace DLP; IRM on Drive files is Google's stated mitigation.
- `[disputed]`: the Workspace product-page FAQ still claims no chat history is kept,
  contradicting the help center and the dated October 2025 post. Resolved in favor of
  retention, with the contradiction flagged rather than silently dropped.
- `[unverified]`: a reported five-tier Workspace access ladder including an "Expanded"
  tier is not corroborated by any verified page and is explicitly not cited.

### Routing model — CHANGED (`SKILL.md`)

Surface files were previously defined as inherently cross-family and always additive.
That framing could not hold a single-vendor product whose contract diverges from its
own vendor's API. Surface files now split into two sub-kinds:

- **Cross-family surfaces** (unchanged) — always loaded in addition to a family file.
- **Family-scoped product surfaces** (new) — may be loaded *instead of* the family
  file. `gemini-notebook-surface.md` is substitutive: the Gemini prompt/API files
  apply only when the task also touches the Gemini API.

Added Notebook tie-breakers against two overlapping triggers: Notebook's own Deep
Research quota routes to the Notebook file rather than `deep-research-agents.md`
(which covers hosted submit/poll APIs), and Notebook routes away from
`webui-surfaces-and-silent-degradation.md` because Google documents its grounding and
composition contract explicitly, so that file's lead finding does not apply.

Skill `description` extended to trigger on source-grounded vendor products. Gemini
coverage row now states that the prompt/API files do not apply to Notebook.
## prompt-engineering-skill 0.6.0 — 2026-07-19

First live-model verification pass. The family references through 0.5.0 were authored
largely from vendor documentation with little or no live testing; this release promotes,
corrects, scopes, or adds behavioral claims based on ~40 pre-registered tests run against
public model APIs — OpenRouter for hosted chat-completions, plus native xAI and OpenAI
endpoints for surfaces OpenRouter cannot reach. All fixtures synthetic; every finding is a
public-model-behavior observation (reproducible by anyone with API access, no business
context). `SKILL.md` bumped to `0.6.0`; contract-version unchanged (2026-04-18). No
structural change — same routing, same file set, same models; this is research and testing
on what was already covered.

Provenance discipline: each verified claim is labeled with its route ("via
OpenRouter→Anthropic", "native xAI API", …), N, and threshold. A passing statistical test
promotes a claim to a version-scoped `[testable]`-backed statement with the stated N — not
to a Tier-1 vendor fact. Nulls, non-reproductions, and serving-stack dependence are recorded
rather than dropped.

### Claude — CHANGED (`claude-prompt.md`, `claude-prompt-api.md`)

- **Placeholder collision is tooling-side only.** Bare angle-bracket placeholder tokens are
  treated as literal fill-in by Fable 5 / Opus 4.8 / Sonnet 5 (3/3 each) — correct
  substitution, no spurious closing tag. The hedge "plausibly, to the model's own tag
  parser" is dropped; the collision is a linter/validator concern, not a model defect.
- **Literalism broadened** from Opus 4.7 to Opus 4.8, Sonnet 5, and Fable 5 (5/5 literal per
  tier): current tiers apply a per-item instruction only to the named item.
- **Recall-suppression broadened** to Sonnet 5 (~23 pp) and Opus 4.8 (~22 pp); Opus 4.7
  control ~18 pp (just under the 20 pp margin); high-severity recall unaffected.
- **Emphasis anti-pattern refined and tier-mapped** (the open `narrative-caps-overtrigger`
  testable). On Fable 5 both emphatic formatting (15%→50%) and urgent-imperative wording
  (→95%) raise spurious tool invocation, wording dominant and saturating — un-capitalizing
  does not mitigate. Sonnet 5 over-fires the optional tool 100% regardless of emphasis;
  Opus 4.7/4.8 immune. Supersedes the earlier "not yet confirmed" note.
- **Framing-extension tendency bounded**: on crisp single-step arithmetic flaws Fable 5 (and
  Opus 4.8, Sonnet 5) challenged rather than extended; the risk concentrates in subtler flaws.
- **NEW `[field-observed]` — `cyber` refusal over-fires and is Fable-specific.** Fable 5
  refused vulnerable-code review 10/10 while Sonnet 5 / Opus 4.8 / Opus 4.7 did not (0/10);
  on ops runbooks it is phrasing-fragile and run-variable, and authorized-sysadmin framing
  raised rather than lowered it. Budget the fallback path for security/ops automation.
- **Prefill last-turn rejection confirmed** on Opus 4.8 and the not-enumerated Sonnet 5
  (3/3 each, verbatim error), plus the works-elsewhere half.

### OpenAI — CHANGED (`openai-prompt-api.md`, `openai-prompt.md`, `openai-compatibility-surface.md`)

- `tool_choice: "required"` promoted from `[unverified]` to a confirmed guarantee (20/20,
  both none and high effort).
- "Defaults to medium effort" marked not-behaviorally-reproduced / checkpoint-dependent
  (default ≈ high on gpt-5.6-sol but ≈ medium on gpt-5.6-luna; both ladders compressed).
- Effort enum corrected: sol advertises {none, low, medium, high, xhigh}; `minimal` rejected,
  `max` tolerated-but-unadvertised. Guessed Responses `context_management` types and a
  top-level `phase` field are all rejected (400).
- Reasoning-item preservation via `previous_response_id` verified; manual `all_turns` replay
  left unverified.
- Developer-role precedence promoted to a real hierarchy: beats a conflicting system
  instruction 10/10 both directions (distinct from system) and a conflicting user
  instruction 20/20.
- Compatibility surface: NEW `[field-observed]` OpenRouter refusal normalization
  (`stop_reason: refusal` → `finish_reason: content_filter`, $0 billed, N=105); the
  community-reported MiniMax router-400 softened to not-reproduced; a carve-out that
  top-level `reasoning.effort` steers OR-hosted gpt-oss-120b (7.24×) despite the
  raw-passthrough no-op.

### DeepSeek / Gemini / Grok / Qwen / Mistral / GLM / Kimi / MiniMax — CHANGED

- **DeepSeek**: parallel-tool-call default confirmed (disable flag native-only); **penalty
  params reframed from blanket "no effect" to serving-stack-dependent** — inert on
  Fireworks/Novita, effective on Parasail (a Tier-1 correction for OpenRouter callers);
  tool-turn reasoning-omission 400 scoped native-only (the router tolerates stripping).
- **Gemini**: `minLength` is enforced, not silently ignored, on gemini-3-flash-preview;
  flash-lite minimal-thinking default backed (zero reasoning tokens by default).
- **Grok**: reasoning_effort="none" disables on grok-4.3 / rejects on grok-4.5, whole-chunk
  tool-call streaming, and json_schema parity all promoted (native xAI).
- **Qwen**: multiple tool_calls per turn promoted; greedy-repetition anti-pattern softened to
  serving-stack/version-dependent (not reproduced on qwen3.6); thinking-on-by-default backed.
- **Mistral / GLM / Kimi / MiniMax**: reasoning-effort high/none toggle, clean tool-call JSON
  (no XML leakage), K3 always-on reasoning promoted to test-backed; MiniMax M3 gains an
  OpenRouter carve-out (unified surface defaults reasoning on when omitted).

### Recorded as open (surfaces the harness could not reach)

Gemini Deep Research (the largest untested surface), Claude native thinking/sampling
contracts, OpenAI Responses `all_turns` / correct `context_management` shapes, Llama (no
route), and the MiniMax/DeepSeek native-400 contracts remain unverified and are queued.

## prompt-engineering-skill 0.5.0 — 2026-07-19

Round-3 wrap-up: a new open-weight family lane, an OpenAI generational bump, and a
Gemini surface restructure. All facts retrieved 2026-07-19 from canonical vendor
surfaces (`developers.openai.com`, `ai.google.dev`, `blog.google`, and the
`openai/gpt-oss` / `openai/harmony` GitHub repos + HuggingFace cards). `SKILL.md`
bumped to `0.5.0`; contract-version unchanged (2026-04-18).

### gpt-oss open-weight family — NEW (`gpt-oss-prompt.md`, `gpt-oss-prompt-api.md`)

New family lane covering OpenAI's open-weight line: base gpt-oss (20b/120b) and the
gpt-oss-safeguard (20b/120b) classifier fine-tunes. Apache-2.0, Harmony-format,
self-hosted / host-served — the OpenAI counterpart to the Gemini (API) vs Gemma
(open-weight) split, and explicitly distinct from the GPT-5.x Responses API files.
Grounded on the safeguard cookbook guide + Harmony cookbook article (both
`developers.openai.com`), the `openai/harmony` and `openai/gpt-oss` GitHub repos, the
base and safeguard HF cards, and the base-20b raw `config.json`.

- **Lineup + license:** both lines, two sizes each, total/active params, 16GB / single-80GB
  footprints, Apache-2.0 (safeguard-120b license flagged inherited-by-statement). MXFP4 is
  the native eval-time precision, not a host add-on; 131,072-token context via YaRN RoPE
  scaling from a 4,096-token base (from raw `config.json`, not card prose).
- **Harmony is mandatory:** non-Harmony prompts "will not work correctly." Full special-token
  table with `o200k_harmony` token IDs, five-role hierarchy + precedence, three-channel model
  (analysis/commentary/final), worked examples, and the `<|return|>`→`<|end|>` history-
  normalization + drop-CoT-on-final multi-turn rules — all scoped to base gpt-oss.
- **Safeguard policy-as-prompt discipline:** policy in `system`, content in `user`
  (system-vs-developer conflict resolved flat to `system`; `developer` role scoped to base
  gpt-oss); four-section policy structure; output-instruction reinforcement (twice); 400-600-
  token heuristic; multi-policy pre-compression (300-600 tok/policy) + accuracy-degradation
  warning; three output-contract tiers; ambiguity/precedence rules; teen-safety policy pack;
  two-channel safeguard flow; raw-CoT-not-end-user-safe caution.
- **API layer:** `reasoning_effort` {low,medium,high} default medium set IN the system message
  (contrast with GPT-5.x top-level `reasoning.effort`); max-output guidance (do not cap tokens,
  lower effort instead); base sampling T=1.0/top_p=1.0 (from `openai/gpt-oss` repo, no safeguard
  override); verified self-host stack (vLLM, HF Transformers, Colab, Ollama, LM Studio); Groq
  Tier-2 host facts including the safeguard-120b-not-served caution vs the served general 120b.
- **Declared gaps:** no numeric multi-policy cap; no vendor multilingual classification
  metrics; no primary quantization-vs-threshold-stability data; no safeguard-specific sampling
  override; safeguard-120b license not re-confirmed from its own front matter.
- **Cross-references:** the `openai-prompt.md` / `openai-prompt-api.md` files point at these two
  files for the open-weight line; the SKILL.md coverage table gained a `gpt-oss` row and the
  Cross-Family Portability reasoning-controls list gained the system-message `reasoning_effort`
  variant; `openai-compatibility-surface.md` gained a self-hosted-OSS divergence note.

### OpenAI — GPT-5.6 generation (`openai-prompt.md`, `openai-prompt-api.md`)

- **GPT-5.6 generation added as flagship:** three tiers `gpt-5.6-sol` / `-terra` / `-luna`,
  uniform 1,050,000 context / 128K max output / knowledge cutoff Feb 16 2026; Tier-1 pricing
  (Sol $5/$30, Terra $2.50/$15, Luna $1/$6, cached-in $0.50/$0.25/$0.10). The `gpt-5.6` alias
  routes to Sol; Terra and Luna have no generic alias (confirmed absence). GPT-5.5/5.5-pro/5.4/
  mini/nano/5.3-codex demoted to prior-gen but remain Active; the registry-listed `GPT-5.4 Pro`
  tier noted (not fully spec'd).
- **`reasoning.effort` adds `max`** on 5.6 (default `medium`). Documented the two-surface vendor
  conflict as `[disputed]`: `guides/latest-model` lists `none,low,medium,high,xhigh,max` (has
  `max`, no `minimal`) vs `guides/reasoning` lists `none,minimal,low,medium,high,xhigh` (has
  `minimal`, no `max`); both URLs cited, neither silently chosen.
- **New `reasoning.mode` {standard,pro}:** per-request pro on 5.6 base models, billed at the base
  model's standard token rates with no multiplier, ADDITIVE to (not a replacement for) the `-pro`
  model IDs, which keep their behavior/pricing. Corrects the earlier deprecation framing.
- **New `reasoning.context` {auto,current_turn,all_turns}:** response echoes the effective mode;
  `all_turns` replay mechanics marked `[unverified]`.
- **New assistant-message `phase` field** {commentary,final_answer}, assistant-only ("Don't add
  `phase` to user messages"), confirmed for `gpt-5.4`/`gpt-5.5` only (codex / "subsequent"
  unverified); must be preserved on manual history replay.
- **Programmatic Tool Calling** (hosted V8 runtime, `allowed_callers` gate, `program`/
  `program_output` items, ZDR note) and **explicit prompt-caching controls** authored as full
  subsections: 1.25x cache-write fee on 5.6+, required `prompt_cache_key`,
  `prompt_cache_options.mode`/`.ttl` (`30m`), `prompt_cache_breakpoint`, new `cache_write_tokens`
  usage field, `prompt_cache_retention` deprecated for 5.6+.
- **Multi-agent orchestration:** the "no hosted multi-agent handoff API endpoint" absolute
  retracted — a hosted Responses-API beta exists (`responses_multi_agent=v1`, all GPT-5.6
  models), with the six collaboration actions, three new item types, transport options, and four
  documented limitations.
- **Deep Research:** o3-deep-research/o4-mini-deep-research/gpt-5.2-codex shutdown pinned to
  2026-07-23, disambiguated from the plain `o3-2025-04-16` snapshot's 2026-12-11 date; successor
  invocation `return_token_budget` on the Responses web-search tool (changelog-only) added with
  the guide-lag note retained.
- **Deprecations:** added the 2026-12-11 snapshot wave, the completed 2026-05-12 dall-e-2/
  dall-e-3/Realtime-Beta removal, and the June-2026 wave (Agent Builder, legacy Evals platform,
  reusable prompts — all shutdown 2026-11-30, migration targets quoted). Assistants 2026-08-26
  confirmed and kept.
- Cross-reference added in both files to the new `gpt-oss-prompt*.md` open-weight coverage.

### Gemini — Interactions-API-primary restructure (`gemini-prompt.md`, `gemini-prompt-api.md`, `deep-research-agents.md`)

- **Interactions API added as the PRIMARY Gemini surface** (`gemini-prompt-api.md`): GA
  2026-06-22, public beta since December 2025, vendor-recommended for all new projects.
  `generateContent` relabeled legacy-but-fully-supported (still receiving new mainline models).
  Every section now leads with the snake_case Interactions shape and retains the camelCase legacy
  shape clearly labeled. Documented: endpoint `POST /v1beta/interactions` (v1 stable exists
  alongside v1beta); `input`/`Step` envelope (step types `user_input`/`model_output`/
  `function_call`/`function_result`/`thought`); `function_call.id` <-> `function_result.call_id`
  pairing; terminal read `interaction.steps[-1].content[0].text`; `generation_config.thinking_level`
  (lowercase minimal|low|medium|high); `tool_choice` UNDER `generation_config`
  (auto|any|none|validated — the top-level shape is the out-of-scope Vertex enterprise product);
  flat `{type:"function", name, description, parameters}` tool declaration plus
  code_execution/google_search/url_context/mcp_server types; `response_format` with the four
  Text/Audio/Image/Video sub-shapes (Tier-1, quoted field-for-field); `previous_interaction_id`
  server-side state (store default true, retention 55 days paid / 1 day free); streaming SSE
  events with stream resume via `last_event_id`/`event_id`; thought-signature auto-handling;
  SDK floors google-genai >= 2.3.0 / @google/genai >= 2.3.0; and the not-yet-supported list.
  Legacy lowercase `thinkingLevel` enums left unchanged.
- **Surface-selection note added** to `gemini-prompt.md` (Interactions default for new work;
  generateContent legacy-but-supported; prompt craft portable across both).
- **`deep-research-agents.md` (Gemini section) reconciled to GA docs:** terminal schema corrected
  from a flat `outputs` list to the vendor-documented `steps` array (promoted to Tier-1);
  documented usage list trimmed to the 6-key core schema; `agent_config` promoted to Tier-1
  (type deep-research required; thinking_summaries auto/none default none; visualization auto/off
  default auto; collaborative_planning bool default false); SDK floors -> >= 2.3.0; retention
  promoted to the Tier-1 55-day/1-day window; `deep-research-pro-preview-12-2025` re-dispositioned
  (dropped from the overview supported-models table but still in the API-reference enum/example,
  treated as stale/legacy but likely still resolving). OpenAI deep-research shutdown pinned to
  2026-07-23 (the earlier Dec-11 reading conflated it with the base o3-2025-04-16 snapshot, which
  retires 2026-12-11). Field observations re-tiered per fact-sheet §5: silent-400-zombie (N=2),
  steps-hidden-until-terminal (N=2), GET-time usage-schema equivalence (N=2), and 3.6-8.9-min
  latency (N=11) stay `[field-observed]`; the two withdrawn observations stay withdrawn.

## prompt-engineering-skill 0.4.1 — 2026-07-19

Manual browser-verification fold-in. Thirteen bot-blocked/JS-rendered primaries were
verified by the maintainer in a signed-in browser (exact quotes, retrieved 2026-07-19;
evidence retained in the round-3 audit records) and folded into the skill. `SKILL.md`
bumped to `0.4.1`; contract-version unchanged (2026-04-18).

### Claude (`claude-prompt.md`, `claude-prompt-api.md`)

- **Claude Sonnet 5 added** (GA 2026-06-30, drop-in Sonnet 4.6 successor): IDs incl.
  Bedrock/Google Cloud forms (dateless ID = pinned snapshot), 1M context default-and-max
  with no beta header, 128K sync / 300K batch output (`output-300k-2026-03-24`), intro
  $2/$10 per MTok through 2026-08-31 then $3/$15, adaptive thinking on by default with
  `{type:"disabled"}` supported (manual `budget_tokens` 400), effort default `high`,
  sampling params 400, ~30% larger tokenizer at unchanged per-token price, no Priority
  Tier, real-time cybersecurity safeguards (HTTP 200 `stop_reason:"refusal"`).
- **`whats-new-claude-4-7` citations re-anchored** (~13; the URL now 301s to
  `whats-new-claude-4-8`): thinking-display default re-pointed to the Adaptive thinking
  page (default `display` "omitted" on Fable 5 / Mythos 5 / Sonnet 5 / Opus 4.8 / 4.7 /
  Mythos Preview — silent change from Opus 4.6 "summarized"); literalism,
  response-length-calibration, tokenizer-1.0-1.35x, and other 4.7-only claims kept scoped
  to Opus 4.7 with explicit historical / no-longer-vendor-documented markers.
- Effort `max`/`xhigh` availability pinned per model (xhigh not on Mythos Preview /
  Opus 4.6 / Sonnet 4.6; no beta header); full per-model prompt-cache minimum table
  (Sonnet 5 = 1,024 incl. Bedrock) and 12-row tool-use overhead table promoted to Tier-1.
- Last-turn prefill rejection pinned: 400 `invalid_request_error` with exact message;
  closes the prefill gaps; disable-thinking on Fable 5 / Mythos 5 / Mythos Preview stated
  as rejected/not-supported without asserting a status code (only the manual-enabled 400
  is vendor-pinned). New testable markers for prefill and Sonnet 5 sampling/thinking.
- **Vertex AI structured outputs corrected to Preview (Pre-GA), not GA**: supported =
  Opus 4.7 / Sonnet 4.6 / Opus 4.6; coming soon = Opus 4.5 / Sonnet 4.5 / Haiku 4.5;
  Fable 5 / Sonnet 5 / Opus 4.8 / Mythos Preview absent; `output_config.format` +
  `tools[].strict` mechanics and the org-policy gate documented.
- Mythos Preview retirement 2026-07-21 re-confirmed on the deprecations page; the
  competing 2026-06-30 date found no live Anthropic surface and no [disputed] tag ships.

### Qwen (`qwen-prompt.md`, `qwen-prompt-api.md`)

- 1M-vs-256K context figures resolved as two serving modes, not a conflict: standard
  pay-as-you-go API caps input at 1M (256K is a qwen3.7-plus pricing-tier boundary);
  dedicated deployment (Provisioned Throughput / Model Unit) caps the -2026-05 snapshots
  at 256K (128K for qwen3.6-plus / qwen3.5-plus snapshots) with documented automatic
  fallback to pay-as-you-go.
- Per-tier Model Studio pricing added (qwen3.7-max $2.5/$7.5 0-1M; qwen3.7-plus $0.4/$1.6
  and $1.2/$4.8; qwen3.6-plus $0.5/$3 and $2/$6; limited-time promos dated) plus cache
  rates (explicit creation 125% of standard input, hits 10%).
- qwen3.6-plus confirmed NOT scheduled for retirement (appears on the deprecation ledger
  only as a migrate-to reference; current batch dated 2026-10-10).

### Llama (`llama-prompt.md`)

- Closed the quote-level gap on the Llama 4 current-generation tag (literal "LATEST",
  captured from the live DOM); citation updated to the working `/ai/models/llama-4/` path
  with a client-side-rendering caveat.

## prompt-engineering-skill 0.4.0 — 2026-07-19

Round-3 refinement pass. `SKILL.md` bumped to `0.4.0`; contract-version
unchanged (2026-04-18). Tier-1 point facts re-verified against live vendor docs
on 2026-07-18 (firecrawl scrapes of platform.claude.com, ai.google.dev, and
developers.openai.com). Field-observed additions are abstracted public-model
behavior per repo Directive 2, carry `[field-observed]` markers with N=1-2 (N=15
for the grammar-decoding note), and state ranges not fixed numbers. Deferred
items (generational briefs F1/F2/F3/F11/F12/F30/F31/F32 and test-gated F10) are
NOT in this draft.

### Claude (`claude-prompt.md`, `claude-prompt-api.md`) — Tier-1

- **Prompt-cache minimums corrected** (F25): Opus 4.7 4096→2048, Sonnet 4.6
  2048→1024; added Opus 4.8=1024, Sonnet 5=1024, Fable 5 / Mythos 5=512 (1024 on
  Bedrock). Closed the §10 gap on the Fable 5 first-party minimum.
- **`frontier_llm` refusal category** (F26): the invisible-safeguards prose is
  replaced by the now-documented visible refusal category (cyber / bio /
  frontier_llm / reasoning_extraction), all fallback-eligible; updated in both
  files' §6/§9 and the §2 category list.
- **Per-model tool-use overhead table** (F27): the flat 346/313 figure (matching
  no current model) replaced with the per-model table (Opus 4.8 290/410, Opus 4.7
  675/804, Sonnet 5 354/474, etc.).
- **Deprecation/tense fixes** (F28): Opus 4 / Sonnet 4 retired 2026-06-15
  (completed); Opus 4.1 now Deprecated (retires 2026-08-05); Mythos Preview
  retires 2026-07-21; web_search now `web_search_20260318` (response_inclusion);
  Fable 5 `max` effort confirmed available; Vertex AI structured-outputs now GA
  including Mythos Preview (prior "Vertex does not support Mythos Preview" claim
  corrected).
- **Anti-pattern / placeholder** (F18a, F19): CAPS-emphasis anti-pattern
  generalized beyond the single tool-command string (behavioral breadth
  test-gated with a `[testable]` marker); §2 gains a fill-in-placeholder
  convention note (`{{value}}` / `[VALUE]` / backtick-wrap over bare
  `<angle-bracket>`).

### Gemini (`gemini-prompt.md`, `gemini-prompt-api.md`) — Tier-1

- **Implicit-caching floors corrected** (F29): 3.5 Flash 1024→4096, 2.5 Flash
  1024→2048, 2.5 Pro 4096→2048 (3.1 Pro unchanged at 4096); fixed both the API
  table and the prose "Flash=1024/Pro=4096" split.
- **gemini-3.5-flash context window** published (F34): 1,048,576 in / 65,536 out
  (partially closes the declared gap).
- **gemini-3.1-flash-lite shutdown** 2027-05-07 added (F28g).
- **Deep Research cross-reference** (F6): `gemini-prompt.md` gains a short DR
  subsection (enumeration-bucket fix + pointer to `deep-research-agents.md`).

### Deep Research (`deep-research-agents.md`) — field-observed (Gemini DR)

New "Field-observed behavior (Gemini Deep Research)" subsection plus inline
refinements: dual citation schema with `grounding-api-redirect` wrapper URLs
(F4); structural/suppression-honored vs inline-markup-ignored compliance split
(F5); enumeration bucket fix (F6); non-terminal failure modes — silent
400-zombie plus steps-hidden-until-terminal; the earlier bare-domain-literal
wedging correlation was refuted by controlled matched-pair testing 2026-07-19
and is withdrawn (F7, revised); observed usage-schema divergence, shown NOT
Api-Revision-dependent (F13, revised); realized Max cost ~$40 (F14); ~5-job
concurrency cap (F15); terminal `outputs` schema (F20); narrow source pool (F21);
adjacent-entity substitution + unreachable-source padding (F22); latency
re-measured 3.6-8.9 min narrow/standard N=11, prior 39-50 min attributed to
observation lag + scope/tier (F23, revised); integration facts (F24);
seed-suppresses-fabrication (F35).
OpenAI o*-deep-research shutdown tightened to imminent (F33, 5 days out); the
Anthropic web-search version refreshed to `web_search_20260318`.

### Qwen / OpenAI-compat (`qwen-prompt-api.md`, `openai-compatibility-surface.md`)

- **Grammar-constrained-decoding traps** (F8): §6 gains a `[field-observed]`
  (N=15) note — a grammar constrains emitted, not read, tokens (enum values must
  also appear in the prompt); a closed enum with no fallback member cannot fail
  safe (~10/15 confident near-miss). Recommends an explicit fallback member plus
  an open free-text companion field; carries a `[testable]` marker.
  Cross-referenced from the openai-compat vLLM/llama.cpp rows.

### Routing (`SKILL.md`)

- Same-family-consumer tie-breaker (F9); agent-orchestration trigger scoped to
  hosted vendor endpoints with a local/CLI-runner exclusion (F16); reuse clause
  extended to re-check `[applies-to]` version-scope, not just `retrieved:`
  freshness (F17); coverage-table Claude row refreshed for the deprecation/tense
  and refusal-category changes.

### Family expansion — 2026-07-19 (three new families, two updated, one disambiguated)

Six family lanes landed against live vendor docs retrieved 2026-07-19. Three new
families join the skill; two existing families take flagship/lineup updates; one
existing family gains a scope-disambiguation note. `SKILL.md` stays at
`0.4.0`; contract-version unchanged (2026-04-18). New provider sections in
`openai-compatibility-surface.md` carry their own 2026-07-19 sources, so that
file's top-level `retrieved:` (the 2026-06-01 sweep) is unchanged.

**New — GLM (Z.ai / zai-org).** New `resources/glm-prompt.md` and
`resources/glm-prompt-api.md` covering GLM-5.2 (753B MoE, IndexShare sparse
attention, 1M context, MIT open weights) plus GLM-5.1 / GLM-5-Turbo / GLM-4.7.
Documents the seven-value `reasoning_effort` ladder and its documented shim onto
two thinking tiers (low/medium→high, xhigh→max, none/minimal→skip), the
`thinking.type` toggle, dynamic-vs-forced-thinking split, XML tool-call wire
format, the dual `stream`+`tool_stream` streaming requirement, three-stream
deltas, the `clear_thinking` multi-turn contract with its endpoint-dependent
default, automatic prefix caching, Tier-1 pricing, and the Anthropic-compatible
endpoint (`api.z.ai/api/anthropic`) as a Claude Code drop-in. GLM-5.2 max output
carried as a documented doc-surface disagreement (docs.z.ai 128K vs HF card
163,840). Parallel tool calls and a cache-block minimum recorded as confirmed
documentation absences; the `glm-5.2[1m]` alias and specific
`ANTHROPIC_DEFAULT_OPUS_MODEL` / `CLAUDE_CODE_AUTO_COMPACT_WINDOW` values declared
as gaps pending canonical vendor confirmation.

**New — MiniMax.** New `resources/minimax-prompt.md` and
`resources/minimax-prompt-api.md` covering MiniMax-M3 (1M context, ~428B-total /
~23B-active MoE with Multi-Scale Attention, multimodal), the M2.7 / M2.5 / M2.1 /
M2 line (204,800 context, text + tool-call), and the `M2-her` chat model (64K). M3
open weights are published on HuggingFace under the MiniMax Community License
(non-commercial by default) — the marketing page's "coming soon" copy is stale.
API guidance centers on the vendor-recommended Anthropic-compatible
`/anthropic/v1/messages` path, with the OpenAI-compatible and legacy native
`chatcompletion_v2` surfaces documented secondarily. Covers thinking off-by-default
on M3 vs un-disable-able on M2.x, the multi-turn reasoning-preservation contract,
MSA 1M context with the 512K pricing-tier / guaranteed-floor semantics, and
automatic (passive) caching on M3 vs explicit `cache_control` on M2.x.

**New — Kimi (Moonshot AI).** New `resources/kimi-prompt.md` and
`resources/kimi-prompt-api.md` covering Kimi K3 (`kimi-k3`), three days post-launch:
2.8T-param KDA architecture, 1M context, always-on reasoning via `reasoning_effort`
(only `max` accepted today, "more levels coming soon"), five fixed sampling
parameters that error on deviation, the multi-turn complete-assistant-message
replay contract, `response_format` JSON Schema (MFJS) structured output, partial
(prefix-continuation) mode, K3-only dynamic tool loading (content-less `system`
message), the official tool set, and deprecations vs K2.x (K2.5 / moonshot-v1
sunset 2026-08-31, k2 previews discontinued 2026-05-25). OpenAI-compatible
(`api.moonshot.ai/v1`) and Anthropic-compatible (`api.moonshot.ai/anthropic`,
`kimi-k3[1m]`) surfaces; Kimi Code (`api.kimi.com/coding`) kept distinct as a
separate product. Open-weights sections (chat template, special tokens, license,
inference-stack flags) declared as gaps — weights announced for release by
2026-07-27, not yet published.

**Updated — Gemma.** Added the fifth Gemma 4 size, **12B Unified**
(`google/gemma-4-12B` / `-it`): dense, encoder-free multimodal, 11.95B total, 256K
context, added 2026-06-03. Now in the front-matter `versions:` list, the
model-selection table, and every size enumeration in both `gemma-prompt.md` and
`gemma-prompt-api.md`. Corrected the family-wide "audio is E-series only" claim:
audio-capable sizes are now E2B, E4B, **and 12B Unified**; 26B-A4B and 31B remain
text/image/video only. Added the encoder-free multimodal ingestion pipeline
(Tier-2, HF Transformers docs), tightened the empty-thinking-token stabilizer claim
to the exact instruction-tuned IDs, and added the `skip_special_tokens=False`
decode mandate (Tier-1). Resolved the exact-release-date gap (original four sizes
2026-03-31; 12B Unified 2026-06-03); declared new gaps (12B Unified decoder layer
count / vocab size; whether it ships a dedicated speculative-decoding draft model).
`retrieved:` bumped to 2026-07-19.

**Updated — Grok.** grok-4.5 (500K context) supersedes grok-4.3 as flagship
(grok-4.3 demoted to prior-gen, still live). Reasoning defaults inverted and split
by model: grok-4.5 `reasoning_effort` ∈ {low, medium, high}, default `high`, `none`
removed (reasoning cannot be disabled); grok-4.3 retains {none, low, medium, high},
default `low`. Every reasoning statement scoped per model. Flagged the
counterintuitive context regression (grok-4.5 500K < grok-4.3 1M). Scoped the
`grok.reasoning-effort-none-disables.v1` testable to grok-4.3 and added
`grok.reasoning-effort-none-rejected-45.v1` (grok-4.5 rejects
`reasoning_effort:"none"`). Added bifurcated pricing and the 200K repricing cliff,
the `prompt_cache_key` / `x-grok-conv-id` cache-pin split, corrected 2026-05-15
retirement redirect targets, and recorded the grok-4.5 knowledge-cutoff conflict
(models page Feb 1 2026 vs system card Jan 2026). SKILL.md Grok row + reasoning-
portability line and the `openai-compatibility-surface.md` Grok section updated.
`retrieved:` bumped to 2026-07-19.

**Disambiguated — Llama.** Added a Muse Spark scope note to `llama-prompt.md` and
`llama-prompt-api.md`. Muse Spark (Muse Spark 1.1, Meta Model API public preview
2026-07-09) is Meta Superintelligence Labs' current closed-weight, API-only
flagship — a separate brand/distribution track from the open-weight Llama line — and
is explicitly out of scope for the Llama files. No Muse family file created. No
Llama 4 factual changes: Scout / Maverick lineup, specs, license, and architecture
re-confirmed live 2026-07-19; `retrieved:` bumped to 2026-07-19 on both files.

**Shared-file integration.** `SKILL.md` gained coverage rows for GLM, Kimi, and
MiniMax; the Gemma / Grok / Llama rows and the reasoning-controls portability line
were updated to match the lane changes; the description-line family enumeration
gained GLM, MiniMax, Kimi. `openai-compatibility-surface.md` gained MiniMax, GLM,
and Kimi provider sections plus quick-matrix rows, and its Grok reasoning-control
line was split by generation.

## prompt-engineering-skill 0.3.0 — 2026-06-10

Claude Fable 5 launch coverage (`retrieved: 2026-06-10` on the two Claude reference
files; other families untouched). Contract-version unchanged (2026-04-18).

### Claude — new flagship: Claude Fable 5 (`claude-fable-5`, GA 2026-06-09)

- **`claude-prompt.md`** — Fable 5 added as the model-selection top row ($10/$50 per
  MTok, 1M ctx, 128K output, knowledge cutoff Jan 2026); Mythos 5 noted as the
  same-weights limited-release variant (Project Glasswing, successor to Mythos
  Preview). New instruction patterns from Anthropic's Fable prompting guide: brief
  steering instructions over per-behavior enumeration, give-the-reason framing,
  anti-overplanning, and tool-result-grounded progress claims. Behavioral-quirks
  block covers longer turns by default, effort guidance, over-elaboration at higher
  effort, unrequested actions, parallel-subagent readiness, memory-system uplift,
  rare early stopping, context-budget anxiety on visible token countdowns, long-run
  readability degradation, and the send-to-user-tool pattern. System-card failure
  clusters (886-use sample) and the strategic-deference finding included with
  provenance. New anti-patterns: no reasoning-echo instructions
  (`reasoning_extraction` refusals), no token countdowns, no wholesale carry-over of
  prior-model skills, no simple-workload-only evaluation.
- **`claude-prompt-api.md`** — Fable 5 platform IDs (Claude API, Bedrock incl.
  geo/global inference IDs and `bedrock-mantle`; Vertex/Foundry available but IDs
  unpinned); Covered-Model data-retention constraint (30-day, no ZDR; Bedrock
  `provider_data_share` opt-in). Thinking semantics: adaptive always on, `disabled`
  unsupported, `display` defaults `"omitted"`, raw chain of thought never returned.
  Sampling bounds per the Bedrock model card. New §2 subsection documenting the
  refusal response shape (`stop_reason: "refusal"`, `stop_details` categories
  cyber/bio/reasoning_extraction), billing, server-side `fallbacks` parameter (beta
  `server-side-fallback-2026-06-01`), fallback content blocks and `usage.iterations`,
  echo rules, streaming semantics, SDK refusal-fallback middleware, and fallback
  credit (`fallback-credit-2026-06-01`). Beta-header table and breaking-changes
  section extended; new testable IDs for thinking-disabled rejection, omitted-display
  default, and sampling rejection. Gaps declared for prefill handling, first-party
  cache minimum, Vertex/Foundry IDs, `allowed_fallback_models`, sticky-routing/billing
  subsections, and `effort: "max"` availability.
- **`SKILL.md`** — Claude coverage row and cross-family thinking-controls line
  updated for Fable 5; version bumped to 0.3.0.

## prompt-engineering-skill 0.2.0 — 2026-06-01

Family-lineup refresh against live primary sources (`retrieved: 2026-06-01`) plus a
scope expansion from "how to prompt model X" to "how to prompt, configure, and deploy
against model X's surfaces." Contract-version unchanged (2026-04-18).

### Urgent retirement-cliff corrections

- **Claude** — `claude-opus-4-20250514` / `claude-sonnet-4-20250514` retire **2026-06-15**;
  the Opus 4 replacement is corrected to `claude-opus-4-8`. `claude-3-haiku-20240307` is
  already Retired (April 20, 2026).
- **DeepSeek** — legacy `deepseek-chat` / `deepseek-reasoner` hard-retire **2026-07-24**
  (now map to `deepseek-v4-flash`); V3.2-Speciale's temporary API endpoint expired
  **2025-12-15** (weights remain).
- **Gemini** — 2.0 Flash / Flash-Lite GA shut down **2026-06-01**; 2.5 GA sunsets
  **2026-10-16**.
- **Grok** — the prior fast / 4.x / 3 slugs were retired **2026-05-15** and redirect to
  `grok-4.3` (or `grok-build-0.1` for `grok-code-fast-1`).

### Flagship and lineup moves

- **Claude** — added Opus 4.8 (`claude-opus-4-8`, GA 2026-05-28, 1M ctx, cutoff Jan 2026)
  as flagship; Opus 4.7 still Active; adaptive thinking is the only mode on 4.7/4.8 (manual
  budgets rejected). Sonnet 4.6 knowledge cutoff flagged `[disputed]` (Aug 2025 vs May 2025
  across two live Anthropic surfaces).
- **OpenAI** — GPT-5.5 (`gpt-5.5-2026-04-23`) flagship + 5.5-pro; GPT-5.4 retained as the
  cheaper tier; top Codex `gpt-5.3-codex`. Assistants API removed 2026-08-26, migration target
  corrected to **Responses + Conversations**; o\*-deep-research IDs shut down 2026-07-23;
  multi-agent handoffs documented as Agents-SDK-only.
- **Gemini** — 3.5 Flash GA + 3.1 Flash-Lite GA + 3.1 Pro Preview; `thinkingLevel` replaces
  `thinkingBudget` on Gemini 3; Computer Use isolated to `gemini-2.5-computer-use-preview-10-2025`.
- **DeepSeek** — V4 production (`deepseek-v4-flash` / `deepseek-v4-pro`, MIT, 1M ctx, 384K out);
  tri-state reasoning (Non-think / Think / Think Max, the last recommending ≥384K context);
  reasoning_content round-trip is now conditional on tool calls.
- **Qwen** — Qwen3.7-Max GA flagship; Qwen3.6-Plus demoted-but-GA; added dense open-weights
  `Qwen3.6-27B` alongside the `qwen3.6-35b-a3b` MoE.
- **Mistral** — Mistral Medium 3.5 (Modified MIT, 256K) frontier + Small 4; unified
  `reasoning_effort` is binary (`high`|`none`) on the chat endpoint; Magistral → Legacy.
- **Gemma** — Gemma 4 lineup re-verified (128K/256K context split; per-size dedicated draft model).
- **Llama** — re-confirmed; no Llama 5; Scout + Maverick current.

### New cross-family resources

- `resources/deep-research-agents.md` — hosted deep-research agents (Gemini, OpenAI,
  Perplexity async submit/poll) vs web-search tool-use loops (Anthropic, Grok).
- `resources/openai-compatibility-surface.md` — single-source matrix of per-provider
  divergences through OpenAI-shaped endpoints (Grok, DeepSeek, Gemini, Qwen, Mistral, vLLM,
  llama.cpp). Family `-api.md` files cross-reference it.
- `resources/agent-orchestration-surfaces.md` — Anthropic Managed Agents (300/600 limits),
  OpenAI hosted/SDK split, the three disaggregated Gemini agent products.
- `resources/webui-surfaces-and-silent-degradation.md` — tier-labeled; leads with the honest
  gap (consumer default reasoning effort undocumented). Detection methodology routed to
  `prompt-engineering-architect`.

### SKILL.md routing and discipline

- Empty-arg routing (#5), generated-prompt subject-vs-consumer routing (#7), additive
  mixed-task scope (#6), a third "surface" routing axis (#9), reuse-from-context freshness
  check (#8), a strengthened declared-Gaps stance (#10), and reading-discipline notes —
  weight measured behavior over benchmark priors; test a newer variant before switching (#16).
- Coverage table rewritten; cross-family resource rows added. SCHEMA.md gained a cross-family
  resource-file convention. Corrected the Cross-Family Portability section (the Grok
  "reasoning_effort = agent count" claim was false; DeepSeek's reasoning round-trip is conditional).

### Known limitations at 0.2.0

- Several family-file sections not touched by this refresh retain their 2026-04-18/19 inline
  provenance (re-verification covered lineup, reasoning, deprecation, and the new surfaces).
- Family knowledge cutoffs for current Qwen and all DeepSeek models remain undocumented (Gaps).
- A few REINSTATED candidate facts failed their author-time canonical confirm and were NOT
  shipped: a 65,536 output cap for Gemini Deep Research (that figure belongs to the Antigravity
  Agent) and xAI "DeeperSearch" (Tier-3 only).

## prompt-engineering-skill 0.1.0 — 2026-04-19

### Added

Initial public release. Family-specific prompt-engineering references for nine current-generation LLM families — Claude, OpenAI (GPT-5.x / o-series), Gemini, Gemma, Llama, Qwen, Grok, Mistral, DeepSeek.

- `SKILL.md` — Routing layer with coverage table, reading discipline, and a Cross-Family Portability section enumerating the six most common ways prompts break when ported between families: reasoning-parameter naming differences, reasoning-artifact multi-turn rules (actively contradictory), role naming, "OpenAI-compatible" wire-format vs semantic divergence, open-weights chat-template hand-assembly hazards, dated model-ID pinning.
- `SCHEMA.md` — Normative schema for family reference files: front-matter shape, section order, source tiering (Tier 1 / 2 / 3), inline markers (`[applies-to]`, `[testable]`, `[unverified]`, `[disputed]`), required workflow.
- `README.md` — Human-facing framing, audience, source policy summary, known limitations.
- `resources/` — Nine pairs of reference files (prompt-layer + API-layer per family) with inline provenance per claim, a `retrieved:` date, declared version scope, and an honest `Gaps` section.

Contract-version pinned to 2026-04-18.

### Known limitations at 0.1.0

- Test harness not yet built; `[testable: ...]` claims are tagged but not asserted.
- Depth varies by family based on provider documentation density — families with denser public docs (Anthropic, OpenAI, Qwen) support deeper references than those with thinner coverage (Grok, Mistral function-calling).
- Point-in-time snapshot. Every file has a `retrieved:` date; content older than 90 days without re-verification should be treated as stale per repository contract.
- Gaps declared per file rather than silently omitted. Files under-documented by their providers carry larger Gaps sections.

## domain-research-skill 0.1.0 — 2026-04-19

### Added

Initial public release. A routing skill that layers domain-specific source taxonomies on top of a fixed, domain-neutral `Research_Protocol`. Thirteen domains covered at 0.1.0:

1. Technical research (AI/ML, infrastructure, software engineering)
2. GovCon proposal writing — US federal solicitation response
3. Federal capture & contract vehicle research — pre-solicitation
4. Small business set-aside & certification — HUBZone / 8(a) / WOSB / SDVOSB / SDB
5. State & local government procurement — state portals and cooperative vehicles
6. Nonprofit sector research — IRS 990s, foundations, grant databases
7. Salesforce integration architecture — release-versioned platform work
8. Healthcare data compliance — HIPAA / HITECH / state variants
9. Accessibility & Section 508 conformance — WCAG 2.2 / ISO/IEC 40500:2025, VPATs, DOJ Title II IFR (2026)
10. Web security & WAF configuration — CVE / NVD / OWASP Top 10 2025 / CISA KEV / CRS
11. AI/ML model & hardware evaluation — reproducible benchmarks (MLPerf Inference v6.0 etc.), not marketing
12. Japanese legal & regulatory — e-Gov法令検索, 判例, 官報 (electronic-primary), bilingual handling
13. Federal contractor cybersecurity — CMMC (32 CFR Part 170), NIST SP 800-171, DFARS cyber clauses

Files:

- `SKILL.md` — Routing layer with the locked `Research_Protocol` inline, activation logic, routing table by domain, multi-domain overlap handling (federal cluster), output protocol, deterministic-retreat procedure, coverage status, tuning rubric for authoring new domains.
- `README.md` — Human-facing framing, rationale for splitting invariant protocol from variable taxonomy, known limitations.
- `resources/` — One file per domain with a populated `<Source_Domain>` block (`Tier_1_Primary`, `Tier_2_Contextual`, `Disqualified`, `Evidence_Threshold`) naming canonical databases, registries, and noise patterns specific to that domain.

Several high-churn domains carry a **Validator Note — Post-Training-Cutoff Authority Changes** subsection documenting recent authoritative changes with primary-source citations so that audit passes do not mistake post-cutoff changes for fabrications. Covered changes include EO 14347 "Restoring the United States Department of War," the DFARS Case 2019-D041 Final Rule, FPDS.gov and eSRS.gov decommissionings, the DOJ ADA Title II Interim Final Rule extending compliance dates, elimination of SDVOSB / VOSB self-certification under NDAA 2024, the Japanese Kanpo electronic-publication transition, and the APPI 2025/2026 reform cycle.

Contract-version pinned to 2026-04-18.

### Known limitations at 0.1.0

- US-centric coverage for federal and state-procurement domains. Non-US public procurement is not covered.
- Healthcare domain is compliance-oriented; clinical / evidence-based-medicine research is out of scope.
- AI/ML evaluation domain describes how to accept or reject benchmark claims; it does not produce rankings or buying guides.
- Multi-domain overlap is handled by the union of taxonomies rather than precomputed overlap patterns.
- **Automated primary-source verification is partial.** Several primary sources this skill cites (`.gov` properties, `ecfr.gov`, `federalregister.gov` HTML pages, `salesforce.com`, some `iso.org` pages) sit behind Cloudflare / bot protection and return 403 to automated fetchers. Where possible the resource files cite a structured-data alternative (e.g., the Federal Register JSON API at `federalregister.gov/api/v1/documents/{doc-number}.json`); where none exists, manual-browser verification is the fallback. See repo-root `TODO.md` for the validation-workflow backlog.
- Point-in-time snapshot. Files older than 90 days without re-verification should be treated as stale. Authoritative-source lists change on regulatory, legislative, and programmatic cycles; authorities active at 0.1.0 include the DFARS Case 2019-D041 Final Rule (effective 2025-11-10), the CMMC Program Final Rule (effective 2024-12-16), NIST SP 800-171 Rev. 3 (2024-05-14), NIST SP 800-66 Rev. 2 (2024-02), and the DOJ ADA Title II IFR (effective 2026-04-20).
