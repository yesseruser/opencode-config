# Global Rules

## Git Commit Author

When making git commits, ALWAYS use the `--author` flag:

1. First run: `git config user.email` to get the current e-mail
2. Then use: `git commit --author="yesseruser (OpenCode) <email-from-step-1>" -m "your message"`

## Pushing to Remote

If a remote exists, always push unless specified otherwise. If the change is non-trivial or makes noticeable changes in the application, create a new branch before committing and open a PR instead of committing directly to main.
