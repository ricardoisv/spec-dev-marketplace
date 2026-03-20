---
allowed-tools: Bash(git log:*), Bash(git diff:*), Bash(git rev-parse:*), Bash(git branch:*), Bash(git worktree:*), Bash(bun run:*), Bash(bun test:*), Bash(make:*), Bash(npm run:*), Bash(pytest:*), Bash(ruff:*), Bash(mypy:*), Bash(ls *), Bash(gh pr view:*), Bash(gh pr diff:*), mcp__linear__get_issue, mcp__linear__create_comment, mcp__linear__list_issues
description: Validate that an implementation plan was correctly executed
argument-hint: [TEAM-XXX or plan path]
disable-model-invocation: false
---

# Validate Plan

Validate that an implementation plan was correctly executed, verifying all success criteria and identifying any deviations or issues. Posts the validation report to the associated Linear ticket.

## Step 0: Resolve Working Directory

The argument can be a ticket ID (TEAM-XXX) or a path to a plan file.

**Priority 1 — Current directory (Conductor workspace or worktree):**
- Check if the current branch contains the ticket ID (e.g., `team-316` in the branch name)
- If yes, use the current directory as the working directory — you're already in the right place
- This is the expected case when running inside a Conductor workspace after `/implement_plan`

**Priority 2 — Ticket ID lookup:**
- If the current branch does NOT match the ticket, check if a worktree exists at `~/worktrees/TEAM-XXX`
- If not there, check `git worktree list` for a worktree with that ticket in the branch name
- If not there, check `~/conductor/workspaces/` for a workspace whose branch matches the ticket
- Use the first match found as the working directory

**If a plan path is provided:**
- Read the plan directly
- Use the current directory as the working directory

**If no argument:**
- Use the current directory
- Extract ticket ID from the branch name (`TEAM-\d+` pattern, case-insensitive)

**Locate the Linear ticket:** Extract `TEAM-\d+` from the branch name or argument.

Once the working directory (`$WORKDIR`) is resolved, ALL subsequent commands must target it:
- `git -C $WORKDIR ...` or run from within `$WORKDIR`
- Read files from `$WORKDIR/...`

## Step 1: Locate the Plan and Contract

1. If plan path was provided, read it
2. Otherwise, search for the plan:
   - Check `thoughts/shared/issues/TEAM-XXXX/` for files matching `*plan*`
   - Check Linear ticket links/comments for plan references
3. If still not found, check recent commits for plan references
4. If no plan found, ask the user
5. **Also search for an issue contract**:
   - Check ticket `links` for a contract document URL
   - Search `thoughts/shared/issues/TEAM-XXXX/` for files matching `*contract*`
   - If found, read it fully — the contract's Acceptance Criteria become the primary definition of "done"

## Step 2: Context Discovery

1. **Read the implementation plan** completely
2. **Identify what should have changed**:
   - List all files that should be modified
   - Note all success criteria (automated and manual)
   - Identify key functionality to verify

3. **Gather implementation evidence**:
   ```bash
   # Check recent commits
   git -C $WORKDIR log --oneline -n 20

   # See what changed vs the base branch
   git -C $WORKDIR diff main..HEAD --stat
   git -C $WORKDIR diff main..HEAD
   ```

4. **Spawn parallel research tasks** to discover implementation:
   - Task 1: Verify all files listed in the plan were modified
   - Task 2: Compare actual code changes to plan specifications
   - Task 3: Check if tests were added/modified as specified

## Step 3: Run Automated Verification

Auto-detect the project type and run appropriate checks:

- If `bun.lockb` exists → `cd $WORKDIR && bun run lint`
- If `Makefile` with `check`/`test` targets → `cd $WORKDIR && make check test`
- If `package.json` with lint/test scripts → `cd $WORKDIR && npm run lint && npm test`
- If `pytest` available → `cd $WORKDIR && pytest`
- If `ruff` available → `cd $WORKDIR && ruff check`
- If `mypy` available → `cd $WORKDIR && mypy .`

Also run any specific verification commands mentioned in the plan's "Automated Verification" or "Success Criteria" sections.

## Step 4: Systematic Validation

### Contract Compliance (if contract exists)

For each acceptance criterion in the contract:
1. **Verify the criterion explicitly** — run the verification command, check the observable output, or confirm the condition
2. **Record pass/fail** for each criterion
3. Report contract criteria status separately from plan phase status

### Plan Phase Verification

For each phase/step in the plan:

1. **Check completion status**:
   - Verify the actual code matches what the plan specified
   - Look for any TODO/FIXME comments left in the code

2. **Assess code quality**:
   - Does the implementation follow existing patterns in the codebase?
   - Are there any regressions?
   - Is error handling robust?

3. **Think deeply about edge cases**:
   - Were error conditions handled?
   - Are there missing validations?
   - Could the implementation break existing functionality?
   - Are there any mocked tests that should test actual functions?
   - Are there disabled/skipped tests that should be running?

## Step 5: Generate and Post Validation Report

Create a validation report and post it as a **Linear comment** on the ticket:

```markdown
### Validation Report

**Branch:** `[branch-name]`
**Working directory:** `[resolved path]`
**Plan:** `[plan-path]`

#### Implementation Status
- [x] [Phase/Step 1 name] - Fully implemented
- [x] [Phase/Step 2 name] - Fully implemented
- [ ] [Phase/Step 3 name] - Partially implemented (see issues)

#### Automated Verification
- [pass/fail] Lint: `[command run]`
- [pass/fail] Tests: `[command run]`
- [pass/fail] Build: `[command run]`

#### Contract Compliance (if contract exists)
- [x/fail] [Functional criterion 1]
- [x/fail] [Functional criterion 2]
- [x/fail] [Regression criterion 1]
- [x/fail] [Quality criterion 1]

#### Matches Plan
- [What was correctly implemented]

#### Deviations from Plan
- [Any differences between plan and implementation]
- [Extra things added not in plan]
- [Things in plan not implemented]

#### Potential Issues
- [Any concerns found during review]

#### Manual Testing Required
1. [ ] [Step to verify manually]
2. [ ] [Step to verify manually]

#### Recommendations
- [Action items before merge]

---
Generated with Claude Code - Plan Validation
```

Use `mcp__linear__create_comment` to post this to the Linear ticket.

After posting the validation report, move the ticket to "Validated" using Linear MCP tools.

## Important Guidelines

1. **Be thorough but practical** - Focus on what matters
2. **Run all automated checks** - Don't skip verification commands
3. **Use the resolved working directory** - Never run commands against the wrong repo
4. **Document everything** - Both successes and issues
5. **Think critically** - Question if the implementation truly solves the problem
6. **Post to Linear** - Always post the validation report as a comment on the ticket
7. **Be honest** - Flag shortcuts, incomplete items, or deviations clearly

## Validation Checklist

Always verify:
- [ ] All phases marked in the plan are actually done
- [ ] Linter passes
- [ ] No regressions introduced
- [ ] Code follows existing patterns in the codebase
- [ ] Error handling is robust
- [ ] No disabled or mocked-out tests that should be real
- [ ] Implementation matches the plan's success criteria

## Workflow Integration

This command fits into the workflow:
1. `/issue_contract` - Understand intent, define "done"
2. `/research` - Research the ticket (scoped by contract)
3. `/plan` - Create the implementation plan
4. `/implement_plan` - Execute the plan
5. **`/validate_plan`** - Verify the implementation (this command)
6. `/code-review` - Multi-agent code review
7. `/commit` - Create atomic commits
