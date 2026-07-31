---
scope: W1frag
---

# Tech debt

Deferred work and known gaps specific to this repository (Fragments &
Unity), kept as a per-item register: every record ever allocated lives as
one file under `tech-debt/`, named by its ID
(`TD-W1frag-<YYMMDD><NN>.md`), with YAML frontmatter carrying the record's
state and a Markdown body describing it. The full format, ID grammar and
scope-code registry are specified in
[docs/TECH-DEBT-REGISTER.md in Poetic-Poems/poetic](https://github.com/Poetic-Poems/poetic/blob/main/docs/TECH-DEBT-REGISTER.md).

Route each item to the right repo: if the deferred work is in the `poetic`
framework (anything under the synced `src/tools/`, `src/templates/`,
`scripts/`, `editors/`, or `docs/`), file it in the **framework** repo's
register instead — not here. This register is for consumer-specific debt
(poems, `.shared.poem`, `public/custom.css`, `.poetic-config`, CI,
deployment).

`perl scripts/td-check.pl` (also `npm run check:td-register`) validates
the register and runs on every pull request via
`.github/workflows/tech-debt-register.yml`, alongside two guards: no file
in `tech-debt/` may ever be deleted or renamed once on `main` (the
append-only Ledger guarantee — IDs are never reused), and no old-format
`### TD` item sections may reappear in this file.

## Filing an item

1. Allocate the ID with `scripts/next-tech-debt-id.pl --ref origin/main`
   (after a `git fetch origin`). It cannot see IDs allocated on unmerged
   branches, so also skim open pull requests and `td/*` branches. If two
   filings do collide, git surfaces it as an add/add conflict on the
   filename; the later one renames to the next free NN before merging.
2. Create `tech-debt/<id>.md` with frontmatter `id`, `title`,
   `status: open`, `filed` (today, matching the ID's date), an optional
   `review:` provenance line, and a body describing what, why it matters,
   where, and a suggested fix.
3. If the item is referenced elsewhere (code comments, docs), note those
   references in the body so whoever resolves it removes them too.

## Claiming an item

Before starting work on an open item, check and take the claim against the
shared state, never against what a local checkout happens to say:

1. `git fetch origin`, then confirm the item's `status:` is `open` (not
   `in-progress`) **as of `origin/main`** — e.g. via
   `perl scripts/get-tech-debt-record.pl --ref origin/main <id>`.
2. Confirm nobody holds a claim: `git ls-remote origin "refs/heads/td/<id>"`
   must print nothing, and skim open pull requests for the ID.
3. Create the claim branch, named exactly **`td/<id>`**, from `origin/main`;
   flip the item's `status:` to `in-progress`; commit and push. The branch
   name is the claim lock: a rejected push means someone else claimed
   first — abandon quietly; never force-push over it.
4. Open a **draft** pull request right away, then do the work on the same
   branch.
5. Once verified, flip the item's frontmatter to `status: resolved` and
   fill `resolved:` (today's date) and `ref:` (the PR number), leaving the
   body in place, and mark the PR ready for review.

If a claim is abandoned, close the draft PR and delete the `td/<id>`
branch — the record on `main` still says `open`.

## Resolution and history

A resolved item's file is its permanent record: the body stays, and
`git log --follow tech-debt/<id>.md` is the item's audit trail. Never
delete or rename an item file, and never flip a resolved item back —
re-opening debt means filing a new item that references the old one. An
item that turns out not to be debt keeps its file too: `status: not-debt`,
with `ref:` pointing at where the content moved.

Aggregated views of the register are generated on demand, never committed.
