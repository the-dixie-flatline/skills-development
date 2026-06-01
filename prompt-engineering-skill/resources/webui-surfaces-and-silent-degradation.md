---
family: cross-family
scope: webui-surfaces
families: [openai, anthropic, gemini]
retrieved: 2026-06-01
primary_sources:
  - https://help.openai.com/en/articles/8096356-chatgpt-custom-instructions
  - https://support.claude.com/en/articles/14546867-set-organization-instructions
  - https://support.google.com/docs/answer/16943683
  - https://help.openai.com/en/articles/8313428-does-chatgpt-tell-the-truth
  - https://support.google.com/gemini/answer/16279220
maturity_note: |
  Consumer and web-UI surfaces are not the API. They expose far less inference
  control and behave differently in ways the vendors mostly do not document.
  The single most important fact about this layer is what is NOT publicly
  documented: the default reasoning effort on consumer tiers, whether the UI
  silently substitutes a different model, and consumer output caps. Almost
  everything here beyond five Tier-1 facts is community-reported and stated
  directionally, never as numbers.
---

# Web-UI Surfaces and Silent Degradation — Cross-Family Reference

The lead finding is a gap. The **default reasoning effort on consumer/web-UI tiers is undocumented** across OpenAI, Anthropic, Google, and xAI consumer products. No vendor publishes the effort level a chat UI applies to a turn, whether it varies by subscription tier, or whether it shifts under load. Do not guess a value. For anyone reasoning about why a prompt behaves differently in a web UI than through the API, this unknown is the most consequential one: the same prompt may be served at a different reasoning budget with no signal to the user.

This reference is deliberately thin. It states the few vendor-documented facts as Tier 1, names the failure modes that follow from the API-vs-UI gap, and routes detection methodology elsewhere. It does not contain detection rules.

## What the API >> web-UI gap looks like

Stated directionally only. Web-UI surfaces typically expose far less control than the API. [community-reported]

- No explicit reasoning-effort knob is exposed on most consumer chat surfaces; the API equivalents (`reasoning.effort`, `thinking`, `thinkingConfig`, `effort`) have no consumer-UI analog the user can read or set. [community-reported]
- A consumer UI may silently substitute a different or smaller model than the labeled one, without disclosure. [community-reported]
- Consumer output caps may be smaller than the documented API maximum-output limits. [community-reported]

None of these are stated here as integers, because no Tier-1 source quantifies them for the consumer surface. Treat any specific number sourced elsewhere for these as unverified.

## Tier-1 documented facts

These five are the only hard claims in this reference.

1. **OpenAI custom instructions** — the long-form custom-instruction text fields have a **1,500-character limit**.
[source: help.openai.com/en/articles/8096356-chatgpt-custom-instructions, retrieved 2026-06-01]

2. **Anthropic organization instructions** — this is an organization/admin field, set by Admins, Owners, and Primary Owners on Team and Enterprise plans. It is **not** the individual consumer instruction field. Maximum length is **3,000 characters**.
[source: support.claude.com/en/articles/14546867-set-organization-instructions, retrieved 2026-06-01]

3. **Google Gemini-in-Docs** — you can create up to **1,000 active instructions** for Gemini in Docs. Access to the feature is gated by the admin via Gemini Alpha, so it is not the case that there is "no admin control" — admins do control access. The feature is **US / English-only**.
[source: support.google.com/docs/answer/16943683, retrieved 2026-06-01]

4. **OpenAI on truthfulness** — OpenAI documents that grounded answers may contain **fabricated quotes, studies, citations, or references to non-existent sources**.
[source: help.openai.com/en/articles/8313428-does-chatgpt-tell-the-truth, retrieved 2026-06-01]

5. **Google on self-description** — Google documents that Gemini **can misrepresent how it works, including how it cites sources**.
[source: support.google.com/gemini/answer/16279220, retrieved 2026-06-01]

## Failure modes (named here, detection routed out)

This reference names the failure modes that follow from the UI/grounding behavior above. It does **not** carry detection rules or mitigation methodology. Detection and mitigation for all three live in the `prompt-engineering-architect` skill.

- **Citation conflation** — a claim attributed to a source that does not support it. Vendor-documented for OpenAI (fact 4) and Google (fact 5). Anthropic's consumer citation-fidelity posture is vendor-silent; treat it as a gap, not as evidence of better or worse behavior.

- **Global-prompt / custom-instruction summarization bias** — standing custom or organization instructions can skew how the model summarizes or weights material across turns. Named only; the documented instruction-length limits above (facts 1–3) bound the input but say nothing about its behavioral effect.

- **Silent model substitution** — a consumer UI may route a turn to a different or smaller model than the one labeled, without disclosure. [community-reported], directional only; no Tier-1 source quantifies or confirms the routing behavior.

For all three: **detection rules and mitigation methodology are out of scope here and live in the `prompt-engineering-architect` skill.**

## Gaps

- **Consumer default reasoning effort** — undocumented across all vendors (OpenAI, Anthropic, Google, xAI). The lead gap of this reference.
- **Anthropic consumer citation fidelity** — vendor-silent. OpenAI and Google document a fabrication/self-misrepresentation risk for their consumer surfaces (facts 4–5); Anthropic does not document an equivalent for the consumer surface in the retrieved Tier-1 sources.
- **Consumer numeric output caps and silent-substitution specifics** — community-reported only. No Tier-1 integers exist for consumer output-token caps or for the conditions under which a UI substitutes a model. Do not assert any number for these.
