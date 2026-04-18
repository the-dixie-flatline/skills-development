---
name: nonprofit-sector
description: Source taxonomy for nonprofit-sector research — IRS Form 990s, 990-PFs, state charity registrations, foundation disclosures, grant databases, Single Audits.
domain: nonprofit-sector
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://apps.irs.gov/app/eos/
  - https://projects.propublica.org/nonprofits/
  - https://candid.org/
---

# Nonprofit Sector Research — Source Taxonomy

Layer for research on 501(c) nonprofits, private foundations, and grantmakers. Applies to donor research, grant prospecting, board-governance due diligence, financial profiling, and mission-alignment analysis.

## Activation Signals

- User is pulling 990 / 990-PF / 990-EZ / 990-N data on a nonprofit or foundation.
- User is researching grantmakers via Candid (GuideStar / Foundation Directory), ProPublica Nonprofit Explorer, or Instrumentl.
- User is verifying state charity registration status or drafting a state charitable-solicitation filing.
- User is reading an A-133 / Uniform Guidance Single Audit for a federal-grant-receiving nonprofit.

<Source_Domain role="nonprofit-sector">

<Tier_1_Primary>
- IRS Form 990, 990-PF, 990-EZ, 990-T, and 990-N records. Accessed through:
  - IRS Tax Exempt Organization Search (apps.irs.gov/app/eos) for current 990s and determination letters.
  - Candid (candid.org, formerly GuideStar) for historical 990 archives, Foundation Directory grants data, and Foundation Center holdings.
  - ProPublica Nonprofit Explorer (projects.propublica.org/nonprofits) for machine-readable 990 fields and historical filings.
  - Each 990 cited by EIN, tax year, and form type (e.g., `EIN 12-3456789, Form 990-PF, tax year 2023`).
- IRS determination letters granting 501(c)(3) or other 501(c) status, cited with issue date.
- State charity registrations:
  - New York State Charities Bureau (charitiesnys.com) registration and CHAR500 filings.
  - California Registry of Charitable Trusts (oag.ca.gov/charities) with RCT number.
  - Other state charity bureaus per state (Texas OAG Charitable Trusts, Illinois AG Charitable Trust Bureau, etc.); the Unified Registration Statement applies in ~40 states with supplements.
- Single Audit reports under the Uniform Guidance (2 CFR Part 200 Subpart F) for nonprofits expending more than $750,000 in federal awards, filed with the Federal Audit Clearinghouse (facdissem.census.gov; note FAC transitioned operators in 2023).
- Audited financial statements posted by the nonprofit itself (primary when posted to the nonprofit's own domain or a state-required disclosure portal).
- Foundation grant databases with grantmaker-filed data: Candid's Foundation Directory, Instrumentl, Philanthropy News Digest grants announcements (tied back to the grantmaker).
- Nonprofit IRS Forms 1023 / 1023-EZ / 1024 application materials when available.
- State-level grant / solicitation registrations (California AG Charity Solicitation records, etc.).
- Board meeting minutes where publicly posted (primary for the specific decisions recorded).
- IRS Publication 78 data through the Tax Exempt Organization Search (tax-deductible-contribution eligibility).
</Tier_1_Primary>

<Tier_2_Contextual>
- Chronicle of Philanthropy, Inside Philanthropy, NonProfit Quarterly, Stanford Social Innovation Review.
- Urban Institute / National Center for Charitable Statistics (NCCS) research (primary when citing NCCS's own produced analyses; secondary when they summarize 990 data).
- Foundation strategic plans and annual reports (frame funder priorities; grant-level substantiation still requires the 990-PF or foundation grants database).
- Academic studies of nonprofit-sector dynamics.
- GuideStar Seals of Transparency (profile-completeness signal, not a financial substantiation).
</Tier_2_Contextual>

<Disqualified>
- Fundraising-consultant blog posts asserting nonprofit financials without 990 citations.
- AI-written "top charity" rankings.
- Crowdfunding aggregator summaries.
- Vendor lead-generation content targeting nonprofit tech buyers.
- Outdated 990 data presented as current (990s are filed on tax-year lag; a "current" claim must cite the most recent filed tax year).
- Charity-navigator-style composite ratings treated as primary for financial claims (the underlying 990 is the primary).
- "Impact" statements absent 990 Part III or current program-service documentation.
</Disqualified>

<Evidence_Threshold>
- Financial claim (revenue, expenses, net assets, officer compensation): specific 990 tax year with Part and Line reference (e.g., `Form 990 TY2023 Part I Line 12 — Total revenue` or `Schedule J Part II for officer compensation`).
- Grant-awarded claim: 990-PF Part XIV grants listing for the tax year, or the foundation's published grants database entry.
- Mission / program claim: 990 Part III program-service accomplishments, Part III narrative, or current audited financial statements.
- Leadership / governance claim: 990 Part VI (governance) and Part VII (officers, directors, key employees) with filing date.
- Tax-exempt status claim: IRS Tax Exempt Organization Search entry with the determination-letter issue date.
- Federal-fund claim: Single Audit report on the Federal Audit Clearinghouse, cited with audit period and FAC accession data.
- Currency requirement: 990s are filed annually on 4½-month to 11-month lag from tax year end (automatic-extension rules under IRS Form 8868 push many filings into year-plus lag). Any "current year" financial claim on a nonprofit is flagged with the specific tax year of the underlying 990. State charity registrations typically require annual renewal; cite registration status with the most recent renewal date.
</Evidence_Threshold>

</Source_Domain>

## Gaps

- Church-classified entities (filings under 501(c)(3) status but classified as a church under IRC § 170(b)(1)(A)(i)) are generally exempt from 990 filing; financial research on them has to rely on voluntary disclosure.
- Donor-advised fund sponsor data is reportable at the sponsor level, not the individual DAF; downstream granting is visible only at the DAF sponsor's 990, not the individual donor.
- Fiscal-sponsor relationships can obscure the operating entity; verify the fiscal sponsor's 990 when the operating nonprofit does not file its own.
