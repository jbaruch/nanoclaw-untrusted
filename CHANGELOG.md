# Changelog

## Unreleased

### CI

- Refresh `review-{anthropic,openai}.lock.yml` — bumps the gh-aw AWF binary pin off `v0.25.28` (which 404s in CI) onto a working version. No source `.md` changes; only generated artifacts move.
- Replace the all-skills `tessl skill review` loop in `publish-tile.yml` with a `uses:` call to `jbaruch/coding-policy/.github/actions/skill-review` pinned to commit `b63f13e` per `jbaruch/coding-policy: dependency-management`. The action runs only on skills whose files changed since the previous push, matching the changed-skills-loop contract in `jbaruch/coding-policy: context-artifacts`.
- Brings this repo into line with the four sibling plugin repos (nanoclaw-admin/-core/-host/-trusted) that completed the same CI cleanup earlier.

### Surface sync

- `tile.json` adds `entrypoint: README.md` per `jbaruch/coding-policy: context-artifacts`.
- `README.md` and `CHANGELOG.md` introduced (none existed previously). Both will be maintained going forward as required by the policy.

The README's rules-table summaries are auto-extracted first-paragraph excerpts from each rule file. Refine them per rule when the wording is misleading; this commit is a structural bootstrap, not authored prose.
