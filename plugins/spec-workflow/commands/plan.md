---
description: Conductor-native planning workflow for Linear tickets - creates implementation plans based on research
argument-hint: [ticket-number]
---

# Planning Workflow

Conductor-native planning workflow. Fetches a Linear ticket with completed research, creates an implementation plan, and updates ticket status. Works directly in the current Conductor workspace.

## PART I - LOCATE THE TICKET AND RESEARCH

### If a ticket number is provided:

1. Use `linear` MCP (`mcp__linear__get_issue`) to fetch the ticket and save it to `./thoughts/shared/issues/TEAM-xxxx/ticket.md`
2. Read the ticket and use `mcp__linear__list_comments` to read all comments
3. Read all linked documents from the ticket's `links` section — this should include the research document from `/research`
4. If no research document is linked, search `thoughts/shared/issues/TEAM-XXXX/` for a file matching `*research*`
5. **Check for an issue contract**:
   - Check ticket `links` for a contract document URL
   - Search `thoughts/shared/issues/TEAM-XXXX/` for a file matching `*contract*`
   - If found, read it fully — the contract's Locked Objective defines the target end state, and its Acceptance Criteria are the outcomes the plan must satisfy
6. If a plan document already exists (in links or `thoughts/shared/issues/TEAM-XXXX/`), EXIT with a link to the existing plan — do not create a duplicate

### If no ticket is provided:

7. If user provides a specific research document or topic, proceed with that context.
8. If the user provided neither a ticket nor context, EXIT IMMEDIATELY and inform the user.

## PART II - PREPARE TO PLAN

think deeply about the research findings and what needs to be planned

9. Read the research document fully. Understand every finding, constraint, and cross-component connection.

10. Move the ticket to "Plan" using Linear MCP tools (skip if no ticket).

11. Conduct any additional targeted research needed to fill gaps:
    - Spawn **codebase-analyzer** agents for specific components that the research identified but didn't deep-dive
    - Spawn **codebase-pattern-finder** agents to find similar features to model the implementation after
    - Only do this if the research document has explicit open questions or gaps

think deeply about the implementation approach

## PART III - CREATE THE PLAN

12. Write the implementation plan to `thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-plan-description.md`:
    - YYYY-MM-DD is today's date
    - TEAM-XXXX is the ticket number (omit if no ticket)
    - description is a brief kebab-case description

13. Use this structure:

```markdown
# [Feature/Task Name] Implementation Plan

## Overview
[Brief description of what we're implementing and why]

## Current State Analysis
[What exists now, what's missing, key constraints — reference the research document]

## Desired End State
[If a contract exists, draw this from the contract's Locked Objective. Otherwise, specify the desired end state and how to verify it.]

## What We're NOT Doing
[Explicitly list out-of-scope items to prevent scope creep]

## Implementation Approach
[High-level strategy and reasoning for the chosen approach]

## Testing Strategy
[TDD approach — what tests to write first, key edge cases]

## Phase 1: [Descriptive Name]

### Overview
[What this phase accomplishes]

### Changes Required:
#### 1. [Component/File Group]
**File**: `path/to/file.ext`
**Changes**: [Summary of changes]

### Success Criteria:
#### Automated Verification:
- [ ] Tests pass
- [ ] Type checking passes
- [ ] Linting passes

#### Manual Verification:
- [ ] [Specific verification step]

---

## Phase 2: [Descriptive Name]
[Similar structure...]

## References
- Original ticket: `thoughts/shared/issues/TEAM-XXXX/ticket.md`
- Contract: `thoughts/shared/issues/TEAM-XXXX/[relevant-contract].md` (if exists)
- Research: `thoughts/shared/issues/TEAM-XXXX/[relevant-research].md`
```

14. Keep phases small and independently verifiable. Each phase should be completable and testable on its own.

15. If an issue contract exists, ensure every contract acceptance criterion is traceable to at least one plan phase's Success Criteria. Add implementation-specific criteria (linter, type checker, pattern compliance) on top of the contract criteria — these are the plan's addition, not the contract's.

16. Never commit any code changes at this stage — this is planning only.

## PART IV - SYNC AND UPDATE

17. Sync the plan:
    ```bash
    thoughts sync -m "Add implementation plan for TEAM-XXXX: [brief description]"
    ```

18. Write plan context to `.context/notes.md`:
    ```bash
    mkdir -p .context
    ```
    - Include: ticket ID, plan file path, phase overview, key decisions made
    - This makes the plan visible in the Conductor UI

19. If a ticket exists:
    a. Attach the plan document to the ticket using `mcp__linear__save_issue` with proper link formatting (GitHub URL: `https://github.com/your-org/thoughts/blob/main/shared/issues/TEAM-XXXX/YYYY-MM-DD-plan-description.md`)
    b. Add a terse comment via `mcp__linear__create_comment` with a link to the plan
    c. Leave the ticket in "Plan" status (the next workflow step will advance it)

20. Print a summary for the user (replace placeholders with actual values):

```
Plan complete for TEAM-XXXX: [ticket title]

Plan document: thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-plan-description.md

Approach: [selected approach description]

Implementation phases:
- Phase 1: [phase 1 description]
- Phase 2: [phase 2 description]
- Phase 3: [phase 3 description if applicable]

Next steps:
- Review the plan document
- Run /implement_plan TEAM-XXXX to implement it in this workspace

View the ticket: https://linear.app/your-org/issue/TEAM-XXXX/[ticket-slug]
```
