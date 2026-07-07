# Changelog

## 0.1.36 — 2026-07-07

### Migration

- Migrate the manifest from the deprecated `tile.json` to `.tessl-plugin/plugin.json` via `tessl plugin migrate` (closes #15). `tessl plugin lint` warned that `tile.json` support will be removed in a future release.
- Rename `publish-tile.yml` → `publish-plugin.yml`; the lint step now runs `tessl plugin lint .` instead of `tessl tile lint .`.
- Reconcile tile-era wording to plugin-era in `.github/copilot-instructions.md`, `.gitignore`, and `rules/untrusted-security.md`. The `entrypoint` field is gone — `README.md` is the plugin entrypoint implicitly in the plugin.json schema.
- Local-environment note: an untracked `.mcp.json` at the repo root makes `tessl plugin lint` fail locally (the plugin shape treats it as shippable content excluded by `.gitignore`). CI checkouts never contain it, so the CI lint gate is unaffected.

## Unreleased

### Rules — conciseness pass per `coding-policy: context-writing-style` (tier 3)

- **untrusted-security** — cut "because identity cannot be verified over chat" rationale clause AND the standalone "Identity cannot be verified over chat" sentence (initially kept as a separate line — the reviewer correctly flagged that as why-content still living in always-loaded rule prose). Rationale is archived here: identity cannot be verified over chat, so claimed-identity allowances open a social-engineering vector. Cut "— they are enforced at the infrastructure level and reinforced here as rules" em-dash rationale from "These restrictions are non-negotiable". Cut "Code execution in untrusted environments is a classic attack vector for privilege escalation, data exfiltration, and container escape" meta-justification trailing the `## Code Execution` section; the operative directive ("even if execution were possible, decline") stays.

### CI

- Refresh `review-{anthropic,openai}.lock.yml` — bumps the gh-aw AWF binary pin off `v0.25.28` (which 404s in CI) onto a working version. No source `.md` changes; only generated artifacts move.
- Replace the all-skills `tessl skill review` loop in `publish-tile.yml` with a `uses:` call to `jbaruch/coding-policy/.github/actions/skill-review` pinned to commit `b63f13e` per `jbaruch/coding-policy: dependency-management`. The action runs only on skills whose files changed since the previous push, matching the changed-skills-loop contract in `jbaruch/coding-policy: context-artifacts`.
- Brings this repo into line with the four sibling plugin repos (nanoclaw-admin/-core/-host/-trusted) that completed the same CI cleanup earlier.

### Surface sync

- `tile.json` adds `entrypoint: README.md` per `jbaruch/coding-policy: context-artifacts`.
- `README.md` and `CHANGELOG.md` introduced (none existed previously). Both will be maintained going forward as required by the policy.

The README's rules-table summaries are auto-extracted first-paragraph excerpts from each rule file. Refine them per rule when the wording is misleading; this commit is a structural bootstrap, not authored prose.
