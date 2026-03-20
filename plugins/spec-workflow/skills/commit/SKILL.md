---
description: Skill for creating git commits with clear, focused commit messages. Always follow these guidelines for commits.
alwaysApply: true
---

# Commit Changes Skill

You are skilled at creating clean, well-organized git commits for changes made during a session.

## When to Apply

This skill activates when the user asks to commit changes, save work, or create a git commit.

## Process

1. **Analyze what changed:**
   - Review the conversation history and understand what was accomplished
   - Run `git status` to see current changes
   - Run `git diff` to understand the modifications
   - Consider whether changes should be one commit or multiple logical commits

2. **Plan your commit(s):**
   - Identify which files belong together logically
   - Draft clear, descriptive commit messages
   - Use imperative mood in commit messages (e.g., "Add feature" not "Added feature")
   - Focus on why the changes were made, not just what

3. **Execute:**
   - Use `git add` with specific files (never use `-A` or `.`)
   - Create commits with your planned messages
   - Show the result with `git log --oneline -n [number]`

## Important Rules

- **NEVER add co-author information or Claude attribution**
- Commits should be authored solely by the user
- Do not include any "Generated with Claude" messages
- Do not add "Co-Authored-By" lines
- Write commit messages as if the user wrote them

## Best Practices

- You have the full context of what was done in this session
- Group related changes together
- Keep commits focused and atomic when possible
- The user trusts your judgment - they asked you to commit
- Separate refactoring from feature changes when practical
