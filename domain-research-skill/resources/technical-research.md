---
name: technical-research
description: Source taxonomy for AI/ML, infrastructure, and software engineering research where claims must trace to primary sources with version, commit, or hardware specificity.
domain: technical-research
contract-version: 2026-04-18
retrieved: 2026-04-19
primary_sources:
  - https://arxiv.org/
  - https://www.rfc-editor.org/
  - https://www.w3.org/TR/
  - https://nvd.nist.gov/
  - https://huggingface.co/
  - https://papers.withcode.com/
---

# Technical Research — Source Taxonomy

Layer for AI/ML, infrastructure, and software engineering research. Applies when the deliverable requires claims about model behavior, protocol specifications, algorithm complexity, library behavior at a specific version, or measured performance.

## Activation Signals

- User references an arXiv ID, RFC number, W3C Recommendation, or CVE.
- User quotes a benchmark score, accelerator spec, library API behavior, or model card claim.
- User asks for a survey of techniques that must distinguish published work from vendor marketing.
- User is evaluating a library upgrade path across versions and needs vendor changelogs plus deprecation notices.

<Source_Domain role="technical-research">

<Tier_1_Primary>
- Peer-reviewed conference and journal papers (NeurIPS, ICML, ICLR, USENIX, OSDI, SOSP, SIGCOMM, SIGMOD, etc.) cited with DOI or venue proceedings URL.
- arXiv preprints cited with arXiv ID and version (e.g., `arXiv:2403.12345v2`); flagged as preprint when not yet venue-accepted.
- Provider model cards on HuggingFace, GitHub, or the provider's domain, pinned to commit or published revision.
- RFCs (IETF) at a specific RFC number; obsoleted RFCs flagged with successor.
- W3C Recommendations at a specific version (e.g., `Candidate Recommendation Snapshot 2024-05-16` vs `Proposed Recommendation 2024-10-01`).
- NVD entries for CVEs cited with CVE ID, CVSS vector, and affected CPE.
- Source code cited by repository URL plus commit SHA or signed tag — not by `main`.
- Official vendor technical documentation pinned to a version/release (e.g., `AWS SDK for Python docs for boto3 1.35.x`).
- Provider release notes, changelogs, and deprecation notices tied to a dated release.
- Reproducible benchmarks published with eval-harness name, commit hash, hardware SKU, and seeds (MLPerf submissions, LM Evaluation Harness runs, SWE-Bench submissions).
- Standards bodies' normative documents (ISO, IEEE, NIST SP series) cited by document number and revision (e.g., `NIST SP 800-53 Rev. 5`).
</Tier_1_Primary>

<Tier_2_Contextual>
- Senior practitioner blogs with a documented citation trail (Simon Willison, Chip Huyen, Lilian Weng, etc.).
- Framework maintainer commentary on issues/PRs in the project's own repo.
- Conference talk summaries and trip reports (frame; direct the reader to the proceedings).
- Reputable independent analysis outlets (Chips and Cheese, ServeTheHome, PortSwigger writeups).
- Well-known Substacks with a reliable citation pattern.
- Academic blogs by recognized authors (Stanford AI Lab, BAIR, etc.) when they summarize their own work.
</Tier_2_Contextual>

<Disqualified>
- Medium, dev.to, and comparable posts that paraphrase papers without citing them.
- "Top N techniques / models / frameworks" SEO listicles.
- AI-generated review sites that aggregate without observing.
- Vendor marketing presenting cherry-picked benchmark numbers without harness or date.
- Undated benchmark results or charts without axis labels and hardware specs.
- Stack Overflow answers without version context, especially older than current minus three major versions.
- Content farms reposting provider release notes.
- "State of X in 2025" reports from organizations whose primary output is the report (lead-generation content).
</Disqualified>

<Evidence_Threshold>
- Performance / capability claim: primary benchmark with harness name and commit, hardware SKU, date, seeds, and prompt/input template. Provider-reported numbers without a reproduction recipe are flagged as vendor-reported, not substantiated.
- Architecture / algorithmic claim: paper DOI or preprint arXiv ID with version; model card when the claim is about a specific release.
- API behavior claim: vendor docs pinned to API version or SDK release; changelog entry where behavior changed.
- Security claim: CVE ID with NVD entry plus vendor advisory; CVSS vector version stated.
- Standards conformance claim: RFC / W3C document with section anchor; ISO / NIST SP citation with revision number.
- Library behavior claim: source code citation by commit SHA plus the test that exercises the behavior, not just the README.
- Currency requirement: re-verify claims against provider release notes whenever the provider ships a new version that touches the claimed surface; flag claims whose retrieval date is older than the most recent provider release.
</Evidence_Threshold>

</Source_Domain>

## Gaps

- Reproducibility evidence varies widely across ML subfields. Where a claim rests on reproducibility but no independent reproduction exists, flag it explicitly rather than treating provider-reported numbers as Tier 1.
- Hardware-performance claims age quickly. Accelerator generations ship faster than this skill's re-verification cadence; cross-check against the current generation on vendor spec pages before relying on older numbers.
