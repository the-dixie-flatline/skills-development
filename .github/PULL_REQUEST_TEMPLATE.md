<!--
Before opening: read CONTRIBUTING.md and CLAUDE.md. The PR checklist below captures the trip-wire items; the full checklist is in CONTRIBUTING.md.
-->

## Summary

<!-- 2-4 sentences. What changed and why. Focus on "why" — the diff shows "what". -->

## Type of change

<!-- Check all that apply. -->

- [ ] Bug fix / correction to existing content
- [ ] Stale content refresh (`retrieved:` dates updated)
- [ ] New coverage within an existing skill (new family, new section)
- [ ] New skill
- [ ] Schema / structural change
- [ ] Docs-only change (README, CONTRIBUTING, etc.)
- [ ] Repository infrastructure (templates, CODEOWNERS, etc.)

## Related issue

<!-- Link the issue this addresses. Use "Closes #N" or "Refs #N". If no issue exists and this is a non-trivial change, consider opening one first. -->

## Scope verification

- [ ] This PR contains **one logical change**. Unrelated changes have been split into separate PRs.
- [ ] The change aligns with the in-scope / out-of-scope rules in `CONTRIBUTING.md`.

## Provenance and sourcing

<!-- For PRs that add or change factual claims in skill content. Skip for purely structural or infrastructure PRs. -->

- [ ] Every new or changed factual claim has inline provenance in the format `[source: url, retrieved YYYY-MM-DD]`.
- [ ] Tier 2 (community-reported) claims are flagged as such.
- [ ] No Tier 3 sources are cited. The exclusion list in `SCHEMA.md` was checked.
- [ ] `retrieved:` dates on files where I re-verified content are updated.
- [ ] `[testable: ...]` markers with stable IDs are used for empirically validatable claims.

## Data-handling confirmation

This is the most important section. A reviewer will reject a PR that does not pass this self-check.

- [ ] I reviewed **every changed file** for non-public data: client names, employer references, internal tooling, personnel, real PII, real credentials, internal URLs.
- [ ] Example data is synthetic or uses widely-known fictional references.
- [ ] No content originated from a private conversation, internal wiki, NDA-covered material, or otherwise non-public source.
- [ ] No sensitive context leaked into commit messages, branch names, or file paths.

If you realize something non-public slipped into this PR, **close it without merging** and follow the process in `SECURITY.md`.

## AI-assistance declaration

<!--
If you used substantial AI assistance (Claude, Copilot, Cursor, etc.), declare it here. This is informational, not disqualifying.

Example: "Claude Code assisted with drafting the reasoning-control section of qwen-prompt-api.md; I verified every claim against the Hugging Face model card and Alibaba Cloud docs myself."

Leave the field blank / remove the line if this was fully manual.
-->

**AI assistance:**

## Self-review

- [ ] I have run through the full PR checklist in [CONTRIBUTING.md §PR Checklist](../blob/main/CONTRIBUTING.md#pr-checklist).
- [ ] SKILL.md files remain under the size ceiling (target 500 lines, ceiling 1000).
- [ ] `CHANGELOG.md` has an entry for this change.
- [ ] The skill's `contract-version:` matches the current `CLAUDE.md`, or a migration note explains why it's older.
- [ ] No marketing language, emojis, or performative helpfulness in skill content.

## Testing / verification notes

<!-- For skill content changes: how did you verify the claims? For structural changes: what did you check? One or two sentences. -->
