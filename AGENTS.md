# Global Rules

## Git Commit Author

When making git commits, ALWAYS use the `--author` flag:

1. First run: `git config user.name` to get the current name
2. Then use: `git commit --author="yesseruser (OpenCode) <email-from-step-1>" -m "your message"`

After committing, restore your original user.name:
  git config user.name "yesseruser (OpenCode)"
