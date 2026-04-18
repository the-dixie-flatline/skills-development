# Domain Research Skill

A Claude skill that layers domain-specific source taxonomies on top of a fixed, domain-neutral Research_Protocol. The invariant decides how claims get grounded. The domain layer decides what counts as a primary source in a given practitioner domain, and what noise looks like there.

## What This Is

A routing layer + thirteen domain taxonomies. When Claude takes on a research task that implicates one of the covered domains, the skill injects that domain's `<Source_Domain>` block alongside the locked Research_Protocol. When no domain matches, the protocol alone governs.

Covered domains:

1. Technical research — AI/ML, infrastructure, software engineering
2. GovCon proposal writing — US federal solicitation response
3. Federal capture & contract vehicle research — pre-solicitation
4. Small business set-aside & certification — HUBZone / 8(a) / WOSB / SDVOSB / SDB
5. State & local government procurement — state portals, cooperative vehicles
6. Nonprofit sector research — 990s, foundations, grant databases
7. Salesforce integration architecture — release-versioned platform work
8. Healthcare data compliance — HIPAA / HITECH / state variants, compliance-oriented
9. Accessibility & Section 508 conformance — WCAG 2.x, VPATs, Revised 508
10. Web security & WAF configuration — CVEs, OWASP, CISA KEV, vendor advisories
11. AI/ML model & hardware evaluation — reproducible benchmarks, not marketing
12. Japanese legal & regulatory — e-Gov法令検索, 判例, bilingual handling
13. Federal contractor cybersecurity (CMMC) — 32 CFR Part 170, NIST SP 800-171, DFARS cyber clauses

## Why A Fixed Protocol Plus A Variable Taxonomy

Research work fails in two distinct ways:

- **Protocol failure.** The researcher accepts paraphrase, undated summaries, or AI-generated content as substantiation. This is domain-independent; the fix is a locked methodology that refuses to substitute contextual sources for primaries.
- **Taxonomy failure.** The researcher uses the right methodology but accepts the wrong things as primaries. A "reputable industry analyst report" substantiates nothing about a federal solicitation; an arxiv preprint does not settle a HIPAA compliance question; a law firm client alert is not a statute.

Splitting the invariant from the taxonomy lets Claude carry the methodology across every session while loading only the taxonomy the current task implicates.

## Files

- `SKILL.md` — Routing layer with the locked Research_Protocol, activation logic, routing table, deterministic-retreat procedure, and multi-domain overlap handling.
- `README.md` — This file.
- `resources/` — One file per domain, each containing a populated `<Source_Domain>` block.

## How To Use

**As a human reader.** Find the resource file for the domain you work in. The `<Source_Domain>` block names what primaries you must cite, what sources merely frame context, what sources to reject outright, and what minimum evidence a claim needs before it ships.

**As a Claude skill consumer.** The skill routes on domain triggers (see the Routing Table in `SKILL.md`), loads the matching resource(s), and treats the injected block as the floor for substantiation. The Research_Protocol itself is non-negotiable; domain resources only tell it what to apply to.

## The Tuning Rubric (For New Domains)

A new domain entry answers five questions with specifics:

1. What is the authoritative artifact in this domain? (Tier_1_Primary)
2. What sources frame but cannot substantiate? (Tier_2_Contextual)
3. What does noise look like specifically here? (Disqualified)
4. What is the minimum citation type required before a claim ships? (Evidence_Threshold)
5. What revision or currency requirement applies? (embedded in Evidence_Threshold)

Generic entries fail the skill's purpose. A rubric answer of "peer-reviewed sources" is useless; "arXiv preprint with the associated venue acceptance, or the venue proceedings directly, cited by DOI" is actionable.

## Known Limitations

- **Point-in-time snapshot.** Each resource file has a `retrieved:` date. Authoritative-source lists change (FPDS-NG consolidated into SAM.gov, SDVOSB certification moved from VA CVE to SBA in 2023, HIPAA-relevant NIST SP 800-66 revised in 2024). Files older than 90 days without re-verification should be treated as stale.
- **US-centric coverage for federal / state-procurement domains.** Non-US public procurement is not covered at 0.1.0.
- **Compliance orientation for the healthcare domain.** Clinical guidance (evidence-based medicine, clinical-trial registries) is explicitly out of scope; the domain covers HIPAA-class data compliance.
- **No model rankings, benchmarks tables, or buying guides.** The AI/ML evaluation domain describes how to accept or reject a benchmark claim; it does not produce one.
- **Overlap is not exhaustively mapped.** Federal work frequently activates three or four domains at once. The skill handles this by loading the union of taxonomies, not by precomputing every overlap pattern.
- **Automated primary-source verification is partial.** Many of the primary sources this skill cites — `.gov` properties, `ecfr.gov`, `federalregister.gov` HTML pages, `salesforce.com`, some `iso.org` pages — sit behind Cloudflare or similar bot protection and return 403 to automated fetchers. An AI-driven validation pass may report such a source as broken even when the URL loads cleanly in a browser. Where possible the resource files cite a structured-data alternative (e.g., the Federal Register JSON API at `federalregister.gov/api/v1/documents/{doc-number}.json`) alongside the HTML URL, but some sources lack a bot-friendly equivalent. When an automated audit reports a 403, **verify the URL manually in a browser before treating it as stale.** See the repo-root [`TODO.md`](../TODO.md) for the validation-workflow backlog.

## Contributing

Contributions welcome. Before submitting a PR:

1. Read the repository-level `CLAUDE.md` and this skill's `SKILL.md`.
2. Confirm every Tier 1 entry names an actual artifact or database, not a category.
3. Confirm every Disqualified entry names a specific noise pattern, not a generic complaint.
4. Confirm no client, employer, or non-public organizational data has leaked into examples. Federal and state-procurement domains carry elevated leak risk; any real PIID, solicitation number, or teaming reference in an example is a failure of the check.
5. Update `retrieved:` where the resource was touched.
6. Declare AI assistance in the PR description if substantial.

Critique-oriented PRs that call out stale canonical-source lists (an agency retires a procurement vehicle, SBA moves certification authority, a state launches a new eProcurement portal) are valued equally with additive PRs.

## License

Inherits from the parent repository. See the repository root for the applicable `LICENSE`.
