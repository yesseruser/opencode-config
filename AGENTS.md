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

All changes must adhere to a CONTRIBUTING.md file and AGENTS.md (or similar files) if they exist in the project root.

All changes must be commited granularly to adhere to git best practices. If a commit can be split into smaller chunks that are still meaningful, split it.

When switching to another branch that you want to transfer current changes to, prefer `git stash` and `git stash pop` over `git commit` and `git cherry-pick`.

## Git Commit Author

When making git commits:

0. Check current commit message style with `git log` and adhere to it for the summary;
   your commit messages should ALWAYS have a highly descriptive description.
1. Add "Co-authored-by: OpenCode <email returned by `git config user.email`>" to your commit message.
2. Unless specified otherwise, push changes (check Pushing to Remote section).

## Pushing to Remote

If a remote exists **AND _NOT_ ON THE MAIN BRANCH**, always push unless specified otherwise. If the change is non-trivial or makes noticeable changes in the application, create a new branch before committing and open a PR instead of committing directly to main. This does not apply to GitHub workflow changes or configuration-only changes that don't affect the resulting library or application (e.g., AGENTS.md), as the PR is mainly for the automated changelog.

## Merging a PR

When merging a PR, use `gh pr merge --merge --delete-branch` while still on the PR branch. This will also delete the current branch and check out the merged master branch.

## Interacting with CodeRabbit

When asked to "Handle CodeRabbit comments", do the following:

On GitHub PRs, @coderabbitai will comment with a summary and walkthrough of changes. It will also review those changes. This comment takes a while to appear - run `sleep` for a couple minutes.
Major comments are posted as actual GitHub code review comments, whereas nitpicks and duplicates (that still apply) are posted as conversation comments.

CodeRabbit has a very small rate limit on OSS projects - it will edit the walkthrough comment with a "Rate limit exceeded" warning with a wait time.
When asked to check for reviews and see this warning, you should wait for that time, and (since CodeRabbit doesn't auto-review after this time) comment "@coderabbitai review" to request a review.

This review can take roughly 5 minutes, if you don't see a review after that time, wait another 3 minutes.

CodeRabbit will also pause reviews when there are a lot of commits on the branch (indicated with an edit on the walkthrough comment). Comment "@coderabbit continue" and (in a new comment) "@coderabbit review" to request a review.

If neither edits on the walkthrough comments are applied and the CodeRabbit check shows completed instead of pending, **and if the walkthrough comment doesn't show a rate-limit or similar message**, then no more comments are to be made and the review is finished (take this as a "LGTM" message.)
DO NOT MERGE PR UNTIL I APPROVE.

When polling GitHub comments, always specify a large comment count (1000 or more), if you don't see a CodeRabbit walkthrough comment but you see other comments, you need to increase (if you don't see other comments, wait another 3 minutes)
