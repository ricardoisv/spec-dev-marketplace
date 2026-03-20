---
description: Understands user intent for a Linear issue and creates a verifiable completion contract — defines what "done" looks like before research or planning begins
argument-hint: [ticket-number]
---

# Issue Contract

Captures user intent for a Linear issue and produces a verifiable completion contract. The contract defines **what "done" looks like** — outcome-focused acceptance criteria that every downstream step (research, planning, implementation, validation) must satisfy.

This is the first step in the pipeline. Do NOT read CLAUDE.md or source files — the contract captures intent, not implementation details.

## PART I — FETCH THE TICKET

### If a ticket number is provided:

1. Use `linear` MCP (`mcp__linear__get_issue`) to fetch the ticket and save it to `./thoughts/shared/issues/TEAM-xxxx/ticket.md`
2. Read the ticket and use `mcp__linear__list_comments` to read all comments
3. Read any linked documents from the ticket's `links` section to understand existing context
4. Move the ticket to "Contract" using Linear MCP tools

### If no ticket is provided:

5. Ask the user: *"Which Linear issue should I create a contract for?"*
6. A contract always needs a ticket anchor — do not proceed without one.

## PART II — UNDERSTAND USER INTENT

This is the contract's primary job. Do NOT rush past it.

think deeply about the user's intent — what are they trying to achieve? What does success look like from their perspective? Think about outcomes, not implementation. Ask any questions needed to fully understand the user's intent.

### Step 2a: Extract from the ticket

Identify:
- **The Problem**: What is broken, missing, or suboptimal?
- **Explicit acceptance criteria**: Did the author specify what "done" looks like?
- **Reproduction steps**: How does the current (broken) behavior manifest?
- **Context from comments**: Have team members added clarifications or constraints?

### Step 2b: Analyze the intent

Based on the ticket, determine the intended outcome:
- What is the user trying to achieve?
- What does success look like from their perspective?
- What are the natural scope boundaries?

Do NOT think about implementation. Think about outcomes.

### Step 2c: Propose or ask

**If you can confidently infer the target state:**
Present a proposal to the user:

```
## Objective Proposal

**Problem:** [what's broken/missing — one sentence]
**Target State:** [how it should look/behave after — one sentence]
**Scope:** [what's in / what's explicitly out]

Does this match your intent? (yes / adjust)
```

**If you CANNOT confidently infer the target state:**
Ask focused clarifying questions. Be specific, not open-ended:

```
I need clarification to define the objective:
1. [Specific question about expected behavior — e.g., "What should happen when a user submits an empty form?"]
2. [Specific question about scope — e.g., "Should this also handle the admin view, or just the user-facing side?"]
3. [Specific question about edge cases if relevant]
```

Maximum 3 questions per round. Questions should probe intent ("What should happen when X?"), not implementation ("Should we use library Y?").

### Step 2d: Lock the objective

Do NOT proceed until the user explicitly confirms. Once confirmed:

```
## Locked Objective

**Issue:** [ID] - [Title]
**Problem:** [one-liner]
**Target State:** [one-liner]
**Scope:** [what's in / what's out]
```

## PART III — DEFINE ACCEPTANCE CRITERIA

think deeply about what deterministic, verifiable criteria would prove this issue is resolved

Outcome-focused, deterministic, verifiable criteria. No codebase knowledge needed — these come from user intent.

7. Write criteria in three categories:

   **Functional** — specific outcome behaviors that must work:
   - e.g., "API returns 200 for valid payload"
   - e.g., "User sees confirmation message after submit"
   - e.g., "Data appears in the dashboard within 5 seconds of creation"

   **Regression** — existing behaviors that must NOT break:
   - e.g., "Existing login flow still works"
   - e.g., "No data loss during the operation"
   - e.g., "Other API endpoints return the same responses as before"

   **Quality** — observable standards:
   - e.g., "No console errors in the browser"
   - e.g., "Page loads in < 2 seconds"
   - e.g., "Error messages are actionable (tell the user what to do)"

8. Use **neutral language** — write what should be true, not "verify there are no bugs." This avoids sycophantic over-reporting.

9. Each criterion must be verifiable by running a command, observing output, or checking a specific condition. If it can't be verified, it shouldn't be in the contract.

10. Include an **Out of Scope** section to prevent drift. The contract is a wall, not a wishlist.

11. Present the criteria to the user for confirmation:

```
## Proposed Acceptance Criteria

### Functional
- [ ] [criterion 1]
- [ ] [criterion 2]

### Regression
- [ ] [criterion 1]

### Quality
- [ ] [criterion 1]

### Out of Scope
- [exclusion 1]
- [exclusion 2]

Do these criteria capture what "done" means for this issue? (yes / adjust)
```

12. Adjust until the user agrees. Do NOT proceed until confirmed.

## PART IV — SAVE AND SYNC

13. Gather metadata:
    ```bash
    date -Iseconds
    ```

14. Write the contract to `thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-contract-description.md`:
    - YYYY-MM-DD is today's date
    - TEAM-XXXX is the ticket number
    - description is a brief kebab-case description of the objective

15. Use this structure:

```markdown
# Contract: TEAM-XXXX — [Title]

## Locked Objective

**Problem:** [one-liner]
**Target State:** [one-liner]
**Scope:** [in / out]

## Acceptance Criteria

### Functional
- [ ] [outcome-focused criterion]
- [ ] [outcome-focused criterion]

### Regression
- [ ] [what must not break]

### Quality
- [ ] [observable standard]

## Out of Scope
- [exclusion 1]
- [exclusion 2]

## Context Files
Files relevant to this issue (re-read these after any compaction):
- `thoughts/shared/issues/TEAM-XXXX/ticket.md`

## References
- Ticket: thoughts/shared/issues/TEAM-XXXX/ticket.md
- Linear: https://linear.app/your-org/issue/TEAM-XXXX/[ticket-slug]
```

16. Sync the contract:
    ```bash
    thoughts sync -m "Add contract for TEAM-XXXX: [brief description]"
    ```

17. If a ticket exists:
    a. Attach the contract document to the ticket using `mcp__linear__save_issue` with proper link formatting (GitHub URL: `https://github.com/your-org/thoughts/blob/main/shared/issues/TEAM-XXXX/YYYY-MM-DD-contract-description.md`)
    b. Add a terse comment via `mcp__linear__create_comment` with a link to the contract
    c. Leave the ticket in "Contract" status (the next workflow step will advance it)

18. Print a summary for the user (replace placeholders with actual values):

```
Contract created for TEAM-XXXX: [ticket title]

Contract: thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-contract-description.md

Locked Objective:
- Problem: [one-liner]
- Target State: [one-liner]

Acceptance Criteria: [N functional, N regression, N quality]

Next steps:
- Review the contract document
- Run /research TEAM-XXXX to research the codebase (scoped by this contract)

View the ticket: https://linear.app/your-org/issue/TEAM-XXXX/[ticket-slug]
```

## GUIDING PRINCIPLES

- **Intent over implementation** — the contract captures WHAT, not HOW. Do not read source files or CLAUDE.md.
- **Neutral language in criteria** — write what should be true, not "verify there are no bugs."
- **Outcome over process** — criteria describe end states, not steps to get there.
- **Minimal scope** — the Out of Scope section prevents drift. The contract is a wall, not a wishlist.
- **User agreement required** — never proceed past Part II or Part III without explicit user confirmation.
- **Context anchoring** — always list relevant files in Context Files so they can be re-read after compaction.

## WORKFLOW INTEGRATION

This command is the first step in the pipeline:
1. **`/issue_contract`** — Understand intent, define "done" (this command)
2. `/research` — Research the codebase, scoped by the contract
3. `/plan` — Create an implementation plan that maps to contract criteria
4. `/implement_plan` — Execute the plan, verify contract criteria before commit
5. `/validate_plan` — Validate implementation against contract + plan criteria
6. `/code-review` — Multi-agent code review
