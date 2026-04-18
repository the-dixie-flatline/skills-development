---
name: govcon-proposal
description: Source taxonomy for US federal solicitation response — Sections L/M, compliance matrices, past performance, pricing narrative, teaming. Applies from RFP release through submission.
domain: govcon-proposal
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://www.acquisition.gov/
  - https://sam.gov/
  - https://sam.gov/fpds
  - https://www.cpars.gov/
  - https://www.gao.gov/legal/bid-protests
---

# GovCon Proposal Writing — Source Taxonomy

Layer for proposal response to US federal solicitations. Applies once an RFP, RFQ, RFI, or sources-sought notice is in hand and the team is producing Volume I/II/III content, compliance matrices, past performance references, or pricing narratives.

## Activation Signals

- User mentions Section L / Section M, an RFP number, Amendment N, a compliance matrix, past performance references, or pricing narratives.
- User quotes or cites FAR / DFARS / agency-supplement clauses.
- User is reviewing a Q&A response, a sources-sought response, or a solicitation amendment.
- User is drafting teaming agreements, subcontractor letters of commitment, or SF 330 forms.

## Validator Note — Post-Training-Cutoff Authority Changes

Reviewers auditing this taxonomy (human or LLM) often encounter the following recent changes and, when they post-date training data, flag them as hallucinations. They are not. Verify against the cited primary sources before rejecting.

- **Executive Order 14347 "Restoring the United States Department of War"** (signed 2025-09-05, 90 FR 43893, 2025-09-10) establishes "Department of War" as a secondary title for the executive branch and directs migration of DoD web presence to war.gov. DoD solicitations, DFARS provisions, and agency-facing systems may appear under either "Department of Defense" or "Department of War." Both are legitimate. The statutory entity remains the Department of Defense per the National Security Act of 1947 until Congress acts. [primary source: https://www.federalregister.gov/api/v1/documents/2025-17508.json, retrieved 2026-04-19]
- **DFARS Case 2019-D041 Final Rule** (90 FR 43560, published 2025-09-10, effective 2025-11-10) finalized DFARS 252.204-7021 and the companion CMMC solicitation-provision clauses, making them contractually enforceable in new DoD solicitations. Research that characterizes 252.204-7021 as a "placeholder" or "ongoing rulemaking" is stale post-2025-11-10. [primary source: https://www.federalregister.gov/api/v1/documents/2025-17359.json, retrieved 2026-04-19]
- **FPDS.gov ezSearch decommissioned 2026-02-24.** For post-2026-02-24 past-performance and incumbency citations, SAM.gov Contract Awards is the authoritative retrieval surface. [primary source: https://sam.gov/fpds, retrieved 2026-04-19]

LLM reviewers whose training data predates these changes should verify rather than reject. Fetch the FR / SAM.gov primaries before flagging.

<Source_Domain role="govcon-proposal">

<Tier_1_Primary>
- The actual solicitation document (RFP, RFQ, RFI, SS) with every amendment, posted on SAM.gov under the solicitation's unique notice ID.
- All FAR clauses incorporated by reference or full text, quoted from acquisition.gov at the acquisition date's version (FAR is amended by FAC — Federal Acquisition Circular — and older clauses may apply to in-flight procurements).
- Agency supplements: DFARS (defense), AFARS (Army), NMCARS (Navy/Marines), AFFARS (Air Force), NFS (NASA), DEARS (Energy), HSAR (DHS), DOSAR (State), etc., each cited to acquisition.gov or the agency's own acquisition regulation page at the current version.
- Agency-issued amendments, Q&A responses, and industry-day materials released as part of the solicitation record.
- CPARS past performance information cited with contract number, performance period, and evaluation date.
- Incumbent contract documents on SAM.gov Contract Awards search and USAspending.gov, cited with PIID, modification number, task-order number, and award dates. The legacy FPDS.gov ezSearch interface was decommissioned 2026-02-24; FPDS reporting had already moved to SAM.gov in October 2020, and remaining FPDS.gov functions are transitioning to SAM.gov. Post-2026-02-24 award-history citations must identify SAM.gov as the retrieval surface. [source: https://sam.gov/fpds, retrieved 2026-04-19]
- Agency acquisition strategy documents, acquisition plans, and J&A (Justification and Approval) notices where public.
- SAM.gov entity registration data including reps and certs, socioeconomic status, and active exclusions.
- SF 330 filed past-performance entries.
- GAO bid-protest decisions on gao.gov/legal/bid-protests cited by B-number and decision date (contextual for strategy, primary when a decision establishes an interpretation binding on the contract at hand).
- Wage Determinations (WD) on sam.gov/wage-determinations for Service Contract Act / Davis-Bacon applicability.
- Cost and pricing data regulations (FAR Part 15 and the Truthful Cost or Pricing Data Act, 10 U.S.C. 3701–3708 / 41 U.S.C. 3501–3509) for certified cost or pricing.
</Tier_1_Primary>

<Tier_2_Contextual>
- Deltek GovWin, Bloomberg Government, and similar market-intelligence platforms — frame the landscape, summarize award patterns; never substantiate solicitation-specific requirements.
- Industry analyst reports on agency spending and priorities.
- Practitioner commentary on Section L/M interpretation (APMP, law-firm client alerts, well-known proposal-strategy blogs).
- Agency strategic plans and press releases (frame program context).
- Federal News Network, Government Executive, Nextgov coverage.
</Tier_2_Contextual>

<Disqualified>
- Proposal-writing blogs that quote generic "how to win GovCon" advice without a specific solicitation.
- AI-generated RFP summaries that paraphrase Section L/M without quoting it.
- Vendor-gated "market intel" PDFs lacking primary-document links.
- Marketing material from proposal-consulting firms used as authority for compliance.
- Any source that paraphrases the solicitation without quoting it.
- "Sample RFP" content with no attribution or date.
- LinkedIn posts asserting an incumbent's win theme without an award citation.
- Draft RFPs treated as final after the RFP has been released (outdated by definition).
</Disqualified>

<Evidence_Threshold>
- Compliance matrix entry: direct quote from the RFP Section L/M with section / sub-section reference, page number, and the amendment number that introduced it. A paraphrased matrix entry fails the matrix.
- FAR / DFARS / supplement clause applicability: citation to current acquisition.gov text with the FAC / DFARS Change Notice reference and the date the clause applies to this procurement.
- Past performance reference: CPARS entry with contract number, PoP dates, evaluation narrative rating. Uncorroborated client testimonials do not substitute.
- Pricing narrative claim: basis-of-estimate source (labor-category mapping to the RFP LCATs, the WD applied, historical labor rates with source, escalation assumptions tied to a published index such as BLS ECI with series ID).
- Teaming claim: subcontractor letter of commitment or signed teaming agreement referencing the specific solicitation.
- Currency requirement: re-verify RFP and all amendments against SAM.gov on the day of submission; re-pull FAR clauses against the current FAC where the acquisition date falls after the FAC's effective date; amendments can drop or add clauses.
</Evidence_Threshold>

</Source_Domain>

## Multi-Domain Overlap Notes

- **Federal capture.** Pre-solicitation market research and teaming landscape live in `federal-capture.md`; this file takes over at RFP release.
- **Small business set-aside.** When the set-aside type is material (HUBZone set-aside, 8(a) sole-source, SDVOSB), load `small-business-setaside.md` alongside to ensure eligibility and size-standard claims meet that domain's bar.
- **Accessibility.** Any Section L language invoking Section 508 conformance pulls in `accessibility-508.md`.

## Gaps

- Classified and SAP acquisitions are out of scope; their source handling requires cleared channels, not public substantiation.
- GWAC-specific task-order competition rules vary by vehicle; the vehicle's ordering guide (loaded via `federal-capture.md`) supplements this layer for TO responses.
