---
name: install-skill
description: Use when the user wants to install, add, or vendor an opencode skill from a GitHub repo, a git/HTTPS URL, or a local directory — e.g. "install this skill", "add the <name> skill", or a pasted repo link. Installs into ~/.agents/skills, the current repo's .claude/skills, or the opencode config repo, asking which when unspecified.
---

# Install Skill

Installs an Agent Skill into a location opencode actually loads from, on any
machine, whether or not that machine uses Nix.

## Establish the inputs

Two things, either given by the user or asked for:

1. **The source** — see [Resolve the source](#resolve-the-source).
2. **The destination** — see [Choose the destination](#choose-the-destination).

Ask only for what the user has not already specified.

## Resolve the source

Accept any of:

- GitHub shorthand: `owner/repo` or `owner/repo/path/to/skill`
- Any git or HTTPS URL
- A local directory containing a `SKILL.md`
- A direct URL to a `SKILL.md`

Enumerate with `gh`:

```bash
gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1 \
  --jq '.tree[].path' | grep 'SKILL\.md$'
```

Fetch contents with:

```bash
gh api repos/<owner>/<repo>/contents/<path> --jq '.content' | base64 -d
```

If the source resolves to several `SKILL.md` files, list them and ask which to
install. Do not guess. Repositories shipping a family of skills — a plugin repo
with `caveman`, `ultracave`, `megacave`, … subfolders — are normal, and are the
reason this step exists.

## Choose the destination

If the user already named a destination, skip straight to its section below.
Otherwise offer the options below, in this order:

1. **`~/.agents/skills`** — opencode auto-loads `~/.agents/skills/*/SKILL.md`
   globally. No git, no rebuild. Not managed by git at all.
2. **The opencode config repo** — go to
   [Resolve the config-repo destination](#resolve-the-config-repo-destination).
3. **The current repo's `.claude/skills`** — go to
   [Resolve the current-repo destination](#resolve-the-current-repo-destination).

Option 3 is conditional. Offer it **only when the CWD is inside a git
repository**: `git rev-parse --show-toplevel` must succeed. When it fails, offer
exactly the first two options.

## Resolve the config-repo destination

**Only do this when the user chose the config repo option.** Probing is wasted
work, and misleading, if they picked `~/.agents/skills` or the current repo.

Take the first candidate that passes its gate:

| Order | Path            | Gate                                                   |
| ----- | --------------- | ------------------------------------------------------ |
| 1     | `~/config-git/opencode` | git repo containing `opencode.json` or `opencode.jsonc` |
| 2     | `~/config-vcs/opencode` | same                                                   |
| 3     | `~/.config/opencode`    | **is not a symlink**                                   |

Notes on the candidates:

- Orders 1 and 2 are the *same* repository under different folder names on
  different machines. This list is per-machine: if this machine names it
  something else, add that path. Never hardcode `~/config-git`.
- Orders 1 and 2 are ordinary local clones. Nix does not know they exist — they
  are referenced by no Nix configuration. Validate them; never create, clone,
  or `mkdir -p` one on their behalf.
- Order 3 is the live opencode config directory on machines that do not use Nix.
  **If it is a symlink, disregard it entirely** — do not offer it, do not
  mention it. A symlink there means a package manager generated it; the target
  is read-only and is overwritten on every switch, so writing into it is both
  impossible and wrong.

**If no candidate passes its gate:** stop and ask the user. Do not fall back to
`~/.agents/skills` on your own. Report which paths you checked and why each was
rejected, then let them choose.

## Resolve the current-repo destination

**Only do this when the user chose the current repo option.** Probing is wasted
work, and misleading, if they picked one of the first two options.

1. `git rev-parse --show-toplevel` — if this fails, the CWD is not in a work
   tree. The option was never valid: say so and go back to offering the first two
   options.
2. The destination is `<worktree root>/.claude/skills/`.

Notes on this destination:

- **`.claude`, not `.opencode`.** opencode reads `.opencode/skills`,
  `.claude/skills`, and `.agents/skills` alike. `.claude/skills` keeps the repo
  usable by Claude Code as well, which is the point of vendoring a skill into
  someone else's project.
- opencode discovers project skills by walking up from the CWD to the git
  worktree root, so the root is the only placement that loads from every
  subdirectory of the repo.
- The file is live the moment it is written. Nothing in Nix fetches it, so there
  is no rebuild, no `flake.lock`, and no switch to wait for.
- If the worktree root *is* the opencode config repo — it holds
  `opencode.json` or `opencode.jsonc` alongside a top-level `skills/` — say that
  the config repo option reaches every machine while `.claude/skills` loads only
  inside this repo. Install where the user asked anyway.

## Install

Write to `<destination>/skills/<name>/`, or — for the current repo option — to
`<worktree root>/.claude/skills/<name>/`.

- The directory is `skills`, plural, in both layouts. opencode does not read
  `skill/`.
- Copy the **entire skill folder**, not just `SKILL.md`. Skills ship supporting
  files — `scripts/`, `references/`, `assets/` — and copying only the manifest
  leaves them orphaned. Installing `caveman-compress` without its `scripts/*.py`
  yields a skill that fails at runtime.
- `mkdir -p` the `skills` parent. It usually does not exist yet.

## Validate

Check the installed `SKILL.md` before reporting success.

Required frontmatter:

- `name` — matches `^[a-z0-9]+(-[a-z0-9]+)*$`, 1–64 characters, equal to the
  folder name. No leading, trailing, or doubled hyphens.
- `description` — present, 1–1024 characters. A skill without one is filtered
  out and never surfaces to the model.

Two frontmatter shapes authored for Claude Code break opencode. Repair them
rather than passing them through:

| Shape                                               | Failure                                                          | Fix                                                 |
| --------------------------------------------------- | ---------------------------------------------------------------- | --------------------------------------------------- |
| `tools:` as a YAML **array**                        | `ConfigInvalidError` at startup — opencode rejects it             | convert to a map (`{read: true}`) or drop the field |
| `model:` with **no provider prefix** (`model: haiku`) | parses as provider `haiku` with an empty id → runtime `Model not found: haiku/` | add a real `provider/model-id`, or drop to inherit |

Report anything you changed. opencode loads config once at startup and does not
hot-reload, so a malformed file means a broken session.

## Complete the install

### `~/.agents/skills`

Nothing to run. Tell the user to restart opencode. Done.

### Config repo

The git sequence runs here. The Nix rebuild does not.

1. Confirm the destination is a git repository. If it is not — a plain
   `~/.config/opencode` on a machine without Nix — skip to step 7.
2. **Ask before committing.** Never commit unprompted.
3. `git commit` — follow the repository's own `AGENTS.md` where present:
   granular commits, descriptive messages, `Co-authored-by:` trailer.
4. `git pull --rebase`
5. **If the rebase conflicts, halt.** Do not resolve it, do not push. Leave the
   rebase in progress, report which files conflicted, and suggest
   `git rebase --abort`.
6. `git push`
7. Print the remaining next steps. Do not run them.

### Current repo

Read the target repo's `AGENTS.md` and `CONTRIBUTING.md` if either exists, and
follow whatever git flow they specify. **When neither file specifies one, install
only:** write the files, leave them uncommitted, and tell the user. Do not branch,
commit, or push on your own initiative.

When a flow *is* specified, follow it under the same safety rules the config repo
section uses:

- **Ask before committing.** Never commit unprompted.
- Stage only the new skill folder. The worktree is the user's; unrelated
  modifications sitting in it are not yours to sweep into a commit.
- Halt on a rebase conflict: leave the rebase in progress, report which files
  conflicted, and suggest `git rebase --abort`.

Then restart opencode. No `nix flake update`, no `nh` — nothing re-materialises
this file, so the remaining next steps do not apply.

### Remaining next steps

These are for the config repo only. `~/.agents/skills` and current-repo installs
need nothing but an opencode restart.

On a Nix machine the file is not live until the config is re-materialised:

```
nix flake update opencode-conf   # in ~/.config/home-manager
nh
```

then restart opencode.

- **The push in step 6 is what makes this work.** Nix fetches the config repo
  from GitHub at switch time; it never reads the local clone. Without it, `nh`
  re-materialises the old revision and the skill silently does not appear.
- `nix flake update` is required because the input declares no `branch`, so it
  stays pinned to the `narHash` already in `flake.lock`. A plain
  `nix flake update` is fine.
- That leaves an uncommitted `flake.lock` change in `~/.config/home-manager`, a
  separate repository on a different host. Mention it; do not commit it.
- Use `nh`, not `home-manager`, where Nix is managed by nh.

Without Nix, restarting opencode is the only remaining step.

## Uninstalling

Remove `<destination>/skills/<name>/` — or, for a current-repo install,
`<worktree root>/.claude/skills/<name>/` — then reverse whatever made it live:
commit the removal and `nh` on a Nix machine, or just restart opencode for
`~/.agents/skills` and current-repo installs. Leaving a stale copy in both
locations loads the skill twice.