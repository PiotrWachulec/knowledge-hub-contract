# Security Policy

## Scope

This repository ships a local, read-only Markdown structure validator
(`scripts/validate_hub.py`), a non-blocking grounding report
(`scripts/check_evidence.py`), and a hub generator
(`scripts/create_hub.py`). All three are Python standard library only,
run entirely locally, make no network calls, and never write outside
the hub directory they're pointed at (the validator and evidence script
never write at all). There's no server, no service, and no credential
handling anywhere in this repository.

Realistic concerns worth reporting: a crafted hub directory or
`hub.json`/document frontmatter that causes the validator or generator
to read or write outside the intended hub boundary, to crash in a way
that masks a real structural problem, or to silently pass content it
should flag (a validator that lies is worse than one that's merely
slow).

## Supported Versions

Only the current `main` branch is maintained. There is no backport
policy for older tagged profile versions — see
`skills/create-knowledge-hub/references/profile.md` for what
`profile_version` actually tracks (it's a content-contract version, not
a software release line with its own security maintenance).

## Reporting a Vulnerability

Please use [GitHub's private vulnerability reporting](https://github.com/PiotrWachulec/master-hub/security/advisories/new)
for this repository rather than opening a public issue, so a fix can
land before the report becomes public. If that's unavailable to you,
open an issue with as little exploit detail as practical and ask for a
private channel.

Please include: the exact input (a minimal `hub.json`/document
frontmatter reproducing the issue), the command you ran, and what you
expected versus what happened.

There's no bug bounty. We'll acknowledge a report, assess it, and aim
to ship a fix (with a CHANGELOG entry and, if it changes validator
behavior, a `docs/migration-*.md` note) at a pace proportional to actual
impact — this is a local CLI tool, not a hosted service with a live
attack surface.
