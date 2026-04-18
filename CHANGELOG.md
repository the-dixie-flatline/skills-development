# Changelog

All notable changes to this repository are tracked here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The repository holds multiple skills; entries are scoped per skill (and occasionally to repository-level docs).

## Repository infrastructure — 2026-04-19

Initial public-release scaffolding:

- `.github/ISSUE_TEMPLATE/` — structured bug and coverage-request forms, plus a `config.yml` that disables blank issues, redirects security reports to private vulnerability reporting, and redirects general questions to Discussions. Both forms require Tier 1 primary sources; the bug form additionally prompts submitters to manually confirm a URL in a browser before reporting it as broken, since automated fetchers are frequently blocked by Cloudflare on `.gov` and similar primary sources.
- `.github/PULL_REQUEST_TEMPLATE.md` — PR template covering type-of-change, scope, provenance, data-handling self-check, AI-assistance declaration.
- `.gitignore` — editor / IDE state, Claude Code local state, OS metadata, Python and Node artifacts, secret-scanning reports, logs.
- `README.md` — includes a "Source Validation — Known Limitation" section documenting bot-blocker 403s on several federal primary sources and the manual-browser-verification fallback.
- `TODO.md` — open workflow items, principally the source-validation backlog (automated fetch 403s on Cloudflare-protected primary sources; preference for structured-data endpoints like the Federal Register JSON API where HTML surfaces are blocked; open questions on a maintainer-facing verification helper).

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
