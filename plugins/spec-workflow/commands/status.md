---
allowed-tools: Bash(git rev-parse:*), Bash(git branch:*), Bash(ls *), mcp__linear__get_issue
description: Check a ticket's pipeline stage and see the next command to run
argument-hint: [TEAM-XXX]
---

# Status

Show where a ticket is in the spec-driven development pipeline and recommend the next command to run.

## Step 1: Resolve Ticket ID

1. If a ticket ID argument was provided (e.g., `PROD-625`), use it directly.
2. If no argument was provided, extract the ticket ID from the current git branch name using the pattern `TEAM-\d+` (case-insensitive). Run:
   ```bash
   git rev-parse --abbrev-ref HEAD
   ```
   Then extract the first match of `[A-Za-z]+-\d+` from the branch name.
3. If no ticket ID can be determined, tell the user: "Could not determine ticket ID. Usage: `/status TEAM-XXX` or run from a branch with a ticket ID."

## Step 2: Fetch Ticket from Linear

4. Use `mcp__linear__get_issue` to fetch the ticket by its ID.
5. Record the ticket's `title` and `status` field.
6. If the ticket cannot be found, report the error and stop.

## Step 3: Scan Local Artifacts

7. Check for local artifacts in `thoughts/shared/issues/TEAM-XXX/` (where `TEAM-XXX` is the ticket ID):
   - Look for files matching `*contract*` (excluding `*handoff*`)
   - Look for files matching `*research*` (excluding `*handoff*`)
   - Look for files matching `*plan*` (excluding `*handoff*`)
8. Record which artifact types are present. These provide supplementary context — Linear status is authoritative.

## Step 4: Determine Pipeline Stage

9. Map the Linear ticket status to a pipeline stage using this table:

| Linear Status | Stage | Stage # | Next Command |
|--------------|-------|---------|--------------|
| Triage | Pre-pipeline | 0 | `/issue_contract TEAM-XXX` |
| Backlog | Pre-pipeline | 0 | `/issue_contract TEAM-XXX` |
| Todo | Pre-pipeline | 0 | `/issue_contract TEAM-XXX` |
| Contract | Contract | 1 | `/research TEAM-XXX` |
| Research | Research | 2 | `/plan TEAM-XXX` |
| Plan | Plan | 3 | `/implement_plan TEAM-XXX` |
| Implemented | Implemented | 4 | `/validate_plan TEAM-XXX` |
| Validated | Validated | 5 | `/code-review TEAM-XXX` |
| In Review | In Review | 6 | Create a PR |
| Done | Done | — | Complete |
| Canceled | Canceled | — | Complete |
| Duplicate | Duplicate | — | Complete |

Replace `TEAM-XXX` with the actual ticket ID in the next command suggestion.

## Step 5: Output

10. Print the status in this exact format:

```
TEAM-XXX: [ticket title]

Stage: [stage name] ([stage #]/6)
Next step: [next command]

Pipeline:
  [marker] Contract
  [marker] Research
  [marker] Plan
  [marker] Implement
  [marker] Validate
  [marker] Review
```

Where `[marker]` is:
- `[x]` for completed stages (stage # less than current)
- `[>]` for the current stage, followed by `    <-- current`
- `[ ]` for future stages

For pipeline stages, use these display labels and corresponding stage numbers:
- Contract = 1
- Research = 2
- Plan = 3
- Implement = 4
- Validate = 5
- Review = 6

**Special cases:**
- If the status is **pre-pipeline** (Triage/Backlog/Todo), show all stages as `[ ]` and display: `Stage: Pre-pipeline (0/6)` with next step `/issue_contract TEAM-XXX`
- If the status is **Done**, show all stages as `[x]` and display: `Stage: Done` with next step `Complete — no further action needed`
- If the status is **Canceled** or **Duplicate**, show: `Stage: [status]` with next step `Complete — ticket is [status]`

**Artifact cross-reference:**
- After the pipeline checklist, if any local artifacts were found in Step 3, add a brief note:
  ```
  Local artifacts: contract, research, plan
  ```
  (list only the types that were found)
- If no artifacts directory exists or no artifacts were found, omit this line entirely.

ARGUMENTS: %ARGUMENTS%
