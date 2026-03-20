---
allowed-tools: Bash(gh issue view:*), Bash(gh search:*), Bash(gh issue list:*), Bash(gh pr diff:*), Bash(gh pr view:*), Bash(gh pr list:*), Bash(git branch:*), Bash(git log:*), Bash(git diff:*), Bash(git blame:*), Bash(git rev-parse:*), Bash(git worktree:*), Bash(ls *), mcp__linear__get_issue, mcp__linear__create_comment, mcp__linear__list_issues
description: Code review a pull request and post findings to the Linear ticket
argument-hint: [TEAM-XXX, PR number, or URL]
disable-model-invocation: false
---

# Code Review

Provide a code review for the given pull request and post the results as a comment on the associated Linear ticket.

## Steps

1. **Resolve working directory and identify the PR:**

   The argument can be a ticket ID (TEAM-XXX), PR number, or PR URL.

   **Priority 1 — Current directory (Conductor workspace or worktree):**
   - Check if the current branch contains the ticket ID (e.g., `team-316` in the branch name)
   - If yes, use the current directory — you're already in the right workspace
   - Get the branch: `git branch --show-current`
   - Then: `gh pr view --json number,headRefName,url`

   **Priority 2 — Ticket ID lookup (if current branch doesn't match):**
   - Check if a worktree exists at `~/worktrees/TEAM-XXX`
   - If not, check `git worktree list` for a worktree with that ticket in the branch name
   - If not, check `~/conductor/workspaces/` for a workspace whose branch matches the ticket
   - Use the first match found as `$WORKDIR`
   - From the match, run: `git -C $WORKDIR branch --show-current` to get the branch
   - Then: `gh pr view --repo <origin-url> <branch-name> --json number,headRefName,url`

   **If a PR URL or number is provided:**
   - Use it directly with `gh pr view`
   - Extract the ticket ID from the PR's branch name

   **If no argument (running from within a workspace):**
   - Detect current branch: `gh pr view --json number,headRefName,url`
   - Extract ticket ID from the branch name

   **Ticket ID extraction:** Look for `TEAM-\d+` (case-insensitive) in the branch name.
   If no ticket ID can be found, ask the user for it.

   Once `$WORKDIR` is resolved, ALL subsequent git commands must target it:
   - `git -C $WORKDIR log ...`
   - `git -C $WORKDIR blame ...`
   - Read files from `$WORKDIR/...`

2. **Eligibility check:**
   Use a Haiku agent to check if the pull request (a) is closed, (b) is a draft, (c) does not need a code review (e.g., automated PR or trivially simple), or (d) already has a code review comment on the Linear ticket from earlier. If so, do not proceed.

3. **Gather CLAUDE.md files:**
   Use a Haiku agent to give you a list of file paths to any relevant CLAUDE.md files from the codebase: the root CLAUDE.md file (if one exists), as well as any CLAUDE.md files in the directories whose files the pull request modified.

4. **Summarize the PR:**
   Use a Haiku agent to view the pull request diff and return a summary of the change.

5. **Multi-agent code review:**
   Launch 5 parallel Sonnet agents to independently code review the change. Each agent returns a list of issues and the reason each was flagged:

   a. **Agent #1 (CLAUDE.md compliance):** Audit changes for CLAUDE.md compliance. Note that CLAUDE.md is guidance for Claude as it writes code, so not all instructions will be applicable during code review.
   b. **Agent #2 (Bug scan):** Read the file changes, do a shallow scan for obvious bugs. Focus on the changes themselves. Focus on large bugs, avoid nitpicks. Ignore likely false positives.
   c. **Agent #3 (History context):** Read the git blame and history of the code modified, to identify any bugs in light of that historical context.
   d. **Agent #4 (Previous PR comments):** Read previous pull requests that touched these files, and check for any comments on those PRs that may also apply to this one.
   e. **Agent #5 (Code comment compliance):** Read code comments in the modified files, and make sure changes comply with any guidance in the comments.

6. **Confidence scoring:**
   For each issue found in #5, launch a parallel Haiku agent that scores the issue 0-100:

   a. **0**: Not confident at all. False positive, doesn't stand up to light scrutiny, or pre-existing issue.
   b. **25**: Somewhat confident. Might be real, might be false positive. If stylistic, not explicitly in CLAUDE.md.
   c. **50**: Moderately confident. Verified real issue, but might be a nitpick or rare in practice.
   d. **75**: Highly confident. Double-checked, very likely real, will be hit in practice. Existing approach is insufficient. Important to functionality or directly mentioned in CLAUDE.md.
   e. **100**: Absolutely certain. Confirmed real issue, will happen frequently. Evidence directly confirms.

   For CLAUDE.md-flagged issues, the agent should verify the CLAUDE.md actually calls out that issue specifically.

7. **Filter:**
   Filter out any issues with a score less than 80. If no issues meet this criteria, post "no issues found" to Linear and stop.

8. **Post to Linear:**
   Use the Linear MCP tools to create a comment on the associated Linear ticket with the review results.

9. **Update ticket status:**
   Move the ticket to "In Review" using Linear MCP tools.

   **Format for issues found:**

   ```markdown
   ### Code Review - PR #[number]

   **Branch:** `[branch-name]`
   **PR:** [PR URL]

   Found [N] issues:

   1. **[brief description]** (confidence: [score]/100)
      - Reason: [CLAUDE.md says "..." / bug due to ... / historical context shows ...]
      - File: `[file-path]` (lines [start]-[end])

   2. **[brief description]** (confidence: [score]/100)
      - Reason: [...]
      - File: `[file-path]` (lines [start]-[end])

   ---
   Generated with Claude Code Review
   ```

   **Format for no issues:**

   ```markdown
   ### Code Review - PR #[number]

   **Branch:** `[branch-name]`
   **PR:** [PR URL]

   No issues found. Checked for bugs and CLAUDE.md compliance.

   ---
   Generated with Claude Code Review
   ```

## False positives to ignore (steps 5 and 6):

- Pre-existing issues
- Something that looks like a bug but is not actually a bug
- Pedantic nitpicks that a senior engineer wouldn't call out
- Issues that a linter, typechecker, or compiler would catch (missing imports, type errors, formatting). Assume CI runs these separately.
- General code quality issues (test coverage, security, documentation), unless explicitly required in CLAUDE.md
- Issues called out in CLAUDE.md but explicitly silenced in code (e.g., lint ignore comments)
- Changes in functionality that are likely intentional or directly related to the broader change
- Real issues on lines the user did not modify in their pull request

## Notes:

- Do not check build signal or attempt to build or typecheck. These run separately in CI.
- Use `gh` to interact with GitHub for fetching PR data
- Use Linear MCP tools (`mcp__linear__create_comment`) to post the review
- Make a todo list first
- Cite specific files and line numbers in each finding
- Always use the resolved working directory for file reads and git commands
