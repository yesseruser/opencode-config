---
name: customize-opencode
description: Use when editing opencode's own configuration — opencode.json, agents, commands, skills, plugins, MCP servers, or permissions. Applies config-vcs repo location and Nix re-materialisation checks.
---

# Customize Opencode

Edits opencode's own configuration on any machine, whether or not that
machine uses Nix. Supplements the built-in `customize-opencode` guidance
(schema, file shapes) with where to edit and how to make the edit live.

## Resolve where to edit

Take the first candidate that passes its gate:

| Order | Path                  | Gate                                                   |
| ----- | --------------------- | ------------------------------------------------------ |
| 1     | `~/config-git/opencode` | git repo containing `opencode.json` or `opencode.jsonc` |
| 2     | `~/config-vcs/opencode` | same                                                   |
| 3     | `~/.config/opencode`    | **is not a symlink**                                   |

Notes on the candidates (copied from install-skill):

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

**If no candidate passes its gate:** stop and ask the user. Report which paths
you checked and why each was rejected, then let them choose.

When the CWD is already inside the config repo (it holds `opencode.json` or
`opencode.jsonc` alongside top-level `skills/`), edit in place — that *is* the
config-repo destination.

## Edit

Follow the built-in `customize-opencode` shapes:

- Full schema reference is `https://opencode.ai/config.json` — fetch it rather
  than guessing when unsure of a field. Preserve `$schema` and untouched fields.
- `model` always carries a provider prefix (`anthropic/claude-sonnet-4-6`).
- `skills` is an object with `paths`/`urls`, `agent`/`command`/`mcp`/`permission`
  are objects keyed by name, `plugin` is an array, `mcp[name].command` is an
  array of strings with required `type`.
- For agents, commands, skills, plugins prefer creating files in the correct
  location (`.opencode/agent/`, `.opencode/command/`,
  `.opencode/skills/<name>/SKILL.md`, `.opencode/plugin/`) over inlining
  everything in `opencode.json`.

Frontmatter rules (opencode compatibility, copied from install-skill):

- Never emit `tools:` as a YAML **array** — convert to a map (`{read: true}`)
  or drop the field, else `ConfigInvalidError` at startup.
- Never emit `model:` with no provider prefix (`model: haiku`) — parses as
  provider `haiku` with empty id → `Model not found: haiku/`. Add a real
  `provider/model-id` or drop to inherit.

## Validate

Check before reporting success:

- `SKILL.md` frontmatter (when touched): `name` matches
  `^[a-z0-9]+(-[a-z0-9]+)*$`, 1–64 chars, equals folder name; `description`
  present, 1–1024 chars.
- `opencode.json` parses and only uses schema fields. A malformed file means a
  broken session on next restart — opencode loads config once at startup and
  does not hot-reload.

## Complete the change (config repo path)

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

### Remaining next steps (Nix check)

On a Nix machine the file is not live until the config is re-materialised:

```
nix flake update opencode-conf   # in ~/.config/home-manager
nh
```

then restart opencode.

- **The push in step 6 is what makes this work.** Nix fetches the config repo
  from GitHub at switch time; it never reads the local clone. Without it, `nh`
  re-materialises the old revision and the change silently does not appear.
- `nix flake update` is required because the input declares no `branch`, so it
  stays pinned to the `narHash` already in `flake.lock`. A plain
  `nix flake update` is fine.
- That leaves an uncommitted `flake.lock` change in `~/.config/home-manager`, a
  separate repository on a different host. Mention it; do not commit it.
- Use `nh`, not `home-manager`, where Nix is managed by nh.

Without Nix, restarting opencode is the only remaining step. Config is loaded
once at startup and is not hot-reloaded — tell the user to quit and restart
opencode for changes to take effect.

## Escape hatches

When config is broken and opencode won't start:

- `OPENCODE_DISABLE_PROJECT_CONFIG=1`: skip local `opencode.json`, start from
  globals only.
- `OPENCODE_CONFIG=/path/to/file.json`: load an additional explicit config.
- `OPENCODE_CONFIG_CONTENT='{"$schema":"https://opencode.ai/config.json"}'`:
  inject inline JSON as final local-scope merge.
- `OPENCODE_PURE=1`: skip external plugins entirely.
