# Contributing

Thanks for considering a contribution. This repository is a collection of publicly
developed Claude skills, and it gets better when people with different experience report
problems, challenge claims, and submit improvements. Critique-oriented contributions
are as valuable as additive ones.

Read this document in full before your first contribution. It's shorter than most.

## Before You Start

### Read the Contract

[`CLAUDE.md`](./CLAUDE.md) is the technical contract for all content in this repository.
It defines the quality bar, the data handling rules, the structural conventions, and the
authority hierarchy. Your contribution is evaluated against it.

The two Prime Directives in `CLAUDE.md` override everything else in this document.
If anything here conflicts with them, they win.

### Check Existing Work

- Search open and closed issues for your topic before filing a new one.
- Search open PRs for overlapping work before starting a new one.
- If you find a relevant existing thread, comment there instead of opening a duplicate.

### Scope of Contributions

In-scope:
- Corrections to factual errors
- Updates for current model versions or deprecated APIs
- New examples (positive or negative) that clarify existing guidance
- New resource files expanding coverage of an existing skill
- New skills following the structural conventions in `CLAUDE.md`
- Improvements to prose clarity without changing meaning
- Test harness additions that validate existing claims

Out-of-scope:
- Opinion pieces, "best model" rankings, or buying guides
- Marketing or evangelism for any provider
- Content derived from private or non-public sources (see Data Handling below)
- Changes that add dependencies without clear justification
- Scope expansion that makes a skill's one-line description require "and"

## Data Handling (Non-Negotiable)

This repository contains only publicly available information. Before submitting any
contribution, you must personally confirm that none of the following has leaked in:

- **Client, employer, or organizational information** of any kind, yours or anyone
  else's. This includes organization names, internal project names, personnel,
  business relationships, and seemingly innocuous context ("at my last company we...").
- **Internal documentation** from any organization. Internal wikis, Slack exports,
  private Notion pages, design docs, architectural diagrams, proprietary prompts, or
  any material not published on a public URL.
- **Anything marked** sensitive, confidential, proprietary, classified, internal-only,
  or NDA-covered — even if the marking was on the source document and not on the excerpt
  you used.
- **Real credentials, tokens, API keys, connection strings, or internal endpoints.**
  Use placeholders like `sk-REPLACE-ME` and `https://your-endpoint.example.com`.
- **Real personal data.** Names (other than public figures in their public capacity),
  email addresses, physical addresses, phone numbers, account IDs, or any other PII.
  Use synthetic or widely-known fictional data in examples.

**When in doubt, exclude.** The cost of omitting an example is trivial. The cost of a
leak is not recoverable.

### If You Realize Something Slipped Through

If you discover that a PR you've submitted contains non-public information:

1. Do not comment explaining what leaked. The comment itself becomes part of the
   permanent record.
2. Follow the process in [`SECURITY.md`](./SECURITY.md) for reporting the leak privately.
3. Close the PR without merging. A new PR can be opened from a clean branch.

### AI-Assisted Contributions

The repository expects that contributors will use AI assistants (Claude, Copilot,
Cursor, etc.). This is fine and explicitly welcome. If you use substantial AI assistance:

- **Verify every claim independently.** Do not submit content you cannot personally
  defend. AI assistants hallucinate confidently, cite sources that don't exist, and
  describe API surfaces that have been deprecated. Read the primary sources yourself.
- **Run a leak check on every changed file.** AI assistants may have context from your
  other work — client engagements, internal tools, proprietary systems. That context
  leaks into generated content in ways that are easy to miss. Review code comments,
  example data, file paths, variable names, and prose for anything that shouldn't
  be public.
- **Declare AI assistance in the PR description.** This is informational, not
  disqualifying. Reviewers approach AI-assisted PRs with calibrated attention, not
  rejection. A template line like `AI assistance: [description of how and for what]`
  at the bottom of the PR description is sufficient.

PRs that appear to be AI-generated without verification — hallucinated API references,
fabricated citations, or generic prose that doesn't engage with the actual content —
will be closed with a request to resubmit after the contributor has done the verification
work themselves.

## How to Contribute

### Reporting a Bug or Incorrect Content

File an issue using the bug template. Include:

- Which skill and which file
- The specific claim, example, or behavior that's wrong
- What you expected or what the correct information is
- A primary source supporting the correct information, if available
- The skill version and contract version you were working against (from the front
  matter of the affected file)

Do not include non-public information in the issue. If demonstrating the problem
requires sensitive context, say so in the issue and use the process in
[`SECURITY.md`](./SECURITY.md) to share the context privately.

### Requesting New Coverage

File an issue using the feature template. Include:

- What model, API, or behavior should be covered
- Why it belongs in this repository (i.e., genuinely public, useful to practitioners,
  not already covered elsewhere at higher quality)
- Links to primary sources that would anchor the coverage
- Your willingness to contribute the PR yourself, if applicable

### Submitting a Pull Request

1. **Fork and branch.** Create a branch with a descriptive name
   (`fix/qwen-sampling-defaults` not `patch-1`).
2. **Make the change.** Follow the structural and content conventions in `CLAUDE.md`.
   Keep PRs focused — one logical change per PR. Multiple unrelated changes should be
   multiple PRs.
3. **Update metadata.** Bump the skill version in affected SKILL.md files. Update the
   `retrieved:` date where you re-verified against primary sources. Add a CHANGELOG
   entry.
4. **Self-review against the PR checklist** (below).
5. **Open the PR.** Use the PR template. Link any related issues.
6. **Respond to review.** Maintainers may request changes, ask for sources, or flag
   concerns. Engage in good faith.

### PR Checklist

Before marking a PR ready for review, confirm every item:

- [ ] I have read `CLAUDE.md` and my contribution conforms to it.
- [ ] No non-public information is included. I have reviewed every changed file for
      client names, employer references, internal paths, real credentials, PII, and
      context from private conversations.
- [ ] Every factual claim has inline provenance pointing to a Tier 1 or Tier 2 source.
- [ ] Tier 2 (community-reported) claims are explicitly flagged as such.
- [ ] No Tier 3 sources (SEO content mills, AI-generated review sites) are cited.
- [ ] Examples use synthetic or placeholder data.
- [ ] `retrieved:` dates are updated on files where I re-verified content.
- [ ] SKILL.md files remain under the size ceiling (target 500 lines, ceiling 1000).
- [ ] No marketing language, emojis, or performative helpfulness in skill content.
- [ ] Python code uses 3.13+ syntax and is production-shaped (typed, error-handled).
- [ ] CHANGELOG.md is updated with the change.
- [ ] The skill's `contract-version:` field matches current `CLAUDE.md` or a migration
      note explains why it's older.
- [ ] If I used AI assistance, I have declared it in the PR description and verified
      all claims independently.

## Review Process

### What Reviewers Check

Reviewers apply the maintainer checklist from `CLAUDE.md`:

- No non-public data
- No AI-slop language
- Provenance on factual claims
- Retrieval dates current
- Structural conformance (SKILL.md size, resource split, naming)
- Examples use synthetic data
- No vendored documentation that should have been a link

Reviewers may also:
- Ask you to re-verify a claim against the primary source
- Request a smaller PR if the scope is too large
- Defer a PR that conflicts with pending architectural changes
- Close a PR that doesn't align with the repository's purpose

### Timeline

This is a maintained project, not a staffed product. Reviews happen on a best-effort
basis. Expect:

- Initial response: within 2 weeks
- Review round-trips: within 1 week each
- Merge or definitive decision: within 1 month of submission

If a PR is stale with no reviewer response for longer than these windows, a polite ping
on the PR is welcome.

### When PRs Get Closed Without Merge

Reasons a PR may be closed:
- Contains non-public information (remove and resubmit from a clean branch)
- Relies on sources excluded by the tier policy
- Scope is larger than a single logical change (split into multiple PRs)
- Appears to be unverified AI-generated content
- Duplicates work already in progress
- Doesn't align with the repository's purpose or structural conventions

Closure isn't personal. Resubmission is welcome after addressing the issue.

## Development Environment

### Python

- Python 3.13+
- `uv` for project management
- No `pip install` in documentation examples unless demonstrating legacy patterns

### JavaScript / TypeScript

Not used in this repository unless a specific skill's subject matter requires it
(for example, a skill about browser automation). Default language is Python.

### Dependencies

Before adding a dependency to any skill or test harness:
- Prefer the standard library if possible
- Prefer a widely-adopted, actively-maintained package if not
- Run an OSS SCA check on the dependency (this repository's convention is to document
  SCA results in the PR description for any new dependency)
- Justify the dependency in the PR description

### Running the Test Harness

When the test harness exists, contribution instructions will appear here and in
`tests/README.md`. Until then, validation is manual and based on the claim-marker
system documented in `CLAUDE.md`.

## Attribution

Contributors are acknowledged in the repository's `CONTRIBUTORS.md` file (created on
first merge). Contribution does not transfer copyright; your contributions remain under
the repository's declared license (`LICENSE`).

This repository does not currently offer bug bounties, stipends, or other financial
acknowledgment.

## Code of Conduct

Be a technical peer. Disagree with claims, not with people. Critique specifically.
Assume good faith. When someone points out a problem with your work, thank them —
they saved you from shipping a problem.

Harassment, personal attacks, or bad-faith engagement are grounds for removal from
the project.

## Questions

For questions about whether a specific contribution fits:
- Open a discussion (not an issue) on the repository
- Reference the specific skill and section you're considering
- Do not include private context; if context matters and is non-public, use the
  process in [`SECURITY.md`](./SECURITY.md) to share it privately

For questions about data handling, suspected leaks, or security concerns:
- Use the process in [`SECURITY.md`](./SECURITY.md). Do not use public channels.