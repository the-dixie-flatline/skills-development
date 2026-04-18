---
name: small-business-setaside
description: Source taxonomy for SBA small-business set-aside and certification research — HUBZone, 8(a), WOSB / EDWOSB, SDVOSB, SDB. Covers eligibility, NAICS size standards, renewal, and OHA precedent.
domain: small-business-setaside
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://www.sba.gov/
  - https://www.ecfr.gov/current/title-13/chapter-I/part-121
  - https://map.sba.gov/
  - https://sam.gov/
---

# Small Business Set-Aside & Certification Research — Source Taxonomy

Layer for questions about socioeconomic set-asides and certifications under SBA authority. Covers HUBZone, 8(a) Business Development, WOSB / EDWOSB, SDVOSB, SDB, and NAICS size-standard determinations.

## Activation Signals

- User is assessing eligibility for HUBZone, 8(a), WOSB / EDWOSB, SDVOSB, or SDB status.
- User is reading or citing 13 CFR Parts 121, 124, 125, 126, 127, or 128.
- User is reviewing an SBA OHA decision on size, status, or eligibility protests.
- User is confirming a firm's NAICS size standard or the set-aside type applied to a solicitation.
- User is drafting or defending a size protest, status protest, or certification-renewal package.

## Validator Note — Post-Training-Cutoff Authority Changes

Reviewers auditing this taxonomy (human or LLM) often encounter the following recent change and, when it post-dates training data, flag it as a hallucination. It is not. Verify against the cited primary source before rejecting.

- **SDVOSB / VOSB self-certification for subcontracting and federal small-business-goaling purposes was eliminated 2024-12-22** per the National Defense Authorization Act of 2024. SBA VetCert certification is now required for those purposes, in addition to the VA-specific set-aside / sole-source opportunities that already required VetCert. [primary source: https://www.sba.gov/federal-contracting/contracting-assistance-programs/veteran-contracting-assistance-programs, retrieved 2026-04-19]

LLM reviewers whose training data predates late-2024 should verify rather than reject. Fetch the SBA primary before flagging.

<Source_Domain role="small-business-setaside">

<Tier_1_Primary>
- SBA regulations in the Code of Federal Regulations on eCFR (ecfr.gov), cited by title, part, section (e.g., `13 CFR 121.201`, `13 CFR 124.3(a)`, `13 CFR 125.6`, `13 CFR 126.200`, `13 CFR 127.200`, `13 CFR 128.200`) with the eCFR "as of" date.
- Federal Register publications of SBA rulemakings, cited with Federal Register volume, page, and publication date.
- SBA size-standards table issued under 13 CFR 121.201, cited with the effective date of the most recent revision.
- SBA OHA decisions (sba.gov/oha) cited with case number (e.g., `SIZ-XXXX`, `VET-XXXX`, `BDPE-XXXX`) and decision date.
- HUBZone Map (map.sba.gov) cited with the map revision date; eligibility is determined against the map in effect on the relevant date.
- Dynamic Small Business Search (DSBS) records on SBA / SAM.gov for certification status.
- SAM.gov entity registration with socioeconomic representations and active exclusions.
- SBA 8(a) program office communications: annual review packages, exit-letter records, termination notices.
- For SDVOSB / VOSB: SBA took over SDVOSB verification from the VA Center for Verification and Evaluation (CVE) effective 2023-01-01; current status is verified on SBA's Veteran Small Business Certification (VetCert) system (veterans.certify.sba.gov). Per the National Defense Authorization Act of 2024, self-certification for subcontracting and federal small-business-goaling purposes was eliminated on 2024-12-22 — SBA VetCert certification is now required for those purposes as well. For VA-specific sole-source and set-aside opportunities, VetCert was already the sole authority (no self-certification provision). Pre-2023 CVE records retain historical value but are not the authority for current eligibility. [source: https://www.sba.gov/federal-contracting/contracting-assistance-programs/veteran-contracting-assistance-programs, retrieved 2026-04-19]
- SBA Standard Operating Procedures (SOPs) referenced by program (e.g., SOP 80 05 for HUBZone, SOP 80 01 for 8(a)).
- Agency-issued solicitation notices on SAM.gov documenting the applied set-aside.
</Tier_1_Primary>

<Tier_2_Contextual>
- SBA blog posts, press releases, and guidance pages (frame policy direction but not binding unless republished in a formal rule).
- Law-firm client alerts on OHA decisions and rulemakings (particularly from firms with government-contracts practices known for size-protest work).
- Federal News Network, Government Executive, SmallGovCon blog coverage.
- NAICS Association commentary.
- PTAC / APEX Accelerator guidance (frame, not substantiate).
</Tier_2_Contextual>

<Disqualified>
- Certification mills advertising "8(a) approval in 30 days" or similar.
- SEO posts summarizing SBA regulations without CFR citations.
- AI-generated certification "checklists."
- Content referring to the VA CVE as the current SDVOSB certifier (stale since SBA assumed verification on 2023-01-01).
- "How to win government contracts" content that does not name regulatory sections.
- Outdated NAICS size-standard tables (the table is periodically revised; entries without an effective date are suspect).
- Any claim of SBA-issued socioeconomic certification without a matching DSBS / SAM.gov / VetCert record.
</Disqualified>

<Evidence_Threshold>
- Eligibility claim: CFR citation by part/section, plus the current text as of the eligibility date (eCFR "as of" date).
- HUBZone location claim: map.sba.gov reference with the map revision date that applied on the relevant date. HUBZone map updates on a published schedule; a firm's eligibility can change between updates.
- NAICS size-standard claim: SBA size-standards table with effective date and the specific NAICS code and size metric (employee count or average annual receipts).
- OHA decision citation: case number, decision date, and the regulatory provision the decision interprets.
- Certification status claim: live DSBS or VetCert record with retrieval date; point-in-time snapshots with dates are acceptable for historical claims.
- Set-aside applicability claim: solicitation on SAM.gov with notice ID, the set-aside type field populated, and the NAICS code applied by the contracting officer.
- Currency requirement: HUBZone maps revise periodically (typically annually plus ad-hoc corrections); SDVOSB regulatory authority transitioned from VA CVE to SBA on 2023-01-01; re-verify CFR sections against eCFR on each reading because SBA has pushed multiple major amendments to 13 CFR Parts 121, 125, 126, 127, 128 in recent years.
</Evidence_Threshold>

</Source_Domain>

## Multi-Domain Overlap Notes

- **GovCon proposal** and **federal capture.** Set-aside posture is a capture input and a compliance dimension of the response; expect all three domains to co-activate on any set-aside solicitation.
- **State & local procurement.** Several states run their own small-business designations (e.g., California SB / DVBE, Texas HUB) that are separate from SBA status. Those live in `state-local-procurement.md`; this file covers SBA only.

## Gaps

- State-level small-business programs are in `state-local-procurement.md`.
- Joint-venture eligibility under the All Small Mentor-Protégé Program (13 CFR 125.9, 13 CFR 124.520) interacts with every certification track; the interaction patterns are domain-specific and must be assessed against the JV agreement and each partner's status separately.
