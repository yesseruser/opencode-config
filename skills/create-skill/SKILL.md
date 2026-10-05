---
name: create-skill
description: Use when creating, scaffolding, or drafting a new opencode skill from scratch. Collects name/description/content, scaffolds a valid SKILL.md with proper frontmatter, and installs the skill via install-skill while defaulting to the opencode config repo. Can accept supporting files/dirs to include in the skill folder.
---
# Create Skill

Creates a new opencode skill, scaffolds it correctly, and installs it. Defaults to the opencode config repository for installation.

## Establish the inputs

Collect the following, asking only for what is missing:

1. **Skill name** (required) — kebab-case: `^[a-z0-9]+(-[a-z0-9]+)*$`, 1–64 characters, no leading/trailing/doubled hyphens. Must equal the folder name used.
2. **Description** (required) — 1–1024 characters. Clear and specific.
3. **Purpose/content** (optional) — high-level goal, workflow, rules, examples, or markdown to include in the body.
4. **Supporting files/dirs** (optional) — paths to include (e.g. `scripts/`, `references/`, `assets/`, `templates/`). Ask to confirm inclusion if found.

Do not guess. Validate name/description immediately when provided.

## Scaffold the skill

Create a staging folder for the new skill: `/tmp/create-skill-<name>/skills/<name>/`. 

Write files into that staging location first (source of truth before install):

- `SKILL.md` with frontmatter:
  - `name: <name>` (exact match to folder)
  - `description: <description>` (trimmed)
- Include any supporting files/dirs the user specified, copied in full (preserve structure). Do not write only `SKILL.md`.

Frontmatter rules (opencode compatibility):

- Never emit `tools:` as a YAML **array**. If tools must be noted, use a map shape or omit. Prefer omit unless strictly necessary.
- Never emit `model:` with no provider prefix (e.g. `model: haiku`). Either include `provider/model-id` or omit to inherit. Repair if encountered.

Compose body with clear sections: purpose, inputs, workflow, validation, notes. Keep concise and actionable.

## Validate scaffold

Before delegating to install, verify:
- `name` matches folder and regex.
- `description` length 1–1024.
- `SKILL.md` parses as valid YAML frontmatter + markdown.
- No disallowed frontmatter shapes (tools array, bare model).

## Delegate to install-skill

Use the staged folder as the source. The user wants a soft default to the config repo:

1. **Source** = local directory containing `SKILL.md`: `/tmp/create-skill-<name>/skills/<name>/` (or point to the parent `skills/`? install-skill accepts a local directory containing `SKILL.md`; pass the skill directory itself).
2. **Destination** = **soft default: opencode config repo**. Do **not** force; if user explicitly wants `~/.agents/skills` or current repo `.claude/skills`, honor it. Otherwise proceed with config repo resolution as specified by `install-skill` (candidates: `~/config-git/opencode`, `~/config-vcs/opencode`, `~/.config/opencode` with gates).

Invoke install-skill logic/workflow to:
- Choose destination (respect soft default; ask only if unclear/overridden)
- Copy entire folder (install-skill already does this)
- Validate installed `SKILL.md`
- Complete install per destination rules

## Complete install (config repo path)

When installing into the config repo (default case), follow install-skill's config repo section:
1. Confirm git repo.
2. **Ask before committing.** Never commit unprompted.
3. Commit with granular message + `Co-authored-by: OpenCode <email from git config user.email>`. Follow repo `AGENTS.md`/style (use `git log`).
4. `git pull --rebase`; if conflicts, halt, report, suggest `git rebase --abort`.
5. `git push` (if remote exists and not main per repo rules; follow AGENTS.md).
6. Print remaining next steps only (Nix: `nix flake update opencode-conf` + `nh`; else restart opencode). Do not execute them.

## Notes

- Staging under `/tmp` avoids polluting working trees until install decides destination.
- Always copy entire folder; never SKILL.md alone.
- If the scaffolded skill's name collides in destination, confirm overwrite before proceeding.
