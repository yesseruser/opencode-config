---
name: git-commit
description: Use when making a commit, deciding how to split changes into commits, or moving work from one branch to another. Covers reading project CONTRIBUTING.md and AGENTS.md first, granular commit splitting, matching the repo's commit message style, and the required Co-authored-by trailer.
---

# Git Commits

How to shape and write commits in this user's repositories.

## Read the project's rules first

Adhere to `CONTRIBUTING.md` and `AGENTS.md` (or similar files) when they exist in
the project root. They take precedence over this skill.

## Split changes into granular commits

Commit granularly, per git best practices. If a commit can be split into smaller
chunks that are still meaningful, split it.

Meaningful means each chunk stands on its own as a change: it does not leave the
tree broken, and it is not an arbitrary fragment of one edit. Splitting
`fix: correct the widget colour` into `fix: correct the widget colour` and
`chore: reformat widget.ts` when the second is just noise is not meaningful.

## Write the message

1. Check the current commit message style with `git log` and adhere to it for
   the summary line.
2. The message should ALWAYS have a highly descriptive description.
3. Add this trailer, using the email that `git config user.email` returns — do not
   hardcode one:

   ```
   Co-authored-by: OpenCode <email returned by git config user.email>
   ```

4. Push the changes, unless told otherwise — see the `github-pr` skill for the
   branch rules that decide whether a PR is expected instead.

## Moving changes between branches

When switching to another branch that should carry the current changes, prefer
`git stash` and `git stash pop`:

```bash
git stash
git switch <target-branch>
git stash pop
```

Do not reach for `git commit` plus `git cherry-pick` to do this. Stash keeps the
changes out of the target branch's history, which is almost always what is meant
when transferring work that way.