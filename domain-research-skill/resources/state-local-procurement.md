---
name: state-local-procurement
description: Source taxonomy for state and local government procurement — state procurement portals, cooperative purchasing vehicles, state master contracts, state small-business designations. Fragmented per jurisdiction.
domain: state-local-procurement
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://www.naspovaluepoint.org/
  - https://sam.gov/
  - https://www.nigp.org/
---

# State & Local Government Procurement — Source Taxonomy

Layer for state, county, and municipal procurement work. The landscape is fragmented; every jurisdiction maintains its own portal, its own procurement code, and its own small-business designations. Cooperative purchasing vehicles partially unify access, but a vehicle is only usable by an entity that has signed on or adopted it.

## Activation Signals

- User is responding to a state, county, city, or special-district solicitation.
- User is evaluating participation on a cooperative purchasing vehicle (NASPO ValuePoint, Sourcewell, OMNIA Partners, TIPS, BuyBoard, regional consortia).
- User is registering on, or researching solicitations via, a state procurement portal (eVA, Cal eProcure, NYS OGS, Texas SmartBuy, COMMBUYS, BidBuy, etc.).
- User is evaluating a state-level small-business or disadvantaged-business designation (e.g., California DVBE, Texas HUB, New York MWBE, Illinois BEP, Virginia SWaM).

<Source_Domain role="state-local-procurement">

<Tier_1_Primary>
- State procurement portals, with solicitations cited by the state's bid / solicitation number and the portal URL. Representative portals:
  - Virginia eVA (eva.virginia.gov)
  - California Cal eProcure (caleprocure.ca.gov) and state DGS master agreements (dgs.ca.gov)
  - New York OGS (ogs.ny.gov) and the NYS Contract Reporter
  - Texas SmartBuy (txsmartbuy.gov) and the Texas CPA Statewide Procurement Division
  - Massachusetts COMMBUYS (commbuys.com) and the Operational Services Division
  - Illinois Procurement Bulletin and BidBuy (illinois.gov/cpo)
  - Florida MyFloridaMarketPlace (myfloridamarketplace.com)
  - Georgia Procurement Registry (doas.ga.gov)
  - Washington WEBS (des.wa.gov)
  - Ohio Procurement Portal and OhioBuys (ohiobuys.ohio.gov)
- State procurement codes and administrative rules: state-specific citations (e.g., California Public Contract Code, Texas Government Code Title 10 Chapter 2155, Virginia Code § 2.2-4300 et seq., NY State Finance Law § 163, MA c.7 and c.30B).
- State Office of the Inspector General or state auditor procurement audit reports.
- Cooperative purchasing master agreements, cited with the cooperative's master contract number and the participating-entity / adopting-entity roster:
  - NASPO ValuePoint master agreements (naspovaluepoint.org), with the lead state and adopting states.
  - Sourcewell (sourcewell-mn.gov) contracts.
  - OMNIA Partners public-sector contracts (omniapartners.com).
  - TIPS (The Interlocal Purchasing System, tips-usa.com).
  - BuyBoard (buyboard.com) and state-specific buy-board cooperatives.
  - GSA Cooperative Purchasing under Section 1122 Program for state and local entities, where applicable, cited with GSA documentation.
- City and county procurement ordinances published on the jurisdiction's `.gov` or `.us` domain.
- State supplier registration records (DSBS-equivalents vary; each state has its own).
- State small-business / disadvantaged-business program regulations and rosters (e.g., California DVBE at DGS OSDS; Texas HUB at CPA Statewide HUB Program; NY ESD MWBE; Illinois CMS BEP; Virginia SBSD SWaM).
- Disadvantaged Business Enterprise (DBE) certification records through 49 CFR Part 26 for USDOT-funded state and local work, cited through the state's Unified Certification Program.
- NIGP (nigp.org) public-procurement code dictionary and model code (NIGP codes are used by many jurisdictions for classification; primary when the jurisdiction has adopted them).
</Tier_1_Primary>

<Tier_2_Contextual>
- NASCIO / NASPO / NIGP reports on state procurement trends.
- State auditor performance reports (frame; primary when the report itself is the authoritative finding).
- Industry coverage: GovTech, StateScoop, Route Fifty, Governing.
- Bid-aggregation platforms: BidNet, Periscope S2G / BidSync, Onvia / DemandStar, Public Purchase — useful for scouting and routing, not substantiation.
- Cooperative-purchasing marketing materials (frame only; the master agreement is the primary).
</Tier_2_Contextual>

<Disqualified>
- Lead-generation sites aggregating "government RFPs" without linking the originating portal solicitation.
- AI-written vendor overviews of state procurement programs.
- Undated "how to sell to state governments" content.
- Marketing that conflates a cooperative contract's availability with a given entity's authority to buy from it (authority depends on the entity having adopted / joined).
- Press releases claiming a cooperative award without the master contract number.
- State-program content that cites a superseded statute or administrative rule without flagging the revision.
</Disqualified>

<Evidence_Threshold>
- Solicitation claim: state portal URL, the state's bid/solicitation number, posting date, and response due date.
- Cooperative contract claim: master contract number on the cooperative's own site, plus the participating / adopting roster confirmation for the specific buying entity.
- State statute claim: state code citation with chapter / section and the most recent revision date (state codes change on legislative sessions; many ship revisions annually).
- Administrative-rule claim: state agency's administrative-rule citation with the rule's effective date (many states publish rules through an Office of Administrative Rules / equivalent register).
- State-level small-business designation claim: the state program's roster entry with certification or verification date.
- DBE claim: state UCP directory entry with certification date and NAICS codes covered.
- Currency requirement: state procurement codes and administrative rules update on legislative / rulemaking cycles; re-verify against the state's current code on each reading. Cooperative-contract rosters of adopting entities change continuously; verify adoption for the specific entity, not the cooperative in the abstract.
</Evidence_Threshold>

</Source_Domain>

## Multi-Domain Overlap Notes

- **Federal capture.** GSA Cooperative Purchasing Section 1122 work straddles both domains; load both when that bridge is in play.
- **Small business set-aside.** SBA programs are federal; state small-business programs are covered here. When a firm is pursuing both federal set-asides and state SB designations, both domains activate.
- **Accessibility.** State procurement often references WCAG conformance or state-specific accessibility rules (California AB 434, Minnesota Statute 16E.03); load `accessibility-508.md` alongside when those apply.

## Gaps

- Municipal-level procurement data is deeply fragmented; some cities publish structured datasets, many do not. Where a city does not publish, flag the gap rather than extrapolate from neighbors.
- Cooperative-purchasing eligibility rules vary; some cooperatives are open to any public entity, others only to members or adopting entities under specific interlocal-agreement statutes.
