---
name: coderabbit
description: Use when asked to handle CodeRabbit comments, check on a CodeRabbit review, or find out whether CodeRabbit has finished reviewing a pull request. Covers polling the walkthrough comment, waiting out rate limits, and re-triggering reviews that CodeRabbit paused.
---

# CodeRabbit

Drives the wait-and-poll loop for getting CodeRabbit's review to completion on a
GitHub pull request, and decides when the review counts as finished.

**Do not merge the PR until the user approves.** Handling the comments and
merging are two separate things. Stop after the review is done and report.

## Poll with a large comment count

CodeRabbit posts several comments per PR, and the walkthrough comment is
created before the review lands. Always request at least 1000 comments:

```bash
gh pr view <pr> --comments
gh api repos/<owner>/<repo>/issues/<pr>/comments --paginate
```

Read the whole list before concluding anything is missing.

## The timeline

1. CodeRabbit comments with a summary and walkthrough of the changes, then
   reviews those changes. **This takes a while.** `sleep` for a couple of
   minutes before checking — an immediate poll reads as "no review yet" when
   the review has simply not started.
2. Major comments arrive as actual GitHub code review comments. Nitpicks and
   duplicates that still apply arrive as conversation comments. Handle both;
   a conversation comment is not automatically ignorable.

## When there is no walkthrough comment

| What you see                                                        | What to do                                                             |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| No comments at all                                                   | `sleep` for another 3 minutes and check again                           |
| Other comments, but no walkthrough comment                          | Raise the comment count and check again                                 |
| Walkthrough comment shows a rate limit                              | Wait the stated time, then comment `@coderabbitai review`               |

CodeRabbit has a very small rate limit on OSS projects — it edits the walkthrough
comment with a "Rate limit exceeded" warning and a wait time. CodeRabbit does not
auto-review after that time passes, so the comment is what restarts it.

## Waiting for the review

A review takes roughly 5 minutes. If no review has appeared after that, wait
another 3 minutes and check again.

CodeRabbit also **pauses** reviews when a branch has a lot of commits, which shows
up as an edit to the walkthrough comment. Neither plain waiting nor `@coderabbitai
review` fixes a paused review:

```bash
gh pr comment <pr> --body "@coderabbit continue"
gh pr comment <pr> --body "@coderabbitai review"
```

The first resumes the paused review, the second requests one. Send both as
separate comments.

## The review is finished

Only declare the review done when **all** of these hold:

- Neither edits on the walkthrough comment are applied,
- and the CodeRabbit check shows **completed** rather than pending,
- and the walkthrough comment shows no rate-limit or similar message.

Take that as an LGTM. Make no further CodeRabbit comments and stop polling.

If any one of the three fails, it is not finished: go back to
[Waiting for the review](#waiting-for-the-review) and keep polling.