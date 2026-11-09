---
name: curate-knowledge-hub
description: Add material or prepare an output in an existing Git hub designated by the user. Follow that hub's own contract; keep knowledge editing separate from contract changes and Git handling.
---

# Curate knowledge in an existing hub

This skill operates on an existing hub; it doesn't assume its name, owner,
or location, and doesn't build a hub registry. The designated hub's own
`AGENTS.md`/`SCHEMA.md`/`hub.json` set the methodology, format, quality
checks, Git mechanics, and approval; they take precedence over this skill
and are the source of truth for *how* to execute a task once you're
working inside the hub. This skill covers what happens before that
point — triage, authorization, and recognizing which track a request
falls into — plus a fallback for a target too old, or too unrelated, to
carry that guidance itself.

This contract is written in English, but that doesn't set the language of
the knowledge you edit: keep content in the language of the source
material, and don't translate it unless the user explicitly asks — see
"Content quality" in the hub's SCHEMA.md.

## Preparation without changing data

1. Determine the target directory/repository and the task scope. Don't
   search neighboring repositories looking for a default hub. Reading and
   analysis don't imply consent to commit, push, or open a PR. Establish
   authorization for those operations from the request, earlier
   agreements, or the hub's adopted contract. In profile 0.2.1 and later, the
   marker for this rule is `policy.remote_copy:
   "required_after_validated_task"` in hub.json, not a section title in
   AGENTS.md. When it applies, commit/push during editing without asking
   "should I push?" again. An instruction for local-only work takes
   precedence. Don't impose this rule on a hub whose contract doesn't
   contain it. In an older hub, a missing field doesn't mean the new rule
   was adopted; follow the existing contract and authorizations instead.
   From profile 0.2.1 onward, a missing/unknown field value, or a conflict between
   the configuration and instructions, stops work until clarified. Don't
   add the field yourself as part of a content edit. Clone the repository
   only into an agreed, new directory.
2. Before checkout/pull, check the status, current branch, remote, and
   local commits. Read the hub's AGENTS.md, SCHEMA.md, and hub.json, and
   the index that follows from `bundle_root`. Don't assume a `knowledge/`
   path for other profiles. Report an unknown version; don't migrate
   automatically. If the contract can't be applied unambiguously, stop
   instead of guessing. Also check the push address and the agreed access
   scope. Report a missing copy destination or authorization before
   editing; don't pick an arbitrary remote or create one yourself. In a
   hub embedded in a project, use that project's Git.
3. Preserve other people's changes. Unrelated changes, e.g. editor
   settings, don't require automatically interrupting the edit if they can
   be safely separated. Don't commit, stash, or reset them yourself. When
   a sync or a branch conflicts with them, agree on a separate worktree or
   ask for the state to be cleaned up. With unknown local commits or work
   on someone else's branch, don't switch or push them automatically.

## Choosing the action and how to save

Recognize adding a source, answering a question, preparing an output, or
changing the contract. Combine them when the request covers several.
Choose the track before the first edit:

| Track | Scope | Default way to save, if authorized |
|---|---|---|
| query | answering a question about the base, without archiving | no write — doesn't require push or PR authorization |
| content, `policy.content_review: "direct"` (default) | sources, drafts, indexes, log, and decisions, including an archived query answer | commit and push to the agreed main branch; a draft isn't approved knowledge |
| content, `policy.content_review: "required_pr"` | same scope as above | commit and push to a branch, then a PR, without merging it yourself — same procedure as `contract` |
| contract | AGENTS.md, SCHEMA.md, hub.json, check tooling, migrations, and changes to the directory model | commit and push to a branch, then a PR, without merging it yourself |

An ordinary new concept or a gold output directory is content, not a
change to the directory model. A new type requiring a hub.json edit is a
contract change. If a task needs both and the contract change wasn't
requested, ask to expand the scope. A requested content + contract bundle
goes through a PR in full.

`policy.content_review` decides which content row applies, independent of
`policy.knowledge_editing` and of `kind`. Missing or unrecognized
`content_review` stops work the same way an unrecognized `remote_copy`
does (see step 1 above), rather than silently defaulting to either value.
The hub's own `AGENTS.md` ("Ongoing remote copy") is the source of truth
for exactly what each policy field means and requires — this table only
classifies the request, it doesn't restate that mechanics.

## Execution

Once you've read the target's own `AGENTS.md`, follow it exactly for venv
setup, package installs (including an on-demand conversion library
instead of a custom parser when the obvious tool is missing — see its
"Mandatory check after editing" section), running `validate_hub.py` and
`check_evidence.py`, and the full commit/push/PR/conflict-resolution
procedure. Don't restate or reinvent any of that here; a stricter local
instruction for this task still takes precedence.

If the target predates that guidance (a hub generated before profile
0.2.1, or a plain Git repository this skill was pointed at that isn't a
knowledge-hub-contract hub at all), fall back to this baseline instead:

- Review the diff and stage only the files or hunks belonging to this
  task. No secrets, no `.venv`, no someone else's uncommitted changes.
- Fetch the agreed remote before pushing; update the base only
  fast-forward, rebasing only your own commits on a clean tree.
- Never force-push or rebase a branch that already has an open PR — merge
  the base into it as a normal merge commit instead. A conflict stops the
  work for a description, not an automatic resolution.
- After pushing, confirm the commit landed on the target remote branch
  (fetch and check it's an ancestor) — a clean local tree isn't proof.
  Don't create an empty commit when there's nothing new to push.
- Record approval and content corrections per whatever contract the
  target does have; a commit or a green check alone doesn't grant
  approval or human-verification status.

## Handoff

State the commit or PR, the remote/branch, the scope of changes, the
check result, and the copy status. If the remote write wasn't authorized
or failed, report "no current remote copy" and what's left locally.
Preserve the work; don't report full completion. With a required backup
pending, don't start further write tasks without removing the blocker or
agreeing on local-only work.

This skill doesn't install sync and doesn't run automatic ingest. It isn't
a lock against concurrent agents; under concurrency, an agreed work split
and separated save locations are needed.
