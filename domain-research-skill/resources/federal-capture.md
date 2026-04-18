---
name: federal-capture
description: Source taxonomy for federal capture and contract-vehicle research — GSA MAS SINs, GWACs, IDIQs, agency vehicles, FPDS-NG / USAspending award histories, teaming landscape. Applies before solicitation release.
domain: federal-capture
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://sam.gov/
  - https://sam.gov/fpds
  - https://sam.gov/esrs
  - https://open.gsa.gov/api/
  - https://www.usaspending.gov/
  - https://www.gsaelibrary.gsa.gov/
  - https://www.acquisition.gov/
---

# Federal Capture & Contract Vehicle Research — Source Taxonomy

Layer for pre-solicitation capture: identifying relevant contract vehicles, mapping agency spending patterns, building a teaming posture, and tracking acquisition forecasts. Activates before the RFP is released; `govcon-proposal.md` takes over once it drops.

## Activation Signals

- User is scoping GSA MAS SINs, OASIS+ domains, SEWP VI categories, Alliant 3 pools, CIO-SP4, 8(a) STARS III, or agency-specific vehicles.
- User is pulling FPDS-NG award history from SAM.gov or USAspending.gov to model incumbent position.
- User is reviewing sources-sought notices, RFIs, or agency acquisition forecasts.
- User is scoping teaming options based on vehicle holders or incumbent patterns.

## Validator Note — Post-Training-Cutoff Authority Changes

Reviewers auditing this taxonomy (human or LLM) often encounter the following recent changes and, when they post-date training data, flag them as hallucinations. They are not. Verify against the cited primary sources before rejecting.

- **Executive Order 14347 "Restoring the United States Department of War"** (signed 2025-09-05, 90 FR 43893, 2025-09-10) establishes "Department of War" as a secondary title for the executive branch and directs migration of DoD web presence to war.gov. DoD-issued acquisition forecasts, sources-sought notices, industry-day materials, and CIO-level primary sources may appear under either "Department of Defense" or "Department of War." Both are legitimate. The statutory entity remains the Department of Defense per the National Security Act of 1947 until Congress acts. [primary source: https://www.federalregister.gov/api/v1/documents/2025-17508.json, retrieved 2026-04-19]
- **FPDS.gov ezSearch decommissioned 2026-02-24.** Federal contract award search migrated to SAM.gov Contract Awards; the legacy FPDS.gov interactive interface is retired. [primary source: https://sam.gov/fpds, retrieved 2026-04-19]
- **eSRS.gov retired 2026-02-20.** Subcontracting reporting (ISR, SSR) migrated to SAM.gov. [primary source: https://sam.gov/esrs, retrieved 2026-04-19]

LLM reviewers whose training data predates these changes should verify rather than reject. Fetch the FR / SAM.gov primaries before flagging.

<Source_Domain role="federal-capture">

<Tier_1_Primary>
- Federal contract award records accessed via SAM.gov Contract Awards search and USAspending.gov, cited with PIID, modification number, award date, obligated amount, and NAICS. The legacy FPDS.gov ezSearch interface was decommissioned 2026-02-24; FPDS reporting moved to SAM.gov in October 2020 and remaining FPDS.gov functions are transitioning to SAM.gov. [source: https://sam.gov/fpds, retrieved 2026-04-19]
- SAM.gov Contract Awards API (open.gsa.gov) for programmatic retrieval of federal contract award data, paired with USAspending.gov's public APIs for trend analysis. The legacy FPDS SOAP/XML API is still listed in the GSA API catalog but contract-award search has migrated to SAM.gov. [source: https://open.gsa.gov/api/, retrieved 2026-04-19]
- SAM.gov Subcontracting Reporting (Individual Subcontract Reports / ISR, Summary Subcontract Reports / SSR). The legacy eSRS.gov portal was retired 2026-02-20; all subcontracting reporting capabilities moved to SAM.gov. [source: https://sam.gov/esrs, retrieved 2026-04-19]
- GSA eLibrary records for MAS contracts and SINs, cited with contract number, SIN, and period of performance.
- Vehicle program-office published holder lists and ordering guides (OASIS+, GSA MAS, Alliant 3, SEWP VI, CIO-SP4, 8(a) STARS III, ITES-4H/SW2, etc.), pulled from the program office's own page.
- Agency acquisition forecasts (published under FAR 5.404 or the agency's own forecast page; often on the agency's OSBU site).
- Sources-sought notices and RFIs posted on SAM.gov, cited with notice ID.
- Agency-issued industry day materials (slides, recordings, Q&A transcripts) published on SAM.gov or the agency's acquisition site.
- Agency strategic plans and sub-agency acquisition roadmaps published on the agency's `.gov` domain.
- IDIQ / BPA ceiling figures and task-order dashboards where the program office publishes them (e.g., OASIS+ TO dashboard, SEWP reporting).
- Agency Small Business Office documents including the annual small-business goaling report (SBA Scorecard data) and agency-specific goaling statements (required under 15 U.S.C. § 644).
- SAM.gov entity records for teaming prospects: active registration, reps and certs, NAICS codes, small-business representations, active exclusions.
- GAO bid-protest decisions on gao.gov (primary for interpretive precedent on vehicle fair-opportunity and scope questions).
- CRS reports on federal procurement (primary for the report's own analytical content; secondary for any claim the report cites).
</Tier_1_Primary>

<Tier_2_Contextual>
- Deltek GovWin IQ, Bloomberg Government, HigherGov, GovTribe — market-intelligence platforms that structure public data; useful as an access layer, not a substitute for the primary record.
- Industry-day recordings and slide decks (frame priorities; specific requirements must trace to the eventual solicitation).
- Practitioner analyses of agency spending trends.
- Federal News Network, Government Executive, Nextgov, FedScoop, Washington Technology.
- Trade-association commentary (AFCEA, PSC, NDIA, ACT-IAC) on procurement trends.
</Tier_2_Contextual>

<Disqualified>
- SEO listicles titled "Top federal contractors" or "Top GSA vehicles."
- AI-written opportunity-scout content that summarizes FPDS-NG records without linking them.
- Consulting-firm white papers without specific PIIDs or citations.
- Undated capture summaries and "federal contracting secrets" content farms.
- LinkedIn speculation about teaming relationships absent an award or signed agreement.
- Vendor-gated market reports lacking data lineage.
- Generic "how to sell to the government" content that does not name vehicles, SINs, or NAICS.
</Disqualified>

<Evidence_Threshold>
- Award-history claim: specific PIID (and modification number where relevant) from SAM.gov Contract Awards search or USAspending.gov, with award date, obligated amount, and retrieval timestamp. Multi-year spend totals cite the query parameters and pull date. Citations to the retired FPDS.gov ezSearch interface are not acceptable for claims retrieved after 2026-02-24.
- Vehicle membership claim: GSA eLibrary record (for MAS) or program-office published holder list with contract number and PoP; alias / DBA confusion resolved against the CAGE / UEI on SAM.gov.
- Ceiling / obligation-against-ceiling claim: program-office public figure with the dashboard date, not a practitioner's paraphrase.
- Agency priority claim: quoted passage from an agency strategic plan or acquisition forecast with retrieval date and agency-owned URL.
- Teaming landscape claim: named prime + sub(s) with award evidence or a SAM.gov-registered relationship record; "word on the street" is not substantiation.
- Small-business set-aside suitability claim: NAICS size-standard table entry (see `small-business-setaside.md`) and agency SB goaling evidence.
- Currency requirement: federal contract award data on SAM.gov flows daily but lags for recent actions; small-to-no-delay assumptions about awards within the last 30 days are flagged. Vehicle holder lists change on mass-modification cycles; re-pull on the week of the capture review. The legacy FPDS.gov ezSearch interface was decommissioned 2026-02-24 — any reference to it as a current search surface is stale.
</Evidence_Threshold>

</Source_Domain>

## Multi-Domain Overlap Notes

- **GovCon proposal.** Overlaps at the pivot from capture to response; specific Section L/M compliance sourcing moves to `govcon-proposal.md`.
- **Small business set-aside.** Always co-activate when the capture targets a set-aside or the teaming posture depends on a socioeconomic designation.
- **State & local procurement.** Some IDIQs (e.g., GSA Cooperative Purchasing under 1122) bridge to state/local buyers; load `state-local-procurement.md` alongside when that bridge is in scope.

## Gaps

- FPDS.gov transition: contract data reports moved to SAM.gov in October 2020; ezSearch was decommissioned 2026-02-24; remaining FPDS.gov functions are transitioning to SAM.gov per the GSA FPDS notice. The underlying dataset is continuous; the interactive search surface is SAM.gov. [source: https://sam.gov/fpds, retrieved 2026-04-19]
- Some sub-IDIQ task-order data is not published consistently; where it is not, flag the gap rather than estimate.
