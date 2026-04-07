# Global Rules

## Waiting

To wait for a certain time, run `sleep` with the desired time.

## Interacting with GitHub

All GitHub interaction should be made with the `gh` CLI, such as:
- `gh pr create`
- `gh pr merge`
- `gh pr comment`
- `gh pr checks`
- `gh repo clone`
- `gh repo create`
- etc.

## Changes

All changes must adhere to a CONTRIBUTING.md file (or similar) if it exists in the project root.

All changes must be commited granularly to adhere to git best practices. If a commit can be split, split it.

When switching to another branch that you want to transfer current changes to, prefer git stash over git commit and git cherry-pick.

## Git Commit Author

When making git commits, ALWAYS use the `--author` flag:

1. First run: `git config user.email` to get the current e-mail
2. Add "Co-authored-by: OpenCode <email you just got>" to your commit message.
3. Unless specified otherwise, push changes (check Pushing to Remote section).

## Pushing to Remote

If a remote exists, always push unless specified otherwise. If the change is non-trivial or makes noticeable changes in the application, create a new branch before committing and open a PR instead of committing directly to main. This does not apply to GitHub workflow changes or configuration-only changes that don't affect the resulting library or application (e.g., AGENTS.md), as the PR is mainly for the automated changelog.

## Merging a PR

When merging a PR, use `gh pr merge` while still on the PR branch. After merging, switch to the main branch (if not already on it), pull from remote, and unless specified otherwise, delete both the local and remote PR branch.

## Interacting with CodeRabbit

When asked to "Handle CodeRabbit comments", do the following:

On GitHub PRs, @coderabbitai will comment with a summary and walkthrough of changes. It will also review those changes.
Major comments are posted as actual GitHub code review comments, whereas nitpicks and duplicates (that still apply) are posted as conversation comments.

CodeRabbit has a very small rate limit on OSS projects - it will edit the walkthrough comment with a "Rate limit exceeded" warning with a wait time.
When asked to check for reviews and see this warning, you should wait for that time, and (since CodeRabbit doesn't auto-review after this time) comment "@coderabbitai review" to request a review.

This review can take roughly 5 minutes, if you don't see a review after that time, wait another 3 minutes.

CodeRabbit will also pause reviews when there are a lot of commits on the branch (indicated with an edit on the walkthrough comment). Comment "@coderabbit continue" and (in a new comment) "@coderabbit review" to request a review.

If neither edits on the walkthrough comments are applied and the CodeRabbit check shows completed instead of pending, then no more comments are to be made and the review is finished (take this as a "LGTM" message.)
DO NOT MERGE PR UNTIL I APPROVE.
