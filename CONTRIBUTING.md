# Contributing

Thanks for considering a contribution. This repository is the generator
and contract templates for portable Markdown knowledge hubs — not a hub
itself. If you're looking for how an agent should work *inside* a
generated hub, that's `AGENTS.md`/`SCHEMA.md` in
`skills/create-knowledge-hub/assets/hub/`, not this file.

## Setup

Standard library only, no external dependencies.

```bash
git clone https://github.com/PiotrWachulec/knowledge-hub-contract.git
cd knowledge-hub-contract
python3 -m venv .venv          # Python 3.10+
.venv/bin/python -B -m unittest discover -s tests -v
```

The test suite generates real hubs into temporary directories and runs
`validate_hub.py`/`check_evidence.py` against them — there's no mocking
of the generator or the validator.

## Before opening a PR

- Run the full test suite locally; CI runs it again on Python 3.10–3.13.
- If you touched `scripts/validate_hub.py`, `scripts/check_evidence.py`,
  `SCHEMA.md`, `AGENTS.md`, or `create_hub.py`, add or update tests in
  `tests/` — behavioral tests against generated output, not unit tests
  against internal functions.
- Keep the change scoped. A bug fix doesn't need surrounding refactors; a
  new capability doesn't need speculative generalization for a case
  nobody asked for.

## What kind of change this is

`skills/create-knowledge-hub/references/profile.md` defines the
versioning policy this project actually follows:

- **patch** (`x.y.Z`) — docs-only wording fix, no validator or
  contract-shape change. No migration doc needed.
- **minor** (`x.Y.0`) — a new, backward-compatible capability: a new
  contract field, a new validator check, a new script. Needs its own
  `docs/migration-x.Y.0.md` and a `CHANGELOG.md` entry.
- **major** (`X.0.0`) — a breaking change to the OKF bundle shape or an
  existing hub's directory structure. Needs a maintainer discussion
  first — open an issue before a PR.

If your change bumps `profile_version`, follow the pattern of the most
recent entry in `CHANGELOG.md` and the most recent `docs/migration-*.md`
file: what changed, why, the adoption steps for an existing hub, and the
honest limits of what the change does and doesn't do. "Deliberately not
built" in `README.md` exists for a reason — this project prefers a
named, explicit gap over a half-automated feature that looks more
capable than it is.

## Adding a convention

A pluggable style/language convention (like `polish-typography`) is a
plain Markdown file under `skills/create-knowledge-hub/assets/conventions/`,
registered nowhere else — it only becomes active in a generated hub that
requests it via `--conventions`. Most conventions need no validator
backstop at all; add one in `validate_hub.py`'s `CONVENTION_CHECKS` only
when there's a cheap, reliable mechanical check for it (see how
`polish-typography` is implemented for the shape to follow).

## Pull requests

- One logical change per PR. Mixing a contract change with an unrelated
  cleanup makes both harder to review.
- Describe *why*, not just what — the commit/PR history is how a future
  reader understands a decision without re-deriving it.
- Never force-push to a shared branch; never bypass CI or review to get
  a change in faster.
- A maintainer merges after review; please don't merge your own PR even
  if you have access.

## Reporting a bug or proposing a feature

Open an issue first for anything beyond a small, obvious fix — especially
a major-version-shaped change to the OKF bundle shape or directory
structure. Use the issue templates; they ask for the context a fix or a
design discussion actually needs.

## Code of conduct

Participation in this project is governed by `CODE_OF_CONDUCT.md`.
