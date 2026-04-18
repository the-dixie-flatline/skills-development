# Claude Skills Playground

A small, growing collection of Claude skills I've built because I needed them and
figured other people probably do too.

This is a playground, not a product. I'm using it to experiment with skill design,
figure out what actually makes a skill useful, and publish the results where they
might help someone else. Expect rough edges, opinions, and the occasional
rewrite-from-scratch when I figure out I was doing it wrong.

## What's Here

Skills aimed at things the average developer does often enough to care about — the
stuff you find yourself doing a few times a week, or a few times a day. Prompt
engineering references, coding workflow helpers, research and documentation tools,
things like that.

Each skill lives in its own directory with a `SKILL.md` and usually some resource
files. Start at the top-level directory listing; each skill's own README explains
what it does and when to use it.

## What It's Not

Not a comprehensive skill library. Not a certified-production anything. Not a
replacement for reading the actual provider documentation when you need ground truth.

There are probably better and more robust out skills you and I can both use out there
but you ultimately learn by doing, not copying.

The skills here reflect how I work and what I've found useful. Your mileage will
vary. Disagreement is welcome; that's what the issue tracker is for.

## Source Validation — Known Limitation

Many primary sources that this repo's skills cite sit behind aggressive bot / agent blockers (Cloudflare and similar) that return 403 to automated fetchers. Notable offenders as of 2026-04-19 include several `.gov` properties (`defense.gov`, `dodcio.defense.gov`, `ecfr.gov`, `federalregister.gov` HTML pages), `salesforce.com` and `help.salesforce.com`, and some `iso.org` standard pages.

**Consequence:** an AI assistant running a validation pass may report a primary source as broken or unreachable when the URL is, in fact, fine in a human-loaded browser. Before reporting a broken source — in an issue, a PR, or a skill update — confirm the URL manually in a browser. Treat 403s from automated fetchers as "verify manually," not "stale."

Where a structured-data alternative exists, skills prefer it: the Federal Register JSON API (`federalregister.gov/api/v1/documents/{doc-number}.json`) has been a reliable fallback when the HTML surface is blocked. See [`TODO.md`](./TODO.md) for the ongoing workflow backlog.

## Contributing

Contributions are very welcome. Bug reports, corrections, better examples, new
skills — all useful.

One scope guideline: skills should target things an average developer actually does
regularly. A few times a week is a good floor; a few times a day is better. Prompt
engineering references, git workflow helpers, debugging patterns, code review
assistance — yes. "Organize my sock drawer by color" — probably not, unless your
sock drawer is somehow a core part of your development workflow, in which case I
have questions.

Before contributing, read [`CONTRIBUTING.md`](./CONTRIBUTING.md) and
[`CLAUDE.md`](./CLAUDE.md). The short version:

- Verify your claims against primary sources.
- No private or proprietary data — yours or anyone else's.
- If you used an AI assistant to help, double-check for leaked context.
- Keep PRs focused.

If you're not sure whether an idea fits, open a discussion before writing the PR.
Saves everyone time.

## Security

If you find non-public data, credentials, or anything that shouldn't be in a public
repo — please report it privately. See [`SECURITY.md`](./SECURITY.md) for the
process. Do not open a public issue for those; the report itself makes it worse.

## Maintained By

This repository is maintained by Michael Traffanstead ([@the-dixie-flatline](https://github.com/the-dixie-flatline)).
I review PRs when I can. Timelines are in `CONTRIBUTING.md`.

## License

MIT. See [`LICENSE`](./LICENSE). Contributions are accepted under the same license.

The aggregate work — the selection, structure, and arrangement of skills in this
repository — is Copyright (c) 2026. Individual contributions remain the copyright
of their authors, licensed to this project under MIT.

## A Final Note

This repo exists because I got tired of keeping prompt engineering notes, skill
patterns, and research in scattered places and wanted somewhere to put them where
other people could benefit and tell me where I'm wrong. If it's useful to you,
great. If you think I'm wrong about something, tell me. If you want to contribute,
read the docs and send a PR. That's the whole pitch.
