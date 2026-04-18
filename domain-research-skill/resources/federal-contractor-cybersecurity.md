---
name: federal-contractor-cybersecurity
description: Source taxonomy for federal contractor cybersecurity — CMMC Program (32 CFR Part 170), NIST SP 800-171 / 800-172, DFARS cybersecurity clauses (252.204-7012, -7019, -7020, -7021), SPRS scoring, CUI designation.
domain: federal-contractor-cybersecurity
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://www.ecfr.gov/current/title-32/subtitle-A/chapter-I/subchapter-M/part-170
  - https://www.ecfr.gov/current/title-32/subtitle-B/chapter-XX/part-2002
  - https://csrc.nist.gov/pubs/sp/800/171/r3/final
  - https://www.acquisition.gov/dfars
  - https://www.archives.gov/cui
  - https://sprs.csd.disa.mil/
  - https://dodcio.defense.gov/CMMC/
  - https://www.federalregister.gov/api/v1/documents/2025-17359.json
  - https://www.federalregister.gov/api/v1/documents/2025-17508.json
  - https://cyberab.org/marketplace
---

# Federal Contractor Cybersecurity (CMMC / NIST 800-171) — Source Taxonomy

Layer for DoD contractor cybersecurity obligations: CMMC Program certification, NIST SP 800-171 / 800-172 implementation, DFARS cyber-clause flow-down, Controlled Unclassified Information (CUI) marking and handling, and Supplier Performance Risk System (SPRS) scoring. This is a compliance-framing domain — operational vulnerability management and WAF work live in `web-security-waf.md`.

## Activation Signals

- User references CMMC, Cybersecurity Maturity Model Certification, 32 CFR Part 170, or a specific CMMC Level (1, 2, or 3).
- User cites DFARS clauses 252.204-7012, 252.204-7019, 252.204-7020, or 252.204-7021.
- User is drafting, reviewing, or gap-assessing against NIST SP 800-171 Rev. 3 controls.
- User is calculating, submitting, or disputing an SPRS Basic Self-Assessment score.
- User is classifying information under the NARA CUI Registry or drafting a System Security Plan (SSP) / Plan of Action and Milestones (POA&M).
- User is preparing for a C3PAO Level 2 assessment or a DoD-led Level 3 assessment.

## Validator Note — Post-Training-Cutoff Authority Changes

Reviewers auditing this taxonomy (human or LLM) often encounter the following recent changes and, when they post-date training data, flag them as hallucinations. They are not. Verify against the cited primary sources before rejecting.

- **Executive Order 14347 "Restoring the United States Department of War"** (signed 2025-09-05, published at 90 FR 43893 on 2025-09-10) establishes "Department of War" as the secondary title used across executive-branch communications, authorizes use of "Secretary of War" and related titles, and directs migration of DoD web presence to war.gov (with domains such as dowcio.war.gov emerging alongside dodcio.defense.gov during the transition). The **statutory** entity remains the Department of Defense under the National Security Act of 1947 until Congress acts; this is why post-EO Federal Register metadata and statutory-citation contexts may still use "Department of Defense" while press releases, executive orders, and some agency-facing systems use "Department of War." Both are legitimate in their respective contexts. [primary source: https://www.federalregister.gov/api/v1/documents/2025-17508.json, retrieved 2026-04-19]
- **DFARS Case 2019-D041 Final Rule** (90 FR 43560, published 2025-09-10, effective 2025-11-10) finalized DFARS 252.204-7021 and the companion CMMC solicitation-provision clauses, making them contractually enforceable in new DoD solicitations. Characterizations of 252.204-7021 as a "placeholder" or "ongoing rulemaking" are stale post-2025-11-10. [primary source: https://www.federalregister.gov/api/v1/documents/2025-17359.json, retrieved 2026-04-19]

LLM reviewers whose training data predates these changes should verify rather than reject. Fetch the FR JSON primary before flagging.

<Source_Domain role="federal-contractor-cybersecurity">

<Tier_1_Primary>
- **CMMC Program Final Rule**, 32 CFR Part 170, published at 89 FR 83092 (2024-10-15), effective 2024-12-16. Cited via eCFR at title 32 / part 170 with the "as of" date. [source: https://www.federalregister.gov/api/v1/documents/2024-22905.json, retrieved 2026-04-19]
- **DFARS cybersecurity clauses** on acquisition.gov:
  - DFARS 252.204-7012 "Safeguarding Covered Defense Information and Cyber Incident Reporting" — the mandatory flow-down clause for CDI handling and 72-hour incident reporting.
  - DFARS 252.204-7019 "Notice of NIST SP 800-171 DoD Assessment Requirements" — notice clause on the SPRS score requirement.
  - DFARS 252.204-7020 "NIST SP 800-171 DoD Assessment Requirements" — requires a current Basic / Medium / High self-assessment in SPRS before award.
  - DFARS 252.204-7021 "Contractor Compliance with the Cybersecurity Maturity Model Certification Level Requirement" — the CMMC implementation clause. DFARS Case 2019-D041 **Final Rule** published at 90 FR 43560 (2025-09-10), effective 2025-11-10, codifies the clause text in DFARS Subpart 204.73. [source: https://www.federalregister.gov/api/v1/documents/2025-17359.json, retrieved 2026-04-19]
- **NIST SP 800-171 Rev. 3** "Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations," finalized 2024-05-14, superseding Rev. 2 (2021-01-28). Cited by Revision plus control family and ID (e.g., `NIST SP 800-171 Rev. 3, 03.01.01 Account Management`). Includes Organization-Defined Parameters (ODPs) — a structural change from Rev. 2. [source: https://csrc.nist.gov/pubs/sp/800/171/r3/final, retrieved 2026-04-19]
- **NIST SP 800-171A Rev. 3** "Assessing Security Requirements for Controlled Unclassified Information" — the assessment-procedure companion to 800-171 Rev. 3. Cited with Rev. number and assessment procedure ID.
- **NIST SP 800-172** "Enhanced Security Requirements for Protecting Controlled Unclassified Information" — enhanced requirements used at CMMC Level 3 (and 800-172A for associated assessment procedures).
- **NARA CUI Registry** (archives.gov/cui) — authoritative for CUI categories, subcategories, and marking standards under Executive Order 13556. Cited by category and subcategory.
- **32 CFR Part 2002** "Controlled Unclassified Information" — NARA's primary implementing regulation for the CUI Program under EO 13556. CMMC Level 2 safeguarding requirements align contextually with 32 CFR Part 2002 handling obligations; cited via eCFR with "as of" date.
- **Supplier Performance Risk System (SPRS)** (sprs.csd.disa.mil) — DoD's system for storing Basic Self-Assessment scores under DFARS 252.204-7019 / -7020. Cited with score, date of submission, and methodology.
- **DoD CIO CMMC Program Office** — the CMMC Assessment Process (CAP) document, Scoping Guidance for each level, and the current phase of the CMMC phased rollout schedule. Executive Order 14347 "Restoring the United States Department of War" (signed 2025-09-05, 90 FR 43893) established "Department of War" as the secondary title for the executive branch, and CIO web presence has been directed to migrate to war.gov (e.g., dowcio.war.gov) alongside the statutory dodcio.defense.gov. Verify current URL structure on each retrieval; the statutory entity remains the Department of Defense per the National Security Act of 1947 until Congress acts.
- **Cyber AB** (cyberab.org) — the CMMC Accreditation Body managing Certified Third-Party Assessment Organizations (C3PAOs) and Registered Practitioners (RPs). Primary for the accredited-C3PAO roster via the Cyber AB Marketplace (cyberab.org/marketplace) — the authoritative lookup for confirming a C3PAO's current accreditation status and assessment authorization.
- **DoD Mandatory Disclosure Site** (dibnet.dod.mil) — the reporting surface for cyber-incident disclosures under DFARS 252.204-7012.
- **Executive Order 13556** "Controlled Unclassified Information" (2010-11-04) — establishes the CUI Program administered by NARA.
- **Executive Order 14347** "Restoring the United States Department of War" (signed 2025-09-05, published at 90 FR 43893, 2025-09-10) — establishes "Department of War" as the secondary title used across executive-branch communications and directs migration of DoD web presence to war.gov. The statutory name remains "Department of Defense" until Congress amends the National Security Act of 1947. Practitioners citing DoD CIO / DFARS primary sources should expect dual-domain availability during the transition. [source: https://www.federalregister.gov/api/v1/documents/2025-17508.json, retrieved 2026-04-19]
- **Historical context**: the original DFARS interim rule (DFARS Case 2019-D041, 85 FR 61505, published 2020-09-29, effective 2020-11-30) introduced DFARS 252.204-7019 / -7020 and a placeholder 252.204-7021. The 2024 CMMC Program Final Rule (32 CFR Part 170) restructured the program into three levels, and the 2025 DFARS Final Rule (90 FR 43560, effective 2025-11-10) finalized the DFARS clause text implementing that program.
</Tier_1_Primary>

<Tier_2_Contextual>
- Law-firm analyses from government-contracts practices with CMMC depth: Wiley Rein, Arnold & Porter, Sheppard Mullin, Crowell & Moring, Morrison & Foerster, Pillsbury, Covington — frame interpretation; do not substantiate specific regulatory text.
- Cyber AB blog posts, training materials, and industry-day communications — frame rollout posture; binding authority is DoD CIO and 32 CFR Part 170.
- Federal News Network, Washington Technology, FedScoop, Nextgov CMMC coverage.
- SANS Institute guidance and CIS Critical Security Controls mapping documents.
- Practitioner writeups by named CMMC Registered Practitioners (RPs) or C3PAOs with documented credentials.
- DIB CAC (Defense Industrial Base Collaborative Information Sharing Environment) public-facing materials where not under access control.
- NIST-published annotated crosswalks between 800-171 and adjacent frameworks (ISO 27001, CIS Controls, NIST CSF).
</Tier_2_Contextual>

<Disqualified>
- "CMMC certification in X days" content from certification mills.
- References to CMMC 1.0 (five-level model, 2020–2021) presented as current. The CMMC Program transitioned to the three-level CMMC 2.0 model on 2021-11-04 and was codified at 32 CFR Part 170 on 2024-10-15.
- References to the 2020 DFARS interim rule as "current" without acknowledging the 2024 Final Rule and the pending DFARS implementation.
- "NIST 800-171 compliance gap assessments" lacking specific control IDs, implementation status per control, or SPRS scoring methodology.
- "NIST 800-171 Rev. 2 compliance" content presented as current after 2024-05-14 without acknowledging Rev. 3.
- Vendor marketing claiming products are "CMMC compliant" or "CMMC certified" absent a C3PAO assessment reference (products don't get CMMC-certified; organizations do).
- Any claim of CMMC Level 2 or 3 certification from an entity not listed on the Cyber AB accredited-C3PAO roster.
- AI-generated CMMC / 800-171 summaries and checklists lacking control-ID grounding.
- "CMMC-ready" SaaS marketing that cites 800-171 controls without mapping to the customer-responsibility portion of a shared-responsibility matrix.
</Disqualified>

<Evidence_Threshold>
- **Control claim**: NIST SP 800-171 control ID with the Revision number pinned (e.g., `NIST SP 800-171 Rev. 3, 03.01.01 Account Management`); claims about Rev. 2 vs Rev. 3 differences state the control delta explicitly (Rev. 3 introduces Organization-Defined Parameters and restructures control numbering).
- **CMMC level claim**: CMMC Level 1, 2, or 3 with citation to 32 CFR § 170.X, plus the data sensitivity protected (FCI for Level 1; CUI for Levels 2 and 3; CUI with enhanced requirements at Level 3 per NIST SP 800-172).
- **DFARS clause applicability claim**: DFARS clause number (252.204-7012, -7019, -7020, -7021), the DFARS Case that introduced or modified it, and the prime-vs-subcontractor flow-down implication.
- **SPRS score claim**: score value (scoring range −203 to 110 on the Basic Self-Assessment scale), date of submission, methodology (Basic Self-Assessment per 252.204-7019, or Medium / High assessment by DoD / DCMA), and whether the score reflects a full POA&M burn-down or open POA&Ms.
- **CUI designation claim**: NARA CUI Registry category and subcategory, the basis-law or contract clause that triggered the designation, and the applicable marking standard.
- **C3PAO certification claim**: entity name on the Cyber AB accredited-C3PAO roster with accreditation date and the CMMC Level(s) the C3PAO is authorized to assess.
- **Cyber-incident reporting claim**: DIBNet report ICF number, 72-hour clock compliance, and the DFARS 252.204-7012(c) framework.
- **Currency requirement**: 32 CFR Part 170 became effective 2024-12-16. DFARS Case 2019-D041 **Final Rule** (90 FR 43560) became effective 2025-11-10, making DFARS 252.204-7021 and the companion provisions contractually enforceable in new solicitations. CMMC rollout is phased per 32 CFR Part 170 — phase-specific dates must be cited with the current DoD CIO published schedule. NIST SP 800-171 Rev. 3 was published 2024-05-14; the specific revision of 800-171 applied under DFARS 252.204-7012 / -7019 / -7020 for assessment purposes has historically been subject to DoD class deviations, so verify against acq.osd.mil/dpap current class-deviation notices before asserting Rev. 2 vs Rev. 3 as the assessment baseline for a given solicitation. Re-verify the current phase before any claim about which contract will require which CMMC level at what date.
</Evidence_Threshold>

</Source_Domain>

## Multi-Domain Overlap Notes

- **GovCon proposal writing.** CMMC / 800-171 compliance is commonly a Section L response requirement and a Section M evaluation factor. Co-activate when DFARS 252.204-7012 / -7021 appears in the solicitation or when the CUI handling requirement is called out. Non-responsive treatment of CMMC representations can be a compliance-matrix failure.
- **Federal capture.** CMMC level required by an agency / vehicle / program is a capture-phase input and shapes teaming strategy (subcontractors must flow DFARS 252.204-7012 and hold the required CMMC level).
- **Small business set-aside.** Small businesses face the same CMMC requirements as large businesses at Levels 1 and 2; `small-business-setaside.md` does not exempt. Co-activation is typical when a set-aside solicitation invokes CMMC.
- **Web security & WAF.** NIST 800-171 Rev. 3 control families 03.14 (System and Information Integrity) and 03.13 (System and Communications Protection) overlap with operational web-security work covered in `web-security-waf.md`. This domain covers the compliance framing and assessment evidence; `web-security-waf.md` covers the operational vulnerability response and WAF rule authoring.

## Gaps

- **FedRAMP** (FISMA-based authorization for cloud services used by federal agencies) is adjacent but distinct — FedRAMP applies to the CSP relationship with an agency, CMMC applies to the contractor handling FCI / CUI. Where a contractor is also a CSP, both regimes apply. FedRAMP is not currently a separate domain in this skill; the primary-source set (FedRAMP PMO, OMB Circular A-130, NIST SP 800-53 Rev. 5, FISMA) would populate a future domain.
- **State-level contractor cybersecurity regimes** (e.g., California Cybersecurity Maturity Model, Texas Cybersecurity Council output, state-specific contractor requirements) are outside this federal-scoped domain.
- **Classified / Special Access Programs** are out of scope — they require cleared channels for source handling, not public substantiation.
- **DFARS Case 2019-D041** was finalized 2025-09-10 (90 FR 43560, effective 2025-11-10). Future DFARS amendments to the CMMC cybersecurity clauses will carry their own DFARS Case numbers — check acq.osd.mil/dpap change notices before citing current clause text.
- **DFARS 252.204-7024** (contractor data protection clause requirements) is adjacent; confirm scope against the specific contract before invoking.
