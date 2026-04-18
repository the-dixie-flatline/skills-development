---
name: domain-research-skill
description: Domain-specific source taxonomies layered on top of a fixed, domain-neutral Research_Protocol for research work in thirteen practitioner domains — technical/AI research, federal GovCon proposal writing, federal capture and contract-vehicle research, SBA small-business set-aside and certification work, state and local procurement, nonprofit-sector research, Salesforce integration architecture, healthcare data compliance, accessibility / Section 508 conformance, web security and WAF configuration, AI/ML model and hardware evaluation, Japanese legal and regulatory research, federal contractor cybersecurity (CMMC / NIST 800-171 / DFARS cyber). Trigger when the task requires grounded claims against primary sources in any of these domains; the skill injects the matching source taxonomy. Skip for tasks not implicated by any listed domain, for general code writing, for conversational or opinion questions, and for questions about how this current Claude session should behave.
version: 0.1.0
contract-version: 2026-04-18
---

# Domain Research Skill

## Purpose

Enforce a single invariant Research_Protocol across every research task the skill touches, and layer a domain-specific source taxonomy on top when the task implicates one of thirteen defined domains. The invariant decides *how* claims are grounded. The domain layer decides *what counts as a primary source in this domain specifically* and *what noise looks like here*.

The protocol below is locked. Domain taxonomies live in `resources/` and load on demand. When no domain matches, the invariant governs alone and the skill stays silent beyond that.

## When To Use

Invoke this skill when the task requires substantiated claims and falls into at least one of the thirteen covered domains. Concrete triggers:

- User is drafting, reviewing, or auditing a federal proposal, capture plan, teaming analysis, set-aside eligibility memo, or state/local procurement response.
- User is compiling research on a nonprofit, grantmaker, or foundation.
- User is evaluating a Salesforce integration design, AppExchange package, or API/governor-limit question tied to a specific release.
- User is working on a HIPAA / HITECH / state health-privacy compliance artifact.
- User is producing a WCAG / Section 508 conformance report, VPAT, or accessibility audit.
- User is analyzing a CVE, writing or reviewing a WAF rule, or assessing a vendor advisory.
- User is benchmarking LLMs, comparing accelerator hardware, or evaluating an inference stack against published results.
- User is researching Japanese statute, precedent, or agency guidance, especially where bilingual sources are in play.
- User is conducting technical research on AI/ML, infrastructure, or software engineering that requires citations to primary sources.
- User is preparing for CMMC certification, drafting a System Security Plan / POA&M against NIST SP 800-171, calculating an SPRS score, or navigating DFARS cybersecurity clauses.

## When NOT To Use

- Task is general programming, refactoring, or debugging where no claim needs external substantiation.
- Task is conversational, opinion-gathering, or brainstorming without a deliverable that cites sources.
- Task is about how this Claude session should behave, not about research output.
- Task is a simple factual lookup the user's own files already answer.
- Task is in a domain not on the coverage list. Say so plainly; do not improvise a fake domain layer.
- Task is a benchmark ranking or "which is best" question. The skill grounds claims against primaries; it does not manufacture comparative judgments.

## The Invariant — Research_Protocol

The protocol below is always in effect when this skill is active. Domain resources never modify it; they only specify what substantiates a claim in their domain. If a domain resource and this protocol appear to conflict, the protocol wins.

<Research_Protocol>
Goal: Ground claims in primary sources; reject AI-generated summaries, SEO content, and marketing copy.

- Source Tiers:
  - Primary (substantiates claims): the actual artifact — regulation, dataset, specification, original research, signed document, source code.
  - Contextual (frames only, never substantiates): analyst reports, trade press, expert commentary.
  - Disqualified: AI-generated summaries, SEO listicles, vendor marketing, undated or unversioned references, content that paraphrases primaries without citing them.
- Chain-of-Verification: For each source, verify authorship/authority, evidence density (cites primaries vs. restates secondaries), and currency.
- Deterministic Retreat: If no primary source supports a substantive claim, halt and return: "Insufficient primary-source grounding. Required: <source type>." Do not paraphrase contextual sources into the gap.
- Provenance: Every claim carries source, locator, and date. Citations are not consolidated away during synthesis.
</Research_Protocol>

## Activation Logic

1. **Detect domain implication** from task context. Signals include keywords, artifact types the user is producing or reading, the user's stated role, or an explicit domain cue. Use the routing table below.
2. **Load the matching resource file(s)** from `resources/`. Each file injects one `<Source_Domain role="...">` block.
3. **If multiple domains match**, load all matching blocks and flag the overlap at the top of the response (see Multi-Domain Overlap below). This is expected for federal work.
4. **If no domain matches**, stay silent on the domain layer. The Research_Protocol alone governs. Do not invent a Source_Domain block for an uncovered domain.
5. **Read the loaded file's front matter first.** If `retrieved:` is older than 90 days, tell the user before relying on the content.

## Routing Table

| Domain                                          | Trigger signals                                                                                                                                                       | Resource file                           |
|-------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| Technical research (AI/ML, infra, SWE)          | arXiv, eval harness, RFC, W3C spec, vendor docs at a specific version, reproducible benchmark                                                                         | `technical-research.md`                 |
| GovCon proposal writing                         | RFP / RFQ / RFI / SS, Section L, Section M, FAR / DFARS clause, compliance matrix, past performance, pricing narrative, SAM.gov representations                       | `govcon-proposal.md`                    |
| Federal capture & contract vehicles             | GSA MAS SIN, GWAC, IDIQ, OASIS+, SEWP, Alliant, agency acquisition forecast, FPDS-NG, USAspending, sources sought, teaming landscape                                  | `federal-capture.md`                    |
| Small business set-aside & certification        | HUBZone, 8(a), WOSB / EDWOSB, SDVOSB, SDB, NAICS size standard, SBA OHA decision, DSBS, 13 CFR 121 / 124 / 125 / 126 / 127 / 128                                      | `small-business-setaside.md`            |
| State & local government procurement            | eVA, Cal eProcure, NYS OGS, Texas SmartBuy, COMMBUYS, BidBuy, NASPO ValuePoint, Sourcewell, OMNIA, TIPS, BuyBoard, state master contract, city/county RFP             | `state-local-procurement.md`            |
| Nonprofit sector research                       | IRS Form 990, 990-PF, Candid, GuideStar, ProPublica Nonprofit Explorer, TEOS, foundation, grantmaker, state charity registration, Single Audit                         | `nonprofit-sector.md`                   |
| Salesforce integration architecture             | Apex, Flow, governor limits, Lightning, AppExchange, Salesforce API version, Trust status, Release Notes (Spring/Summer/Winter YY), Well-Architected                   | `salesforce-architecture.md`            |
| Healthcare data compliance                      | HIPAA, HITECH, 45 CFR 160 / 162 / 164, OCR guidance, NIST SP 800-66 Rev. 2, CMIA, SHIELD, My Health My Data, BAA, breach notification                                 | `healthcare-compliance.md`              |
| Accessibility & Section 508                     | WCAG 2.x, Revised 508, 36 CFR 1194, VPAT, ARIA, ATAG, Trusted Tester, DOJ ADA Title II 2024 rule, California AB 434                                                    | `accessibility-508.md`                  |
| Web security & WAF configuration                | CVE, NVD, CVSS, OWASP Top 10, ASVS, WSTG, CISA KEV, ModSecurity CRS, WAF rule, MITRE ATT&CK, vendor advisory                                                          | `web-security-waf.md`                   |
| AI/ML model & hardware evaluation               | MLPerf, SWE-Bench, ARC-AGI-2, MMLU-Pro, MMMU, GPQA, HumanEval, HELM, LM Evaluation Harness, model card, GPU/TPU spec sheet, harness commit hash                       | `ai-ml-evaluation.md`                   |
| Japanese legal & regulatory                     | e-Gov法令検索, 法令番号, 官報, 判例, 通達, 金融庁 / 経済産業省 / 消費者庁 / 個人情報保護委員会, bilingual Japanese-English legal text, 施行日                        | `japanese-legal.md`                     |
| Federal contractor cybersecurity (CMMC)         | CMMC, 32 CFR Part 170, DFARS 252.204-7012 / -7019 / -7020 / -7021, NIST SP 800-171 (Rev. 3), NIST SP 800-172, SPRS, CUI, C3PAO, Cyber AB, FCI, POA&M, SSP                 | `federal-contractor-cybersecurity.md`   |

## Multi-Domain Overlap

Federal work routinely activates the GovCon proposal, federal capture, and set-aside domains together. The CMMC / federal contractor cybersecurity domain joins that cluster whenever DFARS 252.204-7012 / -7021 appears in a DoD solicitation — which is most of them. State/local and small-business domains also co-occur when a small business is pursuing state cooperative contracts under a federal-adjacent posture. Accessibility and GovCon domains co-occur whenever Section 508 is a Section L response requirement.

When multiple domains load, do the following:

1. State the overlap at the top of the response: "Multiple domains active: <list>. Merging source taxonomies."
2. Apply the union of each domain's Tier 1 requirements. A claim about set-aside eligibility in a federal proposal needs to satisfy both domains' evidence thresholds.
3. Apply the union of each domain's Disqualified list. If any domain disqualifies a source, it is out.
4. Preserve per-domain currency requirements separately. Do not average them.

## Output Protocol

When the skill is active and at least one domain matches:

1. Inject the matching `<Source_Domain role="...">` block(s) from the loaded resources into the working context alongside the Research_Protocol.
2. Treat injected Tier 1 lists as the floor for substantiation. If a claim cannot be grounded in a domain Tier 1 source, trigger Deterministic Retreat; do not substitute a Tier 2 source.
3. Cite inline with source, locator (URL, document section, regulation CFR cite, case number, PIID, 法令番号), and date.
4. Never silently consolidate citations during synthesis. Aggregated claims must list every supporting primary.
5. If the user's task implies a domain on the list but the user has not provided any sources, produce a bounded list of specifically what is needed (e.g., "Required: the current RFP PDF, Amendments 1–N, and the FAR clauses incorporated by reference").

When the skill is active and no domain matches, apply the Research_Protocol alone and do not describe a domain layer.

## Deterministic Retreat Procedure

When a substantive claim lacks primary-source grounding:

1. Stop drafting. Do not fill the gap with contextual-tier paraphrase.
2. Return: `Insufficient primary-source grounding. Required: <specific source type from the active domain's Tier 1 list>.`
3. Offer the minimum acquisition step that would unblock the claim (which database to query, which document to request, which statute section to read).
4. Do not speculate about what the primary source probably says.

Retreat is the correct behavior, not a failure. Do not apologize for it or restructure the response around avoiding it.

## Coverage Status

| Domain                                          | Status    | Notes                                                                                   |
|-------------------------------------------------|-----------|-----------------------------------------------------------------------------------------|
| Technical research (AI/ML, infra, SWE)          | published | Covers arXiv / RFC / W3C / vendor-doc-versioning practice                                |
| GovCon proposal writing                         | published | US federal solicitation response; Sections L/M, compliance matrices, past performance, price |
| Federal capture & contract vehicles             | published | Precedes solicitation release; FPDS-NG, USAspending, eLibrary, GWACs, IDIQs              |
| Small business set-aside & certification        | published | HUBZone / 8(a) / WOSB / SDVOSB / SDB; 13 CFR parts; SBA OHA                              |
| State & local government procurement            | published | Fragmented per jurisdiction; NASPO ValuePoint, state portals, cooperative purchasing     |
| Nonprofit sector research                       | published | IRS 990s, state charity registrations, foundation disclosures                            |
| Salesforce integration architecture             | published | Release-versioned docs, Trust, governor limits, API versioning, AppExchange security     |
| Healthcare data compliance                      | published | Compliance-oriented; HIPAA / HITECH / OCR / NIST SP 800-66 Rev. 2 / state variants       |
| Accessibility & Section 508                     | published | WCAG 2.x, Revised 508, VPATs, state equivalents, 2024 DOJ ADA Title II rule              |
| Web security & WAF configuration                | published | CVE / NVD / OWASP / CISA KEV / CRS / vendor advisories / MITRE ATT&CK                    |
| AI/ML model & hardware evaluation               | published | Benchmarks with harness commit and hardware; MLPerf; spec sheets; not marketing          |
| Japanese legal & regulatory                     | published | e-Gov法令検索 primacy; 判例 citations; bilingual handling                                 |
| Federal contractor cybersecurity (CMMC)         | published | 32 CFR Part 170 (2024-10-15), DFARS cyber clauses, NIST SP 800-171 Rev. 3 (2024-05-14)    |

## Escalation

- **User's task implicates a domain not on the list.** Say so. Offer to apply the Research_Protocol alone, or to draft a new domain layer following the tuning rubric in `README.md`.
- **User's primary sources contradict a loaded Tier 1 list.** Do not silently override the user. Ask for the source, reconcile, and update the resource file as a follow-on PR rather than papering over it in the session.
- **User asserts a claim contextual sources support but no primary does.** Deterministic Retreat. Offer the minimum acquisition step.

## Tuning Rubric (Used When Authoring a New Domain Resource)

Each new domain entry must answer, with specifics:

1. What is the authoritative artifact in this domain? (populates Tier_1_Primary)
2. What sources frame but cannot substantiate? (populates Tier_2_Contextual)
3. What does noise look like specifically here? (populates Disqualified)
4. What is the minimum citation type required before a claim ships? (populates Evidence_Threshold)
5. What revision or currency requirement applies? (embedded in Evidence_Threshold)

Generic entries fail the skill's purpose. Name actual databases, actual registries, actual noise patterns. Where a domain has well-known canonical sources (FPDS-NG, NVD, e-Gov法令検索, Candid), name them explicitly in Tier 1.

A new domain resource is not complete until a skilled practitioner in that domain could use it to decide which sources to accept and which to reject without further guidance.

## Adding a New Domain

1. Copy the block layout from an existing resource in `resources/`.
2. Populate Tier_1_Primary, Tier_2_Contextual, Disqualified, and Evidence_Threshold per the tuning rubric above.
3. Add the domain row to the Routing Table, Coverage Status table, and, if the domain will routinely co-occur with others, update Multi-Domain Overlap.
4. Self-audit: Tier 1 entries name specific artifacts or databases, Disqualified entries name specific noise patterns, Evidence_Threshold is citation-level specific, front matter has `retrieved:` and `primary_sources:`.
5. Do not commit. Human review is required before the domain lands on a public branch.
