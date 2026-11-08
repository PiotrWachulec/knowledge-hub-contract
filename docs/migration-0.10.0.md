# Verification and staleness — 0.9.0 → 0.10.0

`SCHEMA.md`'s "Approval and changes" section already named `verified`
("describes a content check, not mere consent to use it... may record
the actual result of a user's review") and `stale_after` ("marks the
need for a review over time") before this release. Neither had a field
shape, an example, or a `validate_hub.py` check — an agent could write
either key however it liked, and nothing would notice a malformed one.
This release doesn't introduce a new idea; it gives two names the
contract already committed to an actual shape and a mechanical check,
the same move 0.9.0 made for "later versions as new files."

This still isn't automated trust or automated freshness (see
"Deliberately not built" in the root README, which stays accurate): no
code grants verification, computes a trust tier into a stored field, or
compares `stale_after` to the current date. `validate_hub.py` stays
deterministic with no wall-clock input, the same reasoning already
behind `external_refs[].cited_as_of` in 0.9.0 — this release extends
that reasoning to a second field, not a new exception to it.

Profile 0.10.0 adds two optional document-level fields:

- `verified` — a list of `{by, at}` entries. `by` is an actor using the
  same convention as `generated.by` (`human:<id>`, `process:<id>`, or
  `<vendor>/<version>`); `at` is an ISO 8601 datetime with timezone
  recording when that actor checked the content. A reader derives a
  trust tier from the array as a whole, without any stored tier field:
  no `verified` key is unverified; entries present with no `human:`
  actor is machine-confirmed; at least one `human:` entry is
  human-reviewed.
- `stale_after` — an ISO 8601 datetime with timezone marking when a
  document needs review.

`validate_hub.py` checks the shape of both: each `verified[]` entry
needs a nonempty `by` and a valid `at`; `stale_after`, if present, must
be a valid ISO 8601 datetime with timezone. The validator has no
opinion on whether a check actually happened or whether a document is
currently stale — declared, not inferred, the same stance it already
takes on `classification`.

**No breaking CLI change.** Both fields are optional document
frontmatter, not `hub.json` or CLI surface — `create_hub.py`'s
arguments are unchanged.

## Adoption in an existing hub — 0.9.0 → 0.10.0

1. Check its contract, Git state, push address, branch, and agreed
   access. Keep local changes; don't run the generator on an existing
   directory.
2. On a separate branch, set `profile_version` to `0.10.0` in
   `hub.json`. No other `hub.json` field changes.
3. Carry over from the template `SCHEMA.md` the rewritten "Approval and
   changes" section (the concrete `verified`/`stale_after` shapes, the
   trust-tier derivation, and the example) and the two new "Structure
   check" bullets, if the hub keeps its own copy. Resolve collisions
   with local policy; don't overwrite the whole file.
4. No backfill is required: existing documents with no `verified` or
   `stale_after` field are valid as-is — absence means unverified, not
   an error. The fields only apply going forward, the next time a
   document is actually reviewed or given an explicit review horizon.
5. Run the local checks with the `.venv` interpreter to confirm the
   bump; a hub with no existing `verified`/`stale_after` usage sees no
   new errors or warnings from this release.
6. Commit and push to the agreed repository; confirm the commit is
   present on the remote and open a PR without auto-merge.

## Limits

`verified[]`'s trust tier is a derivation a reader performs by eye;
nothing in this profile stores it, surfaces it in a report, or gates
`status: stable` on it — `SCHEMA.md` already defers that coupling to
"a later, explicitly established policy," and this release doesn't
establish one. Nothing invalidates an existing `verified[]` entry when
the document changes afterward; recognizing a stale verification after
a substantive edit stays a manual judgment, same as before this field
had a shape. `stale_after` gets even less mechanical support: the
validator checks only that it parses as a datetime, never that it has
passed — there's no report, warning, or scheduled task that acts on it,
the same limitation `external_refs[].cited_as_of` already has in
0.9.0.
