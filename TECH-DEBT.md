# Tech debt

Deferred work and known gaps specific to this repository (Fragments & Unity).
Record an entry here whenever you defer something, rather than leaving it only in
a commit message or in chat. Keep entries short and dated; remove one when it is
resolved.

Framework-level tech debt — anything under the synced `src/tools/`,
`src/templates/`, `scripts/`, `editors/`, or `docs/` — belongs in the `poetic`
framework repo's `TECH-DEBT.md`, not here.

## TD26071001 CLAUDE.md says to commit `src/poems/yaml/`, but it's gitignored

`CLAUDE.md`'s "Adding or editing a poem" workflow says to commit the `.poem`
file plus the generated `src/poems/yaml/`, `public/`, and `raw/` files, but
`.gitignore` excludes `src/poems/yaml/*.yaml` — it has never actually been
committed. Likely CLAUDE.md is stale after a decision to treat the YAML as a
pure local intermediate (it's fully regenerable from the `.poem` source, unlike
`raw/` and `public/`, which genuinely are committed). Fix by updating
CLAUDE.md's wording to drop `src/poems/yaml/` from the commit list, unless
there's a reason it should actually be tracked that isn't captured here.
