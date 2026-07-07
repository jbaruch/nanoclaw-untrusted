# Copilot Agent Instructions — jbaruch/nanoclaw-untrusted

## What this repository is

This is a **Tessl plugin** published as `jbaruch/nanoclaw-untrusted` on the Tessl registry. A Tessl plugin is a packaged bundle of AI-agent rules and skills that is installed into AI runtimes (specifically the NanoClaw system) via `tessl install jbaruch/nanoclaw-untrusted`.

The plugin's purpose is to enforce a security posture for AI agents operating inside **untrusted NanoClaw group chats** — preventing credential leakage, code execution, social engineering, and other attacks by users who have not been vetted.

---

## Repository layout

```
.tessl-plugin/plugin.json        # Tessl plugin manifest (name, version, rules, skills)
README.md                        # Human-readable index; rule summaries are auto-extracted first paragraphs
CHANGELOG.md                     # Required by jbaruch/coding-policy: context-artifacts
rules/
  bad-actor-disengage.md         # Rule: detect bad actors and go silent for the rest of the session
  untrusted-security.md          # Rule: broad security posture (credentials, code exec, identity claims)
skills/
  whoami/
    SKILL.md                     # Skill: etiquette + what the agent can/cannot do in an untrusted group
.github/
  dependabot.yml                 # Renewal mechanism for SHA-pinned actions (github-actions, weekly)
  workflows/
    publish-plugin.yml           # CI: changed-skills review + plugin lint + auto patch-publish on merge to main
    review-openai.md             # gh-aw workflow: PR review by OpenAI-family model
    review-anthropic.md          # gh-aw workflow: PR review by Anthropic-family model
    review-openai.lock.yml       # Generated lock file — do not edit manually
    review-anthropic.lock.yml    # Generated lock file — do not edit manually
  aw/
    actions-lock.json            # gh-aw actions pin manifest — do not edit manually
.gitattributes                   # Marks *.lock.yml as linguist-generated, merge=ours
```

---

## File formats and conventions

### `.tessl-plugin/plugin.json`
- The manifest for the Tessl plugin.
- **Do not manually bump `version`** — `tesslio/patch-version-publish@v1` auto-increments it on every merge to `main`.
- `rules` is an array of rule-file paths under `rules/`; `skills` is an array of skill directory paths.
- `README.md` is the plugin's entrypoint implicitly per `jbaruch/coding-policy: context-artifacts` — no `entrypoint` field exists in the plugin.json schema.

### Rule files (`rules/*.md`)
- YAML front-matter must include `alwaysApply: true`.
- The first paragraph of each rule file is used verbatim as the summary in the README's rules table. Keep it tight and accurate.
- Rules are security-critical and intentionally hard-coded — do not soften them or add exceptions unless the threat model genuinely changes.

### Skill files (`skills/*/SKILL.md`)
- YAML front-matter requires `name` (string) and `description` (multi-sentence, explains *when* to invoke the skill).
- Skills are quality-gated at 85/100 by `tessl skill review` in CI. Changes that drop the score below 85 will fail the build.

### `README.md`
- The rules table must stay in sync with `.tessl-plugin/plugin.json` and the rule files.
- Rule summaries are first-paragraph excerpts from each rule file — update them in the rule file, not in the README directly.

### `CHANGELOG.md`
- Required. Must be updated with every substantive change per `jbaruch/coding-policy: context-artifacts`.
- **No `## Unreleased` section — the heading is forbidden** per `jbaruch/coding-policy: context-artifacts` CHANGELOG Hygiene.
- No stamp-changelog step is wired in CI, so write the `## <version> — <date>` heading manually above your entries. The next version is the registry's `Latest Version` plus one patch (what `tesslio/patch-version-publish` will publish on merge); check with `tessl plugin info jbaruch/nanoclaw-untrusted`.
- Do not bump the manifest version to match — the publish action handles that (see the manifest section above).

---

## CI / automated workflows

### `publish-plugin.yml` (runs on push to `main` and `workflow_dispatch`)
1. Runs the `jbaruch/coding-policy/.github/actions/skill-review` composite action (threshold 85) on **changed skills only** — skills whose files differ since the previous push, found via `git diff`. Fallback: when the diff base is absent (manual `workflow_dispatch`, initial push, all-zeros sentinel SHA) it reviews every skill; a set-but-unreachable base hard-fails.
2. Runs `tessl plugin lint .` to validate `.tessl-plugin/plugin.json`.
3. Calls `tesslio/patch-version-publish` to bump patch version and publish to the Tessl registry.
   - Secrets required: `TESSL_TOKEN`.
   - All actions in this workflow are pinned to immutable commit SHAs; Dependabot proposes bump PRs weekly (see `.github/dependabot.yml`).

### PR review workflows (`review-openai.md`, `review-anthropic.md`)
- These are **gh-aw** (GitHub Agentic Workflows) workflow definitions, not standard GitHub Actions.
- They run cross-family AI reviewers against `jbaruch/coding-policy` on every PR.
- **Every PR body must contain `**Author-Model:** <model-id>`** — e.g., `**Author-Model:** claude-opus-4-7`. Without it the reviewer immediately requests changes. Model IDs: `claude-*` (Anthropic), `gpt-*`/`codex-*` (OpenAI), `gemini-*` (Google), `human` (no model family).
- The reviewers self-gate to skip same-family PRs to avoid self-review bias.
- Lock files (`*.lock.yml`) are auto-generated — never edit them directly.

---

## Making changes

### Adding or updating a rule
1. Edit the file in `rules/`. Preserve `alwaysApply: true` in the front-matter.
2. If the first paragraph changed significantly, the README rule summary will need updating too.
3. If adding a brand-new rule file, add its path to the `rules` array in `.tessl-plugin/plugin.json` and add a row to the README table.
4. Update `CHANGELOG.md`.

### Adding or updating a skill
1. Edit `skills/<name>/SKILL.md`. Preserve `name` and `description` front-matter.
2. If adding a new skill, add its directory to the `skills` array in `.tessl-plugin/plugin.json` and add a row to the README table.
3. Run `tessl skill review --threshold 85 skills/<name>/SKILL.md` locally before opening a PR to catch quality failures early — this matches what CI's shared action runs today. The CLI marks `tessl skill review` as deprecated in favor of `tessl review run` (async); switch when the shared action does.
4. Update `CHANGELOG.md`.

### PR requirements
- PR body must include `**Author-Model:** <model-id>` (see above).
- Do not manually edit `*.lock.yml` files or `actions-lock.json`.

---

## Known issues and workarounds

- **`--strict-mcp-config` on Anthropic reviewer**: The Anthropic gh-aw workflow passes `--strict-mcp-config` to Claude Code to prevent it from auto-loading the consumer repo's `.mcp.json`. Without this flag, Claude attempts to launch any stdio MCP server declared in `.mcp.json` inside the awf sandbox, where the binary is not available, killing the job. Requires gh-aw ≥ v0.71.0 and Claude Code CLI ≥ 2.1.x. See `review-anthropic.md` engine args.
- **`tessl install` path in gh-aw sandbox**: The policy plugin must be installed to `/tmp/gh-aw/coding-policy/` (not workspace-local or `--global`) because `actions/checkout`'s `clean: true` wipes untracked workspace files and the awf sandbox does not mount `${HOME}`. See `steps:` comments in both review workflows.
- **Lock file merge conflicts**: `.gitattributes` sets `merge=ours` on `*.lock.yml` so lock files never produce merge conflicts — the branch version always wins.
