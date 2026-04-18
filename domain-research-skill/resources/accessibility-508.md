---
name: accessibility-508
description: Source taxonomy for accessibility and Section 508 compliance — WCAG 2.x, Revised 508 Standards (36 CFR 1194), VPATs, ARIA, ATAG, state equivalents, 2024 DOJ ADA Title II rule.
domain: accessibility-508
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://www.w3.org/TR/WCAG22/
  - https://www.access-board.gov/
  - https://www.section508.gov/
  - https://www.federalregister.gov/api/v1/documents/2026-07663.json
  - https://www.iso.org/obp/ui/en/#iso:std:iso-iec:40500:ed-2:v1:en
---

# Accessibility & Section 508 Compliance — Source Taxonomy

Layer for conformance work against WCAG, Section 508, and related state / cross-border accessibility regimes. Applies to VPAT authoring, accessibility audits, remediation plans, and procurement-response Section 508 commitments.

## Activation Signals

- User is authoring or reviewing a VPAT (Voluntary Product Accessibility Template), Accessibility Conformance Report (ACR), or remediation plan.
- User is citing WCAG 2.0, 2.1, or 2.2 success criteria by number, or the Revised 508 Standards.
- User is responding to Section L language invoking Section 508 in a federal solicitation.
- User is mapping state accessibility obligations (California AB 434, Minnesota 16E.03, Colorado HB21-1110, Texas 1TAC Rule 206/213, New York ITS-OPP-S14-002).
- User is evaluating ARIA 1.2 / 1.3 authoring practices or ATAG 2.0 authoring-tool obligations.

## Validator Note — Post-Training-Cutoff Authority Changes

Reviewers auditing this taxonomy (human or LLM) often encounter the following recent changes and, when they post-date training data, flag them as hallucinations. They are not. Verify against the cited primary sources before rejecting.

- **DOJ ADA Title II Interim Final Rule** (FR Doc. 2026-07663, published and effective 2026-04-20) extended compliance dates for web and mobile accessibility requirements under 28 CFR Part 35 Subpart H: entities with 50,000+ population are now due 2027-04-26 (previously 2026-04-24); smaller entities and special districts are now due 2028-04-26 (previously 2027-04-26). [primary source: https://www.federalregister.gov/api/v1/documents/2026-07663.json, retrieved 2026-04-19]
- **WCAG 2.2 adopted as ISO/IEC 40500:2025 (Edition 2)**, superseding ISO/IEC 40500:2012 which had adopted WCAG 2.0. [primary source: https://www.iso.org/obp/ui/en/#iso:std:iso-iec:40500:ed-2:v1:en, retrieved 2026-04-19]

LLM reviewers whose training data predates 2025 or early 2026 should verify rather than reject these references. Fetch the FR JSON API or ISO OBP primary before flagging.

<Source_Domain role="accessibility-508">

<Tier_1_Primary>
- WCAG 2.2 (w3.org/TR/WCAG22/) — current W3C Recommendation; originally published 2023-10-05, current Recommendation version dated 2024-12-12. Adopted by ISO/IEC as **ISO/IEC 40500:2025 (Edition 2)**, superseding ISO/IEC 40500:2012 which had adopted WCAG 2.0. [sources: https://www.w3.org/TR/WCAG22/; https://www.iso.org/obp/ui/en/#iso:std:iso-iec:40500:ed-2:v1:en, retrieved 2026-04-19]
- WCAG 2.1 (w3.org/TR/WCAG21/) — W3C Recommendation; still the standard incorporated by reference in current Revised 508.
- WCAG 2.0 (w3.org/TR/WCAG20/) — cited for legacy claims only.
- Revised Section 508 Standards (36 CFR Part 1194, Appendix A and C), adopted January 2017 (Access Board final rule 82 FR 5790), cited via ecfr.gov or access-board.gov. Revised 508 incorporates WCAG 2.0 Level A and AA.
- U.S. Access Board (access-board.gov) guidance documents on ICT Accessibility Standards.
- Section508.gov (GSA) — Trusted Tester Conformance Test Process documentation and accessibility requirements tool (ART).
- DOJ rule on web and mobile accessibility for state and local governments (Title II ADA; 28 CFR Part 35 Subpart H). Original rule at 89 FR 31320 (2024-04-24). Interim Final Rule (FR Doc. 2026-07663, published 2026-04-20, effective 2026-04-20) extended compliance dates: entities with 50,000+ population now **2027-04-26** (previously 2026-04-24); smaller entities and special districts now **2028-04-26** (previously 2027-04-26). [source: https://www.federalregister.gov/api/v1/documents/2026-07663.json, retrieved 2026-04-19]
- ARIA 1.2 / 1.3 specifications (w3.org/TR/wai-aria-1.2/, w3.org/TR/wai-aria-1.3/) as Recommendations or Candidate Recommendations at the appropriate version.
- ATAG 2.0 (w3.org/TR/ATAG20/) for authoring-tool accessibility.
- WCAG 2.x Techniques and Understanding documents (Working Group Notes; primary for the Working Group's own interpretive reading).
- VPATs produced to the ITI VPAT 2.5 template or later (itic.org/policy/accessibility/vpat), cited with the VPAT version and report date.
- European harmonized standard EN 301 549 v3.2.1 (etsi.org) where cross-reference is required.
- State statutes and rules (cited with state code and effective date):
  - California AB 434 and California Government Code § 11546.7 (state websites).
  - California Unruh Civil Rights Act interpretations (Cal. Civ. Code § 51); note Ninth Circuit case law including Robles v. Domino's as interpretive context, not primary for WCAG criteria.
  - Colorado HB21-1110 (Colo. Rev. Stat. § 24-85-103) and OIT Technology Accessibility Rules.
  - Minnesota Statute 16E.03 Subd. 9.
  - Texas 1TAC Rule 206 and 213.
  - New York ITS-OPP-S14-002.
- DOL OFCCP Section 503 contractor requirements where they bear on employment-site accessibility.
- FCC 21st Century Communications and Video Accessibility Act (CVAA) regulations for telecommunications accessibility (47 CFR Parts 6, 7, 14).
</Tier_1_Primary>

<Tier_2_Contextual>
- Deque University, Level Access, TPGi, Siteimprove technical analyses.
- WAI Working Drafts and Working Group Notes in progress (not yet normative, but authoritative for the Working Group's current thinking).
- Legal case summaries (Gil v. Winn-Dixie, Robles v. Domino's, NFB v. Target) — frame interpretation of ADA applicability to web; do not substantiate WCAG success-criteria claims.
- Practitioner blogs by well-known accessibility engineers (Adrian Roselli, Sara Soueidan, Heydon Pickering, Léonie Watson) — frame technique; verify specifics against specs.
- WebAIM (webaim.org) guidance and the WebAIM Million report.
- Section508.gov community-of-practice materials (primary when they reproduce normative guidance; secondary for practice commentary).
</Tier_2_Contextual>

<Disqualified>
- "Accessibility overlay" marketing (AccessiBe, UserWay, EqualWeb, accessiBe competitor marketing) claiming "one-line WCAG compliance."
- AI-written accessibility guides paraphrasing WCAG without citing success criteria.
- Content treating WCAG 2.0 AA compliance as sufficient for current federal procurement without flagging the DOJ Title II 2024 rule or state laws that have moved to WCAG 2.1 or 2.2.
- AI-generated VPATs.
- VPATs not using the ITI VPAT 2.x family template.
- Compliance claims without specific WCAG success-criterion references.
- Blog posts citing "WCAG AAA compliance" for entire sites without acknowledging AAA is rarely-appropriate scope.
- "Accessible PDF" marketing without citing PDF/UA (ISO 14289).
</Disqualified>

<Evidence_Threshold>
- Conformance claim on a success criterion: WCAG success-criterion number with level and version (e.g., `1.3.1 Info and Relationships, Level A, WCAG 2.2`). Claim is accompanied by the test result and the tested page / component.
- VPAT claim: specific VPAT template version (e.g., VPAT 2.5 WCAG / Section 508 / EU / INT), report date, and author.
- Section 508 federal-contracting claim: Revised 508 Standards (36 CFR Part 1194 Appendix A/C) with the specific functional performance criterion or technical requirement.
- DOJ Title II claim: 28 CFR Part 35 Subpart H, with the compliance date tier for the specific public entity, reflecting the IFR extension (FR Doc. 2026-07663) — 2027-04-26 for 50,000+ population entities, 2028-04-26 for smaller entities and special districts.
- State-law claim: state statute with most-recent-amendment date; note compliance baseline (many states now track WCAG 2.1 AA or 2.2 AA, not 2.0).
- ARIA pattern claim: WAI-ARIA Authoring Practices Guide (APG) with the pattern name and the ARIA version it is tested against.
- Testing methodology claim: Trusted Tester Conformance Test Process version, or a published alternate methodology (IAAP CPACC / WAS). Ad-hoc methodology is not substantiation.
- Currency requirement: WCAG is under active work (WCAG 3 is in Working Draft); cite Recommendation vs Working Draft status explicitly. Revised 508's incorporation of WCAG 2.0 lags WCAG current version; federal procurement references may still cite 2.0 AA until 508 is re-updated. DOJ Title II compliance dates were extended by IFR 2026-07663 (2026-04-20): 2027-04-26 for 50k+ population entities, 2028-04-26 for smaller entities and special districts. Watch for the follow-on Final Rule.
</Evidence_Threshold>

</Source_Domain>

## Gaps

- Mobile-platform accessibility (iOS / Android specific) sits under WCAG 2 but platform-specific testing guidance is fragmented; cite Apple and Google accessibility docs directly for platform-API claims.
- Cognitive and learning-disability success criteria are the weakest area in WCAG 2.x; the COGA Task Force work is in progress and claims that depend on it should flag the unfinished state of the normative guidance.
- PDF accessibility claims should cite PDF/UA (ISO 14289-1:2014 and -2:2024) alongside WCAG.
