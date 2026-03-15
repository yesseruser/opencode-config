# Global Rules

## Changes

All changes should be commited (see next section).
When switching to another branch that you want to transfer current changes to, prefer git stash over git commit and git cherry-pick.

## Git Commit Author

When making git commits, ALWAYS use the `--author` flag:

1. First run: `git config user.email` to get the current e-mail
2. Then use: `git commit --author="yesseruser (OpenCode) <email-from-step-1>" -m "your message"
3. Unless specified otherwise, push changes (check Pushing to Remote section).

## Pushing to Remote

If a remote exists, always push unless specified otherwise. If the change is non-trivial or makes noticeable changes in the application, create a new branch before committing and open a PR instead of committing directly to main. This does not apply to GitHub workflow changes or configuration-only changes that don't affect the resulting library or application (e.g., AGENTS.md), as the PR is mainly for the automated changelog.

## Merging a PR

When merging a PR, use `gh pr merge` while still on the PR branch. After merging, switch to the main branch (if not already on it), pull from remote, and unless specified otherwise, delete both the local and remote PR branch.
