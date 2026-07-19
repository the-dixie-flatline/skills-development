# TODO

Rough list of things that need work but don't block anything shipping now. Keep entries small and actionable; prune when done.

## Source validation workflow

**Problem.** Automated primary-source verification — the core discipline of `domain-research-skill` and a quality expectation across the repo — is frequently blocked by Cloudflare / bot protection on `.gov`, Salesforce, ISO/IEC, and other authoritative surfaces. An AI assistant running a validation pass may report a source as broken or unreachable when the URL is, in fact, fine in a human-loaded browser.

**Observed during 2026-04 validation passes** (see `CHANGELOG.md` for specifics):

- `defense.gov`, `dodcio.defense.gov`, `war.gov`, `dowcio.war.gov` — 403 on automated fetch.
- `ecfr.gov` content pages — redirect to `unblock.federalregister.gov` captcha challenge on automated fetch.
- `help.salesforce.com`, `salesforce.com/news/press-releases/`, `salesforce.com/agentforce/` — 403.
- `iso.org/standard/...` — 403 on some pages.
- `federalregister.gov` HTML document pages — redirect to unblock challenge. (The JSON API at `federalregister.gov/api/v1/documents/{doc-number}.json` works and has been a reliable fallback for the ADA IFR, DFARS Final Rule, and EO 14347 verifications.)

**Risk.** Conservative-validator behavior (reject what you can't verify) produces false negatives on load-bearing primary sources. This is exactly the failure mode the `domain-research-skill` is designed to prevent — and in the 2026-04 pass it tripped me into initially rejecting EO 14347 against primary-source-available reality.

**Open questions / work items:**

- Design a validation-pass convention that recognizes "URL exists and resolves for humans but is unreachable by automated fetch" as a distinct state from "URL is stale or broken." Avoid treating 403 as terminal.
- When automated fetch is blocked, prefer structured-data endpoints (JSON APIs, sitemaps, RSS / Atom feeds) where available. The Federal Register JSON API is the cleanest example; identify equivalent fallbacks for other blocked surfaces.
- Investigate `archive.org` (Wayback Machine) as a verification fallback for stable primary sources. The Wayback Machine itself is periodically blocked or slow; not universal.
- Consider a dedicated verification-helper tool (a small CLI using `curl` with explicit browser-like `User-Agent`, or a browser-automation wrapper using Playwright / Chrome DevTools Protocol) for maintainers running offline pre-PR checks. Keep it out of the main skill path — skills should stay assistant-readable — but a maintainer-facing helper would close the gap.
- Documentation: teach reviewers (human and LLM) to treat a 403 from an automated fetcher as "verify manually," not "stale / fake." This is partially in place via Validator Notes in the `domain-research-skill` resources, and a short note in the root `README.md`; extend as needed.

**Contributions welcome.** If you have a clean pattern for cross-domain primary-source verification that handles bot blockers gracefully, open a discussion before writing the PR.

## Route-out to `prompt-engineering-architect` (from the 2026-06-01 refresh)

The prompt-engineering-skill 0.2.0 refresh surfaced several items that are methodology,
not family fact, and belong in a future `prompt-engineering-architect` skill rather than in
the family-specific reference. They were deliberately NOT authored into prompt-engineering-skill.
Captured here so they are not lost:

- **Fabrication-guard / research-discipline block.** Generic, topic-independent guards for
  authoring deep-research-mode prompts (demand primary-source tiering; flag invented runtimes,
  mis-attributed benchmarks, fabricated CLI flags). Proven prompt-correction asset; model-agnostic.
- **Router-portable / model-agnostic pattern set.** ~~The inverse of "pick one family": enumerate
  forbidden family-specific affordances and substitution-robust alternatives for prompts that must
  run across heterogeneous models behind a router.~~ **Partially landed 2026-07-19** in
  `prompt-engineering-skill/resources/portable-baseline.md` (0.8.0): the structural half —
  exclusion rules, portable skeleton, uncovered-family procedure. Still architect-scoped:
  per-target technique-selection methodology and evaluation strategy for multi-family prompts.
  Router-layer status after the same-day third verification pass: mechanism-layer
  normalization is now verified and cited in that file (LiteLLM drop_params/template
  mapping, OpenRouter Response Healing); prompt-content authoring guidance from router
  vendors remains absent.
- **Citation-conflation detection rule.** The *phenomenon* is named in
  `webui-surfaces-and-silent-degradation.md`; the detection methodology (attribution-match, not
  URL-resolves) is author methodology and belongs in the architect skill.
- **Reading-discipline methodology** beyond the two notes already added to SKILL.md (#16): the
  broader "weight operator-measured behavior over benchmark priors" and version-delta-caution
  discipline as a reusable methodology block.

Declined (confirmed out of scope, do not author anywhere): local inference-stack / runtime tuning,
image-generation prompting, NotebookLM.

## Stale `research-skill/` placeholder directory

An empty `research-skill/` directory exists at the repo root (leftover from early planning before the `domain-research-skill/` name was chosen). Decide whether to remove it or repurpose it for a future skill, then clean up.
