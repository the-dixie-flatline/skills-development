---
name: salesforce-architecture
description: Source taxonomy for Salesforce integration architecture — release-versioned platform docs, Trust status, governor limits, API versioning, AppExchange security-review artifacts.
domain: salesforce-architecture
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://developer.salesforce.com/
  - https://trust.salesforce.com/
  - https://help.salesforce.com/s/releasenotes
  - https://architect.salesforce.com/
---

# Salesforce Integration Architecture — Source Taxonomy

Layer for architectural work on the Salesforce platform: Apex, Flow, Lightning, Platform Events, CDC, Bulk API, Streaming API, External Services, AppExchange integration, sandbox strategy, and Well-Architected assessments. Every bare claim about platform behavior is version-scoped.

## Activation Signals

- User is writing or reviewing Apex, Flow, Lightning Web Components, or Aura at a specific release (Spring, Summer, Winter YY).
- User is evaluating governor limits, API call limits, or data-storage quotas.
- User is integrating against SOAP API / REST API / Bulk API 2.0 / Streaming API / Pub/Sub API / Composite API at a specific API version.
- User is running AppExchange Security Review or preparing ISV technical artifacts.
- User is diagnosing platform status, outages, or known issues.

<Source_Domain role="salesforce-architecture">

<Tier_1_Primary>
- Salesforce Developer Documentation (developer.salesforce.com/docs) pinned to API version and release. URL parameters `?version=` and release labels (Spring YY / Summer YY / Winter YY) must be recorded.
- Apex Developer Guide at release version (for governor-limit tables and language semantics).
- Salesforce Release Notes (help.salesforce.com/s/releasenotes) at Spring / Summer / Winter YY.
- API version lifecycle notice — Salesforce deprecates older API versions on a published schedule; the retired-versions list is authoritative for what is still callable.
- Salesforce Trust (trust.salesforce.com) — instance status, incident history by instance (NA, EU, JP, AP), maintenance announcements. Primary for availability / incident claims.
- Salesforce Known Issues site for known-issue IDs and workaround status.
- Architect.salesforce.com — Well-Architected framework and decision guides (primary for Salesforce-published architectural guidance).
- AppExchange Security Review documentation: the ISVforce Guide and the Security Review Requirements checklist published by Salesforce partner/security teams.
- Salesforce Trailhead units labeled as official documentation supplements (primary when they reproduce the docs; otherwise Tier 2).
- sforce-ci-cd published materials, sfdx / sf CLI release notes on GitHub (github.com/forcedotcom/cli).
- Dreamforce session recordings and Salesforce engineering blog posts where the content is published by Salesforce directly under developer.salesforce.com or the Salesforce Engineering blog.
- Einstein / Data Cloud / MuleSoft-integrated capabilities: documentation on the respective product's `.salesforce.com` domain with the release tag.
</Tier_1_Primary>

<Tier_2_Contextual>
- Salesforce Ben, SFBlogger, Trailhead community content — frame practitioner experience; not platform authority.
- Certified Technical Architect (CTA) and Salesforce MVP blog posts (well-known individuals: Anna Loughnan, Pablo Gonzalez, Peter Knolle, etc.) — frame patterns; verify specifics against the docs.
- Stack Exchange (Salesforce Stack Exchange) answers within the current-release-minus-one window and authored by users with a documented track record.
- Conference talks (Dreamforce, TrailblazerDX) where the slides/recording are not republished on Salesforce-owned infrastructure.
</Tier_2_Contextual>

<Disqualified>
- Tutorial sites mixing Apex examples from multiple API versions without specifying which is current.
- AI-generated Apex / Flow / LWC samples without version context.
- Third-party integration-tooling marketing content paraphrasing governor limits.
- Stack Overflow answers older than current minus three releases without re-verification.
- "Salesforce tips" content farms.
- YouTube channels summarizing Release Notes without citing specific release sections.
- Outdated blog posts asserting deprecated behaviors (e.g., Process Builder, Workflow Rules, deprecated Apex methods) as current.
</Disqualified>

<Evidence_Threshold>
- Platform-behavior claim: Developer Documentation URL with API version and release label, or Release Notes citation for the release that introduced / changed the behavior.
- Governor-limit claim: Apex Developer Guide section with the release version; always pair with whether the limit is per-transaction, per-24-hour, or org-wide.
- API-call-limit claim: the specific limit type (daily API call limit, concurrent Apex request limit, event-delivery limit) with the docs section and the license edition it applies to.
- Outage / availability claim: trust.salesforce.com record with incident ID, affected instance(s), and date.
- Known-issue claim: Known Issues site entry with the W-number (internal Salesforce work-item ID), reported date, and current status.
- AppExchange package claim: listing page on appexchange.salesforce.com with the package version ID and the Security Review date (passed / re-submitted).
- CLI / tooling behavior claim: sf CLI release notes with version tag from github.com/forcedotcom/cli.
- Currency requirement: Salesforce ships three major releases per year (Spring / Summer / Winter). Every bare platform claim pins to a release; claims more than two releases old are re-verified. API versions are retired on published schedules; claims about older API versions flag the retirement status. Governor limits change between releases; always cite the release the claim was verified against.
</Evidence_Threshold>

</Source_Domain>

## Gaps

- Salesforce edition differences (Essentials, Professional, Enterprise, Unlimited, Developer) gate many features and limits; every claim should note the edition scope. Edition-specific details are in the Sales Cloud / Service Cloud / Platform feature guides.
- Hyperforce-vs-legacy-instance differences (data residency, networking, certain API behaviors) need to be specified; the user's org's infrastructure posture on trust.salesforce.com determines which rules apply.
- Government Cloud, Financial Services Cloud, Health Cloud, and Net Zero Cloud have their own compliance and feature overlays; claims on those products cite the vertical cloud's documentation, not the general platform.
