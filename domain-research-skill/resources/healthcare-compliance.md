---
name: healthcare-compliance
description: Source taxonomy for healthcare data compliance research — HIPAA, HITECH, HHS OCR guidance, NIST SP 800-66 Rev. 2, state variants. Compliance-oriented, not clinical.
domain: healthcare-compliance
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://www.hhs.gov/hipaa/
  - https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C
  - https://csrc.nist.gov/publications/detail/sp/800-66/rev-2/final
---

# Healthcare Data Compliance — Source Taxonomy

Layer for compliance work on protected health information (PHI) and related state-law variants. Scope is HIPAA / HITECH / OCR enforcement, NIST implementation guidance, Business Associate relationships, breach-notification obligations, and state variants. Clinical / evidence-based-medicine research is out of scope.

## Activation Signals

- User is drafting, reviewing, or auditing a Business Associate Agreement, breach-notification letter, Notice of Privacy Practices, risk analysis, or HIPAA Security Rule control mapping.
- User is reading or citing 45 CFR Parts 160, 162, or 164, HITECH, OCR resolution agreements, or NIST SP 800-66 Rev. 2.
- User is evaluating a state health-privacy statute (California CMIA, New York SHIELD, Washington My Health My Data, Texas Medical Records Privacy Act, etc.).
- User is drafting a data-processing addendum or security exhibit for a Covered Entity or Business Associate.

<Source_Domain role="healthcare-compliance">

<Tier_1_Primary>
- HIPAA Privacy Rule: 45 CFR Parts 160 and 164 Subparts A and E on eCFR (ecfr.gov).
- HIPAA Security Rule: 45 CFR Parts 160 and 164 Subparts A and C (§§ 164.302–164.318) on eCFR.
- HIPAA Breach Notification Rule: 45 CFR Parts 160 and 164 Subpart D (§§ 164.400–164.414) on eCFR.
- HIPAA Transactions and Code Sets: 45 CFR Part 162.
- HITECH Act, Public Law 111-5, Title XIII (Subtitle D for privacy/security modifications).
- Omnibus Rule (78 FR 5566, January 25, 2013) and subsequent modifications.
- HHS OCR Guidance (hhs.gov/hipaa/for-professionals/guidance) — official interpretive guidance on Privacy, Security, and Breach Notification Rules.
- HHS OCR Resolution Agreements and Civil Monetary Penalties, cited with case number, covered entity, settlement amount, and date.
- HHS OCR Breach Notification Portal ("Wall of Shame," hhs.gov/ocr/breach/) for the 500+ breach record.
- NIST SP 800-66 Rev. 2 (Implementing the HIPAA Security Rule, issued February 2024), cited by section.
- NIST SP 800-53 Rev. 5 and NIST Cybersecurity Framework 2.0 for control mappings referenced by OCR.
- FDA medical-device cybersecurity guidance where the device is the data source: 2023 FDA Cybersecurity in Medical Devices Final Guidance (fda.gov) and 21 CFR Part 820.
- State statutes (cited by state code and section with effective date):
  - California Confidentiality of Medical Information Act (Civ. Code §§ 56–56.37) and California Consumer Privacy Act / CPRA interplay with medical data.
  - New York SHIELD Act (N.Y. Gen. Bus. Law § 899-bb).
  - Washington My Health My Data Act (RCW 19.373; effective March 31, 2024 for regulated entities).
  - Texas Medical Records Privacy Act (Tex. Health & Safety Code Ch. 181).
  - Massachusetts 201 CMR 17.00.
  - Nevada SB 220 / NRS Chapter 603A health-data provisions.
  - Connecticut Data Privacy Act health-data provisions (Public Act 22-15).
  - Florida Information Protection Act (Fla. Stat. § 501.171).
- HHS OIG compliance program guidance for the health-care industry (primary for the industry-segment guidance document it issues).
- CMS Conditions of Participation and Conditions for Coverage where PHI handling is implicated.
- Office of the National Coordinator (ONC) certification criteria (45 CFR Part 170) for certified health IT.
</Tier_1_Primary>

<Tier_2_Contextual>
- HHS OCR annual enforcement reports (frame priorities).
- AHIMA, HIMSS, HCCA (Health Care Compliance Association) published guidance.
- Law-firm analyses of OCR resolution agreements (Ropes & Gray, Hogan Lovells, Crowell, Polsinelli, etc.).
- Academic journals on health-information policy (Journal of AHIMA, Health Affairs).
- Practitioner blogs by named HIPAA attorneys, tied back to OCR guidance or rulemakings.
</Tier_2_Contextual>

<Disqualified>
- Vendor marketing claiming "HIPAA compliance" or "HIPAA certified" without citing specific technical controls. There is no federal HIPAA certification program; any claim of "HIPAA certification" is a noise signal.
- AI-generated HIPAA compliance summaries.
- Outdated content treating NIST SP 800-66 Rev. 1 (2008) as current after Rev. 2 was issued February 2024.
- Undated compliance checklists.
- Blog posts asserting BAA scope without quoting 45 CFR 164.502(e) and 164.504(e).
- Consulting-firm "HIPAA quick guides" without CFR citations.
- Content treating HIPAA as preemption-absolute where state law is more stringent (HIPAA sets a floor; more-stringent state law applies).
</Disqualified>

<Evidence_Threshold>
- Regulatory-requirement claim: 45 CFR section citation with the eCFR "as of" date.
- Breach-precedent claim: OCR resolution agreement with case number, covered entity, settlement amount, agreement date, and the specific finding of violation.
- Technical-control claim: NIST SP 800-66 Rev. 2 section with publication date (2024-02); NIST SP 800-53 Rev. 5 control ID; NIST CSF 2.0 function / category.
- State-variance claim: state statute citation with enactment and most-recent-amendment date; note explicitly whether the state provision is more stringent than HIPAA (if so, it applies independently).
- BAA scope claim: 45 CFR 164.502(e), 164.504(e), or the specific contractual provision being analyzed.
- Breach-notification-timing claim: the specific rule subsection (164.404 for individual notice, 164.406 for media notice, 164.408 for HHS notice) with the applicable timing window.
- Currency requirement: HIPAA Security Rule NPRM on strengthening cybersecurity was published late 2024; watch for the Final Rule publication in the Federal Register. NIST SP 800-66 Rev. 2 (2024-02) supersedes Rev. 1. State health-privacy law is an active area; re-verify state citations against the current state code annually and against active legislative sessions.
</Evidence_Threshold>

</Source_Domain>

## Gaps

- Research-use PHI (HIPAA § 164.512(i), 45 CFR 46 Common Rule interplay) has overlapping authorities; citations must track both HIPAA and IRB / Common Rule authority.
- The 2024 HIPAA Security Rule NPRM is not yet a Final Rule as of this file's retrieval date; compliance programs preparing for it should cite the NPRM as proposed, not as operative text.
- Cross-border data transfers (particularly to EU where GDPR applies, or UK, Canada, Japan) add an additional regime not covered here; cite the applicable cross-border instrument separately.
