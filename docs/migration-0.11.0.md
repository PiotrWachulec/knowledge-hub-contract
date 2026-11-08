# supersedes must target a .md document — 0.10.0 → 0.11.0

A fresh-context repository review found a gap between what `SCHEMA.md`
already documented and what `validate_hub.py` actually checked.
`SCHEMA.md`'s own "Source freshness and supersession" prose has said,
since 0.9.0, that `supersedes` "only makes sense between documents that
carry a `status` field, so it points at another `.md` document, not a
raw `.txt` dump." The "Structure check" bullet list separately promised
this as a hard error: a `supersedes` target whose `status` isn't
`deprecated` fails validation.

In practice, that promise only held when the target happened to be a
`.md` file. `check_supersedes()` looked up a target's status in
`doc_status`, a dict populated only for `.md` documents the walk
actually parsed — a `.txt` file (explicitly allowed elsewhere in
`SCHEMA.md` as a valid bronze format: "Raw text can be kept as `.txt`
and described by a bronze note with metadata") never gets an entry.
`supersedes: old-source.txt` pointing at a real, existing `.txt` file
therefore produced **zero errors or warnings** — the exact "this
replaces that" claim the old file can never actually back up (a `.txt`
file structurally can't carry `status: deprecated`), passing silently.

This release closes that gap. It doesn't change what a valid hub looks
like — any hub whose `supersedes` already points at `.md` documents, as
the contract always intended, is unaffected.

## What changed

`validate_hub.py`'s `check_supersedes()` now rejects a `supersedes`
target that exists on disk but isn't a `.md` file, with a new error:
`supersedes must point to a .md document, not a raw source file`. This
check runs before the existing `status: deprecated` check, and is
independent of it — a missing target still produces the pre-existing
"does not exist" error from `local_file()`, unchanged.

**No breaking CLI change, no new frontmatter field.** This is a stricter
validator check on an already-documented rule, not a new capability a
hub has to opt into. It only fails a hub that was already violating the
spirit of `supersedes` without the validator catching it.

## Adoption in an existing hub — 0.10.0 → 0.11.0

1. Check its contract, Git state, push address, branch, and agreed
   access. Keep local changes; don't run the generator on an existing
   directory.
2. On a separate branch, set `profile_version` to `0.11.0` in
   `hub.json`. No other `hub.json` field changes.
3. Carry over from the template `SCHEMA.md` the updated "Source
   freshness and supersession" sentence and the updated "Structure
   check" bullet (both now say `.md` document explicitly), if the hub
   keeps its own copy.
4. Run the local checks with the `.venv` interpreter to confirm the
   bump. A hub whose every `supersedes` already points at a `.md`
   document sees no new errors. A hub that happens to have a
   `supersedes` pointing at a `.txt` file will now see a new error —
   that was always an incorrect claim; fix it by either removing the
   `supersedes` field or converting the old source to a proper `.md`
   document with `status: deprecated`.
5. Commit and push to the agreed repository; confirm the commit is
   present on the remote and open a PR without auto-merge.

## Limits

This closes one specific gap (a `.txt` target bypassing the
`status: deprecated` check entirely) found by a direct read of the
validator code against the documented contract — it wasn't found by
broader fuzzing or a systematic audit of every check, so a similar gap
elsewhere in `validate_hub.py` isn't ruled out by this release. The new
check only fires for a target that exists on disk; a missing target
still relies entirely on the pre-existing "does not exist" error, which
this release doesn't change.
