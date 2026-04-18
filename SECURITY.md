# Security

This document covers how to report security issues and data leaks in this repository.
The primary concern for a public documentation repository is inadvertent disclosure of
non-public information, not traditional software vulnerabilities — but both are in scope.

## Scope

Report via the private process below if you find:

- **Non-public data committed to the repository.** Client names, employer references,
  internal paths, personnel information, organization-specific context, or any material
  marked (or reasonably assumed to be) sensitive, confidential, proprietary, or
  internal-only.
- **Credentials or secrets in history.** API keys, tokens, connection strings, private
  endpoints, SSH keys, certificates, or authentication material — even if the repository
  currently compiles without them, the credential may still be live.
- **Real personal data.** Names other than public figures in their public capacity,
  email addresses, phone numbers, physical addresses, account identifiers, or other PII.
- **Security flaws in example code** that would reasonably be copied into production
  by practitioners following the guidance (SQL injection, unsafe deserialization,
  authentication bypasses, etc.).
- **Supply chain concerns.** Suspicious dependencies, typosquatted package names,
  packages that have been compromised upstream, or packages with known unpatched
  vulnerabilities that haven't been addressed.
- **Prompt injection or exfiltration patterns** in skill content that could cause Claude
  (or any AI assistant loading the skill) to behave unsafely.

## How to Report

**Do not open a public issue for any of the above.** Public issues become part of the
permanent searchable history, and the report itself can compound the disclosure.

Instead, use one of the following private channels, in order of preference:

1. **GitHub's private vulnerability reporting.** On the repository's Security tab,
   select "Report a vulnerability." This creates a private advisory visible only to
   repository maintainers.

2. **Email.** Send to the security contact listed on the repository's main page.
   Subject line: `[SECURITY]` followed by a brief non-disclosive summary
   (for example, `[SECURITY] Possible credential in commit history`). Do not include
   the actual leaked content in the subject line.

3. **If neither is available**, open a minimal public issue that says only
   "Requesting private contact for security report" with no further detail, and
   wait for a maintainer to reach out.

## What to Include in a Report

- **What you found** — the specific file, line, or commit range.
- **Why it's a concern** — which category above it falls into.
- **How you found it** — helps maintainers assess scope (is this isolated or systemic?).
- **Whether the disclosure is already public** — was the data visible only in the
  repository, or is it on the public internet already through other channels?
- **Your contact information** — for follow-up questions.

Do not include the actual sensitive content in the report body unless necessary for
identification. References to commit SHAs and line numbers are sufficient in most cases.

## What Not to Do

- **Do not publish the finding publicly** before maintainers have had a reasonable
  chance to respond (see Response Timeline below).
- **Do not attempt to clean up the leak yourself via a public PR.** Pull requests
  that reference the leaked content in their diff, commit messages, or descriptions
  extend the exposure. Report privately and let maintainers handle the rewrite.
- **Do not test leaked credentials** to confirm they're live. If credentials appear
  in the repository, report them and assume they are live regardless.
- **Do not probe for more leaks by social engineering contributors.** This isn't a
  penetration testing engagement; treat it as collaborative cleanup.

## Response Timeline

- **Acknowledgment:** within 3 business days of receiving the report.
- **Initial assessment:** within 7 business days, including whether the report is in
  scope and what the remediation path looks like.
- **Remediation:** depends on severity.
  - Live credentials or active PII exposure: immediate, within 24–48 hours of
    assessment.
  - Historical non-public data: within 2 weeks, coordinated with affected parties
    where possible.
  - Unsafe example code: within one release cycle.
- **Disclosure:** if the report uncovered something that affects users of the repository
  (not just the repository itself), maintainers will publish a security advisory after
  remediation. The reporter is credited if they wish.

If a report does not receive acknowledgment within 3 business days, a polite follow-up
via the same channel is appropriate.

## Remediation Process

### For Leaked Non-Public Content

When non-public content is confirmed in the repository history, remediation follows
this order:

1. **Assess whether the content is already public elsewhere.** If it's already public
   (the contributor is discussing it on their own blog, it's in another public repo),
   the remediation priority is lower — but the repository should still reflect the
   project's content standards.
2. **Rewrite history.** Use `git filter-repo` or equivalent to remove the content from
   all commits in which it appears, not just add a fixing commit. A subsequent commit
   that removes a credential does not remove the credential from history.
3. **Force-push the rewritten history.** This invalidates existing forks and clones.
   Maintainers coordinate a `main` branch reset and document the rewrite in
   `CHANGELOG.md` with the date and a non-disclosive reason ("history rewritten to
   remove non-public content; forks must re-fetch").
4. **Rotate any credentials.** If credentials were exposed, treat them as compromised
   regardless of whether anyone appears to have accessed them. Notify the credential
   owner if it's not the contributor themselves.
5. **Notify affected parties.** If the leak affected a party other than the contributor
   (e.g., a previous employer), the contributor is responsible for notifying them.
   Maintainers may assist with language but will not make the notification on the
   contributor's behalf.

### For Security Flaws in Example Code

1. Determine whether the flaw is theoretical (bad practice demonstrated as bad practice)
   or actual (bad practice demonstrated as good practice, likely to be copied).
2. Fix the example. Add a test to the harness if applicable.
3. If the flaw was present for more than 30 days, publish an advisory describing the
   affected file and the corrected pattern.

### For Supply Chain Concerns

1. Verify the concern against the dependency's current published state and known
   advisories.
2. If confirmed, remove or replace the dependency.
3. Document the finding in `CHANGELOG.md`.
4. If the dependency is used in example code that practitioners may have copied,
   publish an advisory.

## For Contributors: Preventing Leaks

The most common cause of leaks in a public skills repository is AI-assisted
contributions that pull context from the contributor's private work. Contributors
using AI assistants must:

- **Verify every changed file manually** for references to clients, employers,
  internal tooling, proprietary systems, or non-public URLs.
- **Review AI-generated examples specifically** — example data, variable names, file
  paths, and comments are the highest-risk surfaces.
- **Check commit messages** — AI assistants may include context in commit descriptions
  that shouldn't be public.
- **Use a fresh AI assistant context** when working on this repository where possible.
  Reduces the chance that other-project context leaks in.
- **Consider running a secret-scanning tool locally** before pushing. Tools like
  `gitleaks`, `trufflehog`, or `detect-secrets` catch common credential patterns.
  They don't catch organizational context, so manual review is still required.

This repository may add pre-commit hooks or CI-based secret scanning. Even if those
exist, they are a second line of defense. The contributor remains primarily responsible
for what they submit.

## For Maintainers: Incident Handling

When a report comes in:

1. **Acknowledge within 3 business days**, even if the full assessment takes longer.
2. **Keep discussion private** until remediation is complete. Do not discuss specifics
   in public issues, commit messages, or PR descriptions.
3. **Document internally** what was found and what was done. A private record of
   incidents helps identify patterns (e.g., if a particular contributor repeatedly
   leaks data, the pattern is visible).
4. **Do not retaliate against reporters** for finding problems. This includes reports
   about maintainers' own commits.
5. **Publish an advisory after remediation** if users of the repository need to know
   (credentials to rotate, example code to update, forks to re-clone).

## Scope Limitations

This repository is documentation and guidance, not production software. Several
traditional security concerns don't apply directly:

- **We do not have a "product" with a threat model** in the conventional sense. The
  "product" is text that humans and AI assistants read.
- **We do not handle user data at runtime.** There is no service, no database, no
  authentication system.
- **Vulnerabilities in the AI models themselves** (Claude, Gemini, etc.) are out of
  scope. Report those to the respective providers.
- **Vulnerabilities in the tools used to consume this content** (Claude Code,
  Claude.ai, etc.) are out of scope. Report those to Anthropic via their security
  channels.

In scope is the content of this repository and the code and examples within it.

## Credits

Reporters who make valid reports are credited in the advisory published after
remediation, unless they prefer to remain anonymous. This repository does not offer
financial bounties. Acknowledgment is the only form of compensation.

## Questions

For general questions about security policies for this repository, open a discussion
(not an issue). For actual security concerns, use the private reporting channels above.