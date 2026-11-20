# OKF 0.3 alignment — 0.11.1 → 0.12.0

This project's templates were written against
[OKF v0.2](https://github.com/open-knowledge-format/open-knowledge-format/blob/main/SPEC.md).
OKF has since released v0.3 (the project also moved out of the
`GoogleCloudPlatform` GitHub org into its own `open-knowledge-format` org).
This release checks what v0.3 actually changed against what this contract
already had, and closes the gaps that turned out to be real — it isn't a
wholesale rewrite to track an upstream spec version number for its own
sake.

Three things changed, each independently:

- **`okf_version` moves from `"0.2"` to `"0.3"`** in `hub.json` and
  `knowledge/index.md`'s frontmatter. This project's existing
  path-resolution convention (`resource` relative to the containing
  document, not the bundle root) already matches OKF 0.3's new default —
  that's the one change OKF 0.3 itself calls breaking between minor
  versions — so nothing about how links or `sources[].resource` resolve
  actually changes here. The bump is a declaration catching up to a
  convention this contract already followed.
- **`superseded_by`**, a new optional document field: the reverse of
  `supersedes`, declared on the old (deprecated) document to name the new
  one that replaced it. `docs/migration-0.9.0.md`'s "Limits" section
  already named this gap when `supersedes` first shipped ("the old file's
  frontmatter gains no backlink... reading the old file alone doesn't
  show where its successor lives"); OKF 0.3 formalizing the same idea is
  what prompted actually closing it now.
- **A stale-verification warning.** `SCHEMA.md`'s "Approval and changes"
  section already said a trust tier is derived from `verified[]`, but
  flagged that nothing invalidates an old entry after a substantive
  edit. OKF 0.3 formalizes exactly this as "only a current verification
  counts" (an entry's timestamp at or after the content's last
  meaningful change). `validate_hub.py` now applies that rule as a
  non-blocking warning: an entry whose `at` predates the document's own
  `generated.at` stays in the array as history but doesn't count toward
  the tier a reader derives.

A smaller, docs-only change rides along: the actor-identifier convention
for `generated.by`/`verified[].by` now documents a `team:<id>` prefix
alongside `human:`/`process:`/`<vendor>/<version>`, matching OKF 0.3's
own addition. `validate_hub.py` already accepted any nonempty `by`
string, so this adds no new check.

**Deliberately not adopted from OKF 0.3:** bundle-root `index.md`
frontmatter for `title`/`description`/`license` (this contract's
`knowledge/index.md` already has a fixed, different shape — four layer
links plus `okf_version` — and doesn't need those fields); typed
relationships, the attestation runtime protocol, and validity windows,
all explicitly deferred in OKF 0.3 itself; and the `usecases/` layer,
which doesn't apply to a bundle-scoped bronze/silver/gold/decisions
pattern.

**No breaking CLI change.** `superseded_by` is optional document
frontmatter, not `hub.json` or CLI surface — `create_hub.py`'s arguments
are unchanged.

## Adoption in an existing hub — 0.11.1 → 0.12.0

1. Check its contract, Git state, push address, branch, and agreed
   access. Keep local changes; don't run the generator on an existing
   directory.
2. On a separate branch, set `profile_version` to `0.12.0` and
   `okf_version` to `"0.3"` in `hub.json`. No other `hub.json` field
   changes.
3. Set `okf_version` to `"0.3"` in `knowledge/index.md`'s frontmatter —
   the only other file this profile bump touches directly. The four
   layer links below it are unaffected.
4. Replace the hub's own copies of `scripts/validate_hub.py` and
   `scripts/check_evidence.py` with the versions from this profile (both
   now check against `profile_version: "0.12.0"` and
   `okf_version: "0.3"`); don't hand-edit the version strings into an
   old copy of the script.
5. Carry over from the template `SCHEMA.md` the extended "Source
   freshness and supersession" section (the `superseded_by` paragraph
   and the second example), the sharpened "Approval and changes" section
   (the current-verification trust-tier rule), the `team:` mention in
   "Documents and references", and the corresponding "Structure check"
   bullets, if the hub keeps its own copy. Resolve collisions with local
   policy; don't overwrite the whole file.
6. No backfill is required: existing documents with no `superseded_by`
   are valid as-is, and an existing `verified[]` entry that predates its
   document's `generated.at` only produces a new non-blocking warning,
   never an error — review those warnings once, add a fresh `verified[]`
   entry where the content has genuinely been re-checked since, and
   otherwise treat the warning as informational. Add `superseded_by` to
   already-deprecated documents only where their replacement is actually
   known; it's a convenience, not a retroactive requirement.
7. Run the local checks with the `.venv` interpreter to confirm the bump;
   a hub with no existing `supersedes`/`superseded_by` relationships and
   no stale `verified[]` entries sees no new errors or warnings from this
   release.
8. Commit and push to the agreed repository; confirm the commit is
   present on the remote and open a PR without auto-merge.

## Limits

`superseded_by` is a convenience pointer, not a consistency guarantee:
the validator checks its own shape (existing `.md` target, not self, the
declaring document's own `status: deprecated`) but never cross-checks it
against a `supersedes` on the other side — two files can disagree about
who replaced whom, and nothing here notices. The stale-verification
warning compares against `generated.at` only; it has no concept of which
specific change made an old verification stale, and a document whose
`generated.at` was bumped for an unrelated reason (not a substantive
content change) will still age out its old `verified[]` entries. Neither
change computes or stores a trust tier anywhere — both stay a reader's
derivation from the raw array, same as before this release.
