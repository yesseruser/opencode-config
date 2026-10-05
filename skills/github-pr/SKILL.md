---
name: github-pr
description: Use when opening, pushing, or merging a pull request, or when deciding whether a change should get its own branch and PR instead of being committed to main. Also covers the gh CLI conventions for all GitHub interaction, such as `gh pr create`, `gh pr merge`, `gh pr checks`, and `gh repo clone`.
---

# GitHub PRs

Branch, push, and merge conventions for GitHub pull requests, plus the rules for
deciding when a change earns its own PR.

## Use `gh` for everything

All GitHub interaction goes through the `gh` CLI — never the REST API by hand,
never a web page:

```bash
gh pr create
gh pr merge
gh pr comment
gh pr checks
gh repo clone
gh repo create
```

`gh` handles auth, and it is the tool the user already has configured. Reach for
`gh api` only for endpoints `gh pr` does not cover.

## Branch and PR or commit straight to main

Push by default whenever a remote exists **and you are not on the main branch**.
Committing directly to main is the exception, not the default.

Committing straight to main is acceptable when:

- The change is configuration-only or GitHub workflow and does not affect the
  resulting library or application (for example, `AGENTS.md` or other agent
  config).

Everything else that is non-trivial or makes a noticeable change in the
application gets a **new branch before committing**, and a PR rather than a
commit to main.

A PR is mainly for the automated changelog, so even config-only changes are fine
to land as a PR — the reason the exception exists is that such changes do not
need one.

## Before committing

Check the current commit message style with `git log` and adhere to it for the
summary; commit messages should ALWAYS have a highly descriptive description. See
the `git-commit` skill for the full commit rules.

## Merging

Use `gh pr merge --merge --delete-branch`, while still on the PR branch. That
deletes the current branch and checks out the merged master branch as a side
effect — do not delete the branch or switch branches by hand afterwards.

```bash
gh pr merge --merge --delete-branch
```

**Never merge a PR without the user's explicit approval**, even when its checks
are green and the review looks finished. Report the review status and stop.