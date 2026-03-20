---
description: Set up the current workspace to review a colleague's branch
argument-hint: [gh_username:branchName or PR-URL]
---

# Local Review

Fetch a colleague's branch into the current Conductor workspace and prepare it for code review. Works directly in the current workspace — no separate worktree creation needed.

## Process

### 1. Parse the input

The argument can be:
- `gh_username:branchName` (e.g., `colleague:username/team-316-add-api-endpoint`)
- A GitHub PR URL (e.g., `https://github.com/your-org/repo/pull/42`)
- A PR number (e.g., `42`)

**If `gh_username:branchName`:**
- Extract GitHub username and branch name

**If a PR URL or number:**
- Run `gh pr view <arg> --json headRefName,headRepository,headRepositoryOwner,number,url`
- Extract the branch name and fork owner from the response

**If no parameter provided:**
- Ask for it: "Please provide a `gh_username:branchName`, PR URL, or PR number"

### 2. Extract ticket information

- Look for ticket numbers in the branch name (e.g., `team-316`, `TEAM-316`)
- This will be used to link the review to the Linear ticket

### 3. Fetch the branch into the current workspace

```bash
# Get repo info
REPO_NAME=$(basename $(git rev-parse --show-toplevel))

# Add the colleague's fork as a remote (if not already present)
git remote -v | grep -q USERNAME || git remote add USERNAME git@github.com:USERNAME/REPO_NAME

# Fetch their branch
git fetch USERNAME BRANCHNAME

# Check out their branch in the current workspace
git checkout USERNAME/BRANCHNAME
```

If the checkout fails because of uncommitted changes, inform the user and stop.

### 4. Install dependencies

Auto-detect and install:
- If `bun.lockb` exists → `bun install`
- Else if `Makefile` with `setup` target → `make setup`
- Else if `package-lock.json` exists → `npm install`
- Else if `package.json` exists → `npm install`

### 5. Report and suggest next steps

Print:
```
Review environment ready!

Branch: USERNAME/BRANCHNAME
Ticket: TEAM-XXX (if found)
PR: #[number] (if available)

The current workspace is now on the colleague's branch.

Next steps:
- Review the diff: Cmd+D in Conductor
- Run code review: /code-review TEAM-XXX
- Run tests manually if needed

To return to your previous branch:
  git checkout [previous-branch-name]
```

## Error Handling

- If remote already exists with a different URL, inform the user
- If fetch fails, check if the username/repo/branch exists and suggest corrections
- If checkout fails due to local changes, suggest stashing or committing first
- If dependency install fails, report the error but continue

## Example Usage

```
/local_review colleague:username/team-316-add-api-endpoint
/local_review https://github.com/your-org/your-app/pull/42
/local_review 42
```
