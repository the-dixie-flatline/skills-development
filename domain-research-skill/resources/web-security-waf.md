---
name: web-security-waf
description: Source taxonomy for web security and WAF configuration — CVEs, NVD, OWASP, CISA KEV, WAF vendor rule behavior, CRS, MITRE ATT&CK, vendor advisories.
domain: web-security-waf
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://nvd.nist.gov/
  - https://www.cve.org/
  - https://owasp.org/
  - https://owasp.org/Top10/2025/
  - https://www.cisa.gov/known-exploited-vulnerabilities-catalog
  - https://coreruleset.org/
---

# Web Security & WAF Configuration — Source Taxonomy

Layer for vulnerability research, WAF rule authoring and tuning, security control mapping, and vendor-advisory triage. Every CVE claim, rule claim, and exploit-likelihood claim must trace to a primary record.

## Activation Signals

- User is triaging a CVE, CVSS score, or vendor advisory.
- User is authoring or tuning WAF rules (AWS WAF, Cloudflare, Akamai, Fastly, Imperva, F5, Azure Front Door, ModSecurity CRS).
- User is mapping controls to OWASP Top 10, ASVS, or MITRE ATT&CK.
- User is responding to a penetration-test finding, coordinated disclosure, or bug-bounty report.
- User is reviewing a CISA KEV-listed vulnerability in scope of an organization's exposure.

<Source_Domain role="web-security-waf">

<Tier_1_Primary>
- CVE Program records at cve.org (MITRE) and NIST NVD (nvd.nist.gov) — cited with CVE ID, CVSS vector (state CVSS v2 / v3.1 / v4.0), affected CPE, and publication / modification dates.
- CISA Known Exploited Vulnerabilities catalog (cisa.gov/known-exploited-vulnerabilities-catalog) — primary for "exploited in the wild" assertion, cited with the KEV addition date and remediation-due date for FCEB agencies.
- Vendor security advisories: MSRC (microsoft.com/msrc), RHSA (access.redhat.com/security), USN (ubuntu.com/security), Apache Security (apache.org/security), nginx (nginx.org/en/security_advisories), OpenSSL (openssl.org/news/vulnerabilities), Cisco PSIRT (sec.cloudapps.cisco.com), Oracle CPU (oracle.com/security-alerts), VMware VMSA, Fortinet PSIRT, Palo Alto Networks security advisories — correlated to CVE IDs.
- OWASP Foundation materials (owasp.org):
  - OWASP Top 10 Web Application Security Risks (current edition: **2025**), cited by year and category ID. Current categories: A01:2025 Broken Access Control, A02:2025 Security Misconfiguration, A03:2025 Software Supply Chain Failures, A04:2025 Cryptographic Failures, A05:2025 Injection, A06:2025 Insecure Design, A07:2025 Authentication Failures, A08:2025 Software or Data Integrity Failures, A09:2025 Security Logging and Alerting Failures, A10:2025 Mishandling of Exceptional Conditions. [source: https://owasp.org/Top10/2025/, retrieved 2026-04-19]
  - OWASP API Security Top 10 (2023).
  - OWASP Application Security Verification Standard (ASVS 4.0.3 / 5.0 in progress), cited by version and control ID (e.g., `ASVS 4.0.3 V5.3.3`).
  - OWASP Web Security Testing Guide (WSTG) cited by version and test ID (e.g., `WSTG 4.2 WSTG-INPV-01`).
  - OWASP Cheat Sheet Series, cited by cheat-sheet name and revision.
- CWE (cwe.mitre.org) — primary for weakness taxonomy; cited by CWE ID.
- MITRE ATT&CK framework (attack.mitre.org) — cited with technique ID (e.g., `T1190 Exploit Public-Facing Application`) and the matrix version.
- CAPEC (Common Attack Pattern Enumeration and Classification) — cited with CAPEC ID.
- ModSecurity Core Rule Set (CRS) documentation (coreruleset.org) — cited with CRS version (e.g., CRS 4.x) and rule ID (e.g., `942100 SQL Injection Attack Detected via libinjection`).
- WAF vendor rule documentation:
  - AWS WAF managed rule groups (docs.aws.amazon.com/waf/).
  - Cloudflare Managed Rulesets and WAF documentation (developers.cloudflare.com/waf/).
  - Akamai App & API Protector / Kona Site Defender docs (techdocs.akamai.com).
  - Fastly Next-Gen WAF (Signal Sciences) docs (docs.fastly.com).
  - Imperva Cloud WAF docs (docs.imperva.com).
  - F5 Advanced WAF / NGINX App Protect.
  - Azure Web Application Firewall managed rules (learn.microsoft.com).
- Standards and normative references:
  - TLS 1.3 (RFC 8446), HTTP/2 (RFC 9113), HTTP/3 (RFC 9114), HTTP Semantics (RFC 9110), Caching (RFC 9111).
  - CORS Fetch Standard (fetch.spec.whatwg.org).
  - Content Security Policy Level 3 (w3.org/TR/CSP3/).
- FIRST.org CVSS v3.1 and v4.0 specification documents.
- CISA advisories and alerts (cisa.gov/news-events/cybersecurity-advisories) including joint-sealed advisories.
- Vendor PSIRT disclosures published through their own advisory channels.
</Tier_1_Primary>

<Tier_2_Contextual>
- Security research from credentialed teams: Project Zero (googleprojectzero.blogspot.com), Rapid7 research, Trail of Bits, NCC Group, PortSwigger Research, Snyk Labs, Wiz research — primary for the research artifact itself (detailed analysis of a bug), contextual when the claim is about CVE details (defer to NVD / vendor advisory).
- Academic conference proceedings (USENIX Security, CCS, IEEE S&P, NDSS, WOOT).
- HackerOne / Bugcrowd disclosed reports (hackerone.com/reports) — primary for the specific finding; always disclosed with the reporter's consent.
- Krebs on Security, The Register's security desk, Ars Technica security coverage — frame incident context; primary claims go to vendor / CISA.
- Vendor engineering blogs describing rule behavior post-hoc (frame tuning patterns).
</Tier_2_Contextual>

<Disqualified>
- AI-written CVE summaries that paraphrase NVD without adding analysis or verification.
- SEO posts titled "Top 10 web vulnerabilities 2024" with no OWASP grounding.
- Vendor marketing claiming "100% WAF protection" or similar absolutes.
- Undated "common exploits" listicles.
- WAF-rule claims not traceable to a vendor rule ID, CRS rule ID, or a documented regex pattern.
- Blog posts asserting active exploitation absent CISA KEV or a named-vendor advisory.
- "Threat intelligence" feeds without attribution to a primary publisher.
- CVE claims that cite only the CVE ID without the CVSS vector and affected CPE.
</Disqualified>

<Evidence_Threshold>
- Vulnerability claim: CVE ID with NVD entry, CVSS vector (version stated; CVSS v4.0 preferred for claims analyzed after 2023-11 when v4.0 went GA, v3.1 retained for historical entries not re-scored), affected CPE, and publication date.
- Exploit-in-the-wild claim: CISA KEV entry with addition date, or a named-vendor advisory explicitly stating active exploitation.
- Patch-availability claim: vendor advisory with patched version and release date.
- WAF-behavior claim: vendor rule reference (by managed-rule-group name and rule ID, or CRS rule ID with CRS version). A regex-only claim without a rule-ID anchor is flagged.
- False-positive claim: reproduction artifact (request/response), CRS GitHub issue or vendor case number, the rule ID causing the block, and the proposed tuning (exclusion, scoring adjustment, anomaly-threshold change).
- Control-mapping claim: OWASP ASVS / WSTG / Cheat Sheet citation, CWE / CAPEC IDs, or MITRE ATT&CK technique ID with matrix version.
- Protocol-behavior claim: IETF RFC number and section, or WHATWG Standard section.
- Currency requirement: CVE records revise as new data appears (CVSS rescoring, CPE additions, NVD enrichment lag); cite retrieval date. CISA KEV updates continuously. WAF managed rule groups version continuously; record the managed-rule-group version and last-updated date when asserting rule-set behavior. OWASP Top 10 revises roughly every 3–4 years; state the edition year — the **2025 edition** is current and supersedes 2021 (several categories renumbered, A03:2025 "Software Supply Chain Failures" and A10:2025 "Mishandling of Exceptional Conditions" are new category framings).
</Evidence_Threshold>

</Source_Domain>

## Gaps

- NVD analysis lag has been documented; recent CVEs may have sparse CVSS / CPE data. Where NVD is incomplete, cite the CNA's own advisory and flag the NVD gap rather than inferring.
- Some vendor advisories are paywalled or require account access; cite them and note the access gate rather than substituting a secondary.
- Zero-day claims without a CVE are flagged as unattributed until a CVE is assigned.
