# Fragments & Unity — Poems by Warwick Allen

Personal poem collection built with the [poetic](https://github.com/Poetic-Poems/poetic) framework.
Published at <https://warwick-allen.github.io/fragments-and-unity/> and Blogger.

This VS Code workspace has two roots: this repo (poems) and `../Code/poetic` (the framework).

## What this repo is

Source poems → build pipeline → generated HTML in `public/`. Do not edit `public/` or
`src/poems/yaml/` directly; they are build artefacts.

## Directory map

```
poems/               ← symlink → src/poems/poem/   (edit poems here)
src/poems/poem/      ← canonical .poem source files (same dir, two paths)
src/poems/yaml/      ← generated YAML (build artefact, do not edit)
src/tools/           ← build scripts (Node.js)
src/templates/       ← Pug template for HTML output
public/              ← generated HTML + CSS (build artefact, do not edit)
raw/                 ← generated plain-text versions (build artefact)
docs/                ← documentation (YAML-SCHEMA.md, POEM-SYNTAX.md, etc.)
scripts/             ← shell helpers (sync-framework, setup-linux, etc.)
editors/             ← Vim syntax highlighting
```

## Build pipeline

```
.poem → (poem-to-yaml.js) → src/poems/yaml/*.yaml → (build-poems.js) → public/*.html
      → (poem-to-raw.js)  → raw/*
```

Full build: `npm run build`  
Build + serve: `npm run build:all` → <http://localhost:8080>  
Tests: `npm test` · Lint/whitespace check: `npm run check` · Build-artefact check: `npm run check:build`

**On WSL/Linux**, npm may resolve Windows binaries. Wrap commands (including the test/check
scripts above):
```bash
./scripts/setup-linux.sh npm run build
```

## Poem file format

Files live in `src/poems/poem/` (accessible as `poems/` from the repo root).

```
Title of the Poem
YYYY-MM-DD

Stanza one line one
Stanza one line two

Stanza two line one
```

- Line 1: title
- Line 2: ISO date (`YYYY-MM-DD`)
- Line 3: blank
- Remaining lines: poem body (blank lines = stanza breaks)
- Files starting with `_` or `.` are ignored by the build (use for drafts/templates)
- `.shared.poem` in `src/poems/poem/` is auto-prepended to every poem before processing
  (defines shared variables like disclaimer text); **user-owned — not overwritten by sync**

See `docs/POEM-SYNTAX.md` for the full syntax (variables, markup, embedded languages, etc.)
and `poem-syntax.ebnf` for the formal grammar.

## Adding or editing a poem

1. Edit/create the `.poem` file in `src/poems/poem/` (or equivalently `poems/`)
2. Run `npm run build` (or `./scripts/setup-linux.sh npm run build` on Linux/WSL)
3. Commit the `.poem` file, plus the generated `public/` and `raw/` files (`src/poems/yaml/`
   is a local build intermediate, excluded via `.gitignore`)

## Framework sync

The build tools in `src/tools/`, `src/templates/`, `scripts/`, and `editors/` are owned by the
upstream `poetic` framework. Sync them with:
```bash
scripts/sync-framework.sh          # uses version in .poetic-version
scripts/sync-framework.sh --ref main   # pull latest
```

Do not hand-edit files that are synced from the framework — changes will be overwritten.
Exceptions — files that are **user-owned** and never overwritten by sync:
- `src/poems/poem/.shared.poem` — shared variables (author name, etc.)
- `public/custom.css` — personal CSS customisations (add styles here)
- `.poetic-config.yaml` — personal build settings (committed to this repo; see below)

`public/poetic.css` is framework-owned (synced). To stop it being overwritten (e.g. if you
pin a local tweak), add it to `skip_paths` in `.poetic-config.yaml`:
```yaml
skip_paths:
  - public/poetic.css
```

`.poetic-config.yaml` is committed to this repo so CI picks it up when building for GitHub
Pages. Keys are grouped hierarchically (e.g. `blogger.sync`, `auto_sync.schedule`) — see
`examples/poetic-config.example.yaml` in the poetic framework repo for a fully-commented
reference of every option, and `docs/BUILD.md` for full details.

## Branch workflow

`main` is protected: it does not accept direct commits or pushes. Every change goes through
a pull request, which the repo owner reviews and merges. Write PR titles in Conventional
Commits format (e.g. `docs: add branch workflow section`), matching the repo's commit history.

All Poetic repositories, this one included, operate in a multi-agent environment: autonomous
and interactive agents, and the maintainer, may push branches, merge pull requests, and move
`main` at any time. Before commencing any changes, make your own dedicated shallow clone of
`origin/main` and work in that — never in a checkout shared with anyone else, such as the
user's working copy (which may be edited at any moment) or a clone another agent is already
using:

```bash
git clone --depth 1 https://github.com/Warwick-Allen/fragments-and-unity.git <scratch-dir>/fragments-and-unity
```

Deepen the clone if the task turns out to need history — for example, a rebase onto a `main`
that has moved needs the merge base (`git fetch --unshallow`, or `git fetch --depth=<n>`).
Commit, push the feature branch, and open the pull request from that clone; delete the clone
once the work has landed.

When you open (or update) a pull request, do not assume `origin/main` is still in the state
it was when you cloned — another change may have merged meanwhile. Confirm the PR is actually
mergeable via `gh` (e.g. `gh pr view <n> --json mergeable,mergeStateStatus`); if it conflicts,
rebase onto the current `main` and push the fix.

## Tech debt

When you defer work, take a shortcut, or notice a known gap, record it in `TECH-DEBT.md`
at the repo root — do not leave it only in a commit message or in chat. Keep entries short
and dated, and delete one when it is resolved.

Route it to the right repo: if the deferred work is in the `poetic` framework (anything
under the synced `src/tools/`, `src/templates/`, `scripts/`, `editors/`, or `docs/`), log
it in the **framework** repo's `TECH-DEBT.md` instead — not here. This repo's `TECH-DEBT.md`
is for consumer-specific debt (poems, `.shared.poem`, `public/custom.css`, `.poetic-config`,
CI, deployment). If you add an entry and refer to it elsewhere (e.g., in code comments), note
that cross-reference in the entry itself, so whoever resolves it knows to also remove those
references.

`TECH-DEBT.md` ends with a permanent Ledger table recording every ID ever allocated, so a
removed entry's ID is never reused. Get a new entry's ID from `scripts/next-tech-debt-id.pl`
rather than counting by hand; add the entry's body under the `## Current Items` heading as a
`### <id> <title>` section, and add a Ledger row (`open`) alongside it. When picking up an
existing open item, follow the "Claiming an item" workflow at the top of `TECH-DEBT.md`.

## Documentation principles

`CHANGELOG.md` (repo root) is the place to record what changed and when. Add an `[Unreleased]`
entry (Keep a Changelog format) for any notable change — one visible to poem readers or site
publishers. Patch-level fixes and routine doc updates don't need entries.

All other docs are as-built: describe the current state only — no "previously", "used to be",
"now uses", "migration completed", or "old format (deprecated)" phrasing. Git log already
records history.

If you encounter historical language in an existing doc, remove it and move the substance to
`CHANGELOG.md` if it's significant.

## Key docs

| File | Contents |
|------|----------|
| `docs/POEM-SYNTAX.md` | Complete `.poem` format spec |
| `docs/YAML-SCHEMA.md` | YAML poem schema |
| `docs/POEM-TO-YAML.md` | Converter docs |
| `docs/BUILD.md` | GitHub Pages deployment |
| `docs/BLOGGER.md` | Auto-publishing poems to Blogger |
| `docs/VIM-SYNTAX.md` | Vim integration docs |
| `poem-syntax.ebnf` | Formal EBNF grammar |
