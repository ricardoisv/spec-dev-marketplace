---
description: Implementation workflow for Linear tickets - executes approved plans (Conductor-native)
argument-hint: [ticket-number]
---

# Implement Plan Workflow

Conductor-native implementation workflow. Fetches a Linear ticket with an approved plan, sets up the current workspace, and implements the plan directly — no subprocess spawning.

## PART I - LOCATE THE PLAN

### If a ticket number is provided:

1. Use `linear` MCP (`mcp__linear__get_issue`) to fetch the ticket and save it to `./thoughts/shared/issues/TEAM-xxxx/ticket.md`
2. Read the ticket and use `mcp__linear__list_comments` to read all comments
3. Identify the linked implementation plan document:
   a. Check the ticket's `links` section for a plan document URL
   b. Check comments for a plan document link
   c. Search `thoughts/shared/issues/TEAM-XXXX/` for a file matching `*plan*`
4. If no plan was found in any of these locations, EXIT IMMEDIATELY with an explanation — do NOT move the ticket status.
5. **Check for an issue contract**:
   - Check ticket `links` for a contract document URL
   - Search `thoughts/shared/issues/TEAM-XXXX/` for a file matching `*contract*`
   - If found, read it — the contract's Acceptance Criteria must be verified before the implementation is considered complete

### If no ticket is provided:

6. If user explicitly requests to implement a specific plan, search for it in `thoughts/shared/issues/` and use it.
7. If no plan document exists, EXIT IMMEDIATELY and inform the user.
8. If the user does not mention a ticket number or a specific plan, EXIT IMMEDIATELY and inform the user.

## PART II - PREPARE THE WORKSPACE

think deeply about whether the plan is complete and implementable

9. Read the full implementation plan document. Understand every phase, every file to modify, and every acceptance criterion.

10. Now that the plan is confirmed to exist and is readable, proceed with implementation. (The ticket stays in "Plan" status until completion, then moves to "Implemented".)

11. Rename the current branch to match the ticket:
    ```bash
    git branch -m <prefix>/team-XXXX-[ticket-slug]
    ```
    - Derive `<prefix>` from the current branch name's prefix (the part before the first `/`), or fall back to `git config user.name | tr ' ' '-' | tr '[:upper:]' '[:lower:]'`
    - If the branch already has a name matching the ticket, skip this step
    - If the target branch name already exists, append a short suffix (e.g., `-v2`)

12. Ensure dependencies are installed (auto-detect):
    - If `bun.lockb` exists → `bun install`
    - Else if `Makefile` with `setup` target → `make setup`
    - Else if `package-lock.json` exists → `npm install`
    - Else if `package.json` exists → `npm install`
    - Skip if `node_modules/` (or equivalent) already exists and lockfile hasn't changed

13. Write context for this workspace:
    ```bash
    mkdir -p .context
    ```
    - Write a summary of the ticket and plan to `.context/notes.md` — include: ticket ID, title, plan file path, key implementation steps
    - Write pre-merge verification items to `.context/todos.md` (e.g., `- [ ] Tests pass`, `- [ ] Plan criteria met`)

## PART III - IMPLEMENT THE PLAN

think deeply about the implementation approach, use TodoWrite to track your tasks.

14. Follow the implementation plan phase by phase:
    - For each phase, implement the changes described
    - Run tests after each phase to catch regressions early
    - If the plan specifies test-driven development, write tests first
    - If a phase depends on a previous phase, verify the dependency is satisfied before proceeding
    - If the plan asks for tests, use a sub-agent to write those tests. This agent must be a separate agent to the one implementing the other changes.

15. After all phases are complete:
    - Run the full test suite (auto-detect: `bun.lockb` → `bun test`, `Makefile` → `make test`, `package.json` → `npm test`, `pytest`, `ruff`, `mypy`)
    - Run linting if available
    - If an issue contract exists, verify each contract acceptance criterion. If any criterion fails, fix before proceeding.
    - Fix any failures before proceeding

16. Create a commit following the commit skill guidelines:
    - Group changes logically
    - Use imperative mood
    - `git add` specific files (never `-A` or `.`)
    - Commit directly without asking for confirmation — this command is user-initiated and runs end-to-end
    - Never mention co-authoring in the commit message

17. Update `.context/todos.md` — check off completed items

## PART IV - POST-IMPLEMENTATION

18. Move the ticket to "Implemented" using Linear MCP tools.

19. Add a comment to the Linear ticket via `mcp__linear__create_comment`:
    - Summarize what was implemented
    - List any deviations from the plan
    - If an issue contract exists, include contract compliance status (which criteria passed/failed)
    - Note: "PR will be created via Conductor"

20. Print a summary for the user (replace placeholders with actual values):

```
Implementation complete for TEAM-XXXX: [ticket title]

Branch: [current branch name]
Commit: [short commit hash] - [commit message]

Contract compliance: [if contract exists: "All N criteria passed" or "N/M criteria passed — see details above"; if no contract: "No contract found"]

Changes are ready. Next steps:
- Review the diff in Conductor (Cmd+D)
- Create a PR via Conductor (Cmd+Shift+P)
- Run /validate_plan TEAM-XXXX to verify against plan and contract criteria

View the ticket: https://linear.app/your-org/issue/TEAM-XXXX/[ticket-slug]
```
