## What changed and why

<!-- The "why" matters more than the "what" — the diff already shows the what. -->

## Kind of change

- [ ] Docs-only fix (patch) — no validator/contract-shape change
- [ ] New backward-compatible capability (minor) — includes a `docs/migration-x.Y.0.md`
- [ ] Breaking change to bundle shape or directory structure (major) — discussed in an issue first
- [ ] Repo infrastructure (CI, templates, governance docs) — no `profile_version` change

## Checklist

- [ ] `python -B -m unittest discover -s tests -v` passes locally
- [ ] Tests added/updated for any validator, generator, or contract behavior change
- [ ] `CHANGELOG.md` updated, under `[Unreleased]` (only for a `profile_version` change)
- [ ] Relevant docs updated: `SCHEMA.md`, `AGENTS.md`, `references/profile.md`, `README.md` (only for a `profile_version` change)

## Test plan

<!-- How you verified this, beyond the automated suite — e.g. a scratch hub you generated and validated by hand. -->
