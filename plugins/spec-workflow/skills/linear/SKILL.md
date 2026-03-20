---
description: Skill for managing Linear tickets - creating, reading, updating issues, and adding comments. Always follow these guidelines when interacting with Linear.
globs:
  - "**/*"
alwaysApply: false
---

# Linear Ticket Management Skill

You are skilled at managing Linear tickets for the team, including creating tickets, updating existing tickets, adding comments, and following the team's workflow patterns.

## When to Apply

This skill activates when the user asks to:
- Create a Linear ticket/issue
- Read or fetch a ticket
- Add a comment to a ticket
- Update ticket status or details
- Search for tickets
- Attach documents or links to tickets

## Linear Workspace Info

<!-- TODO: Replace these with your team's actual Linear workspace details -->
- **Workspace**: your-org
- **Team**: Product (ID: `<your-team-uuid>`)
- **URL Format**: `https://linear.app/your-org/issue/TEAM-XXXX/[ticket-slug]`

## Team Members

<!-- TODO: Replace with your actual team members -->
| Name | Display Name | Email | ID |
|------|--------------|-------|-----|
| Alice Smith | alice | alice@your-company.com | `<your-user-uuid-1>` |
| Bob Jones | bob | bob@your-company.com | `<your-user-uuid-2>` |
| Carol Lee | carol | carol@your-company.com | `<your-user-uuid-3>` |

## Available Statuses

<!-- TODO: Replace UUIDs with your actual Linear status IDs -->
| Status | Type | ID | Workflow Step |
|--------|------|-----|--------------|
| Triage | triage | `<your-status-uuid-1>` | Initial review |
| Backlog | backlog | `<your-status-uuid-2>` | Waiting to be prioritized |
| Todo | unstarted | `<your-status-uuid-3>` | Ready to work on |
| Contract | started | `<your-status-uuid-4>` | `/issue_contract` |
| Research | started | `<your-status-uuid-5>` | `/research` |
| Plan | started | `<your-status-uuid-6>` | `/plan` |
| In Progress | started | `<your-status-uuid-7>` | General active work |
| Implemented | started | `<your-status-uuid-8>` | `/implement_plan` |
| Validated | started | `<your-status-uuid-9>` | `/validate_plan` |
| In Review | started | `<your-status-uuid-10>` | `/code-review` |
| Done | completed | `<your-status-uuid-11>` | Completed |
| Canceled | canceled | `<your-status-uuid-12>` | — |
| Duplicate | canceled | `<your-status-uuid-13>` | — |

### Changing Status

To change a ticket's status, use `mcp__linear__update_issue`:
```
mcp__linear__update_issue
- id: "TEAM-XXX" or ticket UUID
- state: "In Progress" (use status name, not ID)
```

### Workflow Status Progression

Each workflow command moves the ticket to its corresponding status:

```
Triage → Backlog → Todo → Contract → Research → Plan → Implemented → Validated → In Review → Done
```

| Command | Sets status to |
|---------|---------------|
| `/issue_contract` | Contract |
| `/research` | Research |
| `/plan` | Plan |
| `/implement_plan` | Implemented |
| `/validate_plan` | Validated |
| `/code-review` | In Review |

## Available Labels

<!-- TODO: Replace UUIDs with your actual Linear label IDs -->

### Type Labels
| Label | ID |
|-------|-----|
| Bug | `<your-label-uuid-1>` |
| Feature | `<your-label-uuid-2>` |
| Improvement | `<your-label-uuid-3>` |
| docs | `<your-label-uuid-4>` |
| tech debt | `<your-label-uuid-5>` |

### Area Labels
| Label | ID |
|-------|-----|
| frontend | `<your-label-uuid-6>` |
| backend | `<your-label-uuid-7>` |
| infrastructure | `<your-label-uuid-8>` |
| SDK | `<your-label-uuid-9>` |
| API | `<your-label-uuid-10>` |
| database | `<your-label-uuid-11>` |

### Priority Labels
| Label | ID |
|-------|-----|
| p0 | `<your-label-uuid-12>` |
| p1 | `<your-label-uuid-13>` |
| p2 | `<your-label-uuid-14>` |
| p3 | `<your-label-uuid-15>` |

## Linear MCP Tools Reference

### Reading Issues
```
mcp__linear__get_issue
- id: ticket ID or identifier (e.g., "TEAM-123")
- includeRelations: true to include blocking/related issues
```

### Listing Issues
```
mcp__linear__list_issues
- team: "Product"
- state: status name (e.g., "Todo", "In Progress")
- assignee: user name or "me"
- query: search text
- limit: number of results (default 50, max 250)
```

### Creating Issues
```
mcp__linear__create_issue
- title: clear, action-oriented title
- team: "Product"
- description: markdown description
- state: status name (default to "Triage" for new tickets)
- priority: 0=None, 1=Urgent, 2=High, 3=Normal, 4=Low
- assignee: user name or "me"
- labels: array of label names
- links: array of {url, title} objects for attachments
```

### Updating Issues
```
mcp__linear__update_issue
- id: ticket ID
- state: new status name
- description: updated description
- priority: new priority
- links: array of {url, title} objects
```

### Comments
```
mcp__linear__list_comments
- issueId: ticket ID

mcp__linear__create_comment
- issueId: ticket ID
- body: markdown comment content
```

## Best Practices

### Creating Tickets

1. **Always include a clear problem statement:**
   - If user only gives implementation details, ask: "To write a good ticket, please explain the problem you're trying to solve from a user perspective"

2. **Ticket structure:**
   ```markdown
   ## Problem
   [Clear description of the problem to solve]

   ## Details
   - [Key implementation notes]
   - [Technical constraints]

   ## References
   - [Links to relevant documents or code]
   ```

3. **Default values:**
   - Status: "Triage" for new tickets
   - Priority: 3 (Normal) unless specified
   - Team: "Product"

### Adding Comments

1. **Focus on valuable information:**
   - Key insights and discoveries
   - Decisions and tradeoffs made
   - Blockers resolved
   - What's different now

2. **Keep comments concise (~10 lines):**
   - Focus on what matters for a human reader
   - Include relevant file references with backticks

3. **Comment structure example:**
   ```markdown
   Implemented retry logic to address rate limit issues.

   Key insight: The 429 responses were clustered during batch operations,
   so exponential backoff alone wasn't sufficient - added request queuing.

   Files updated:
   - `src/handlers/webhook.py`
   - `thoughts/shared/issues/TEAM-XXX/2026-01-08-research-rate-limits.md`
   ```

### Attaching Documents

When attaching thoughts documents or research:

1. **Use the `links` parameter** (not just markdown links in description)
2. **Provide meaningful titles** for links
3. **GitHub URL format for thoughts:**
   <!-- TODO: Replace with your actual GitHub org and thoughts repo -->
   - `thoughts/shared/...` → `https://github.com/your-org/thoughts/blob/main/shared/...`
   - Example: `thoughts/shared/issues/TEAM-123/2026-01-15-research-foo.md` → `https://github.com/your-org/thoughts/blob/main/shared/issues/TEAM-123/2026-01-15-research-foo.md`

### Updating Status

Follow the workflow progression:
1. **Triage** → Initial review
2. **Backlog** → Waiting to be prioritized
3. **Todo** → Ready to work on
4. **Contract** → Defining acceptance criteria (`/issue_contract`)
5. **Research** → Codebase/web research (`/research`)
6. **Plan** → Implementation planning (`/plan`)
7. **In Progress** → General active work (non-pipeline)
8. **Implemented** → Code complete (`/implement_plan`)
9. **Validated** → Plan/contract verified (`/validate_plan`)
10. **In Review** → Code review (`/code-review`)
11. **Done** → Completed

When changing status, consider adding a brief comment explaining why.

## Common Operations

### Fetch a ticket and show details
```
1. mcp__linear__get_issue with id: "TEAM-XXX"
2. Display: title, status, description, assignee, links
3. mcp__linear__list_comments to show discussion
```

### Create ticket from research
```
1. Read the research document
2. Extract problem statement and key findings
3. Draft ticket with proper structure
4. Present draft to user for confirmation
5. mcp__linear__create_issue with links to research doc
```

### Add research findings to existing ticket
```
1. mcp__linear__get_issue to fetch current state
2. mcp__linear__update_issue to add link to research doc
3. mcp__linear__create_comment summarizing findings
4. Optionally update status to reflect progress
```

## Important Notes

- Always verify Linear MCP tools are available before proceeding
- Use proper markdown formatting in descriptions and comments
- Include code references as: `path/to/file.ext:linenum`
- Ask for clarification rather than guessing project/status
- Keep tickets scannable - concise but complete
- Focus on "what" and "why", include "how" only if well-defined
