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

## Stale `research-skill/` placeholder directory

An empty `research-skill/` directory exists at the repo root (leftover from early planning before the `domain-research-skill/` name was chosen). Decide whether to remove it or repurpose it for a future skill, then clean up.
