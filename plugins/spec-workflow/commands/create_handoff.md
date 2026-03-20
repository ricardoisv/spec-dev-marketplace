---
description: Create a handoff document to pass work to another agent session
argument-hint: [TEAM-XXX or description]
---

# Create Handoff

Create a handoff document to pass work to another agent in a new session. The document should be thorough but concise - compact and summarize context without losing key details.

## Process

### 1. Filepath & Metadata

Create your file under `thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD_HH-MM-SS-handoff-description.md`, where:
- YYYY-MM-DD is today's date
- HH-MM-SS is the current time in 24-hour format
- TEAM-XXXX is the ticket number (replace with `general` if no ticket)
- description is a brief kebab-case description

Examples:
- With ticket: `thoughts/shared/issues/TEAM-316/2026-01-26_14-30-00-handoff-api-endpoint-implementation.md`
- Without ticket: `thoughts/shared/issues/general/2026-01-26_14-30-00-handoff-refactor-auth-flow.md`

Gather metadata:
```bash
# Get current info
git rev-parse HEAD          # commit hash
git branch --show-current   # branch name
basename $(git rev-parse --show-toplevel)  # repo name
date -Iseconds              # current timestamp
```

### 2. Handoff Writing

Use the following template structure:

```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Your name or "Claude"]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "[Feature/Task Name] Implementation"
tags: [handoff, implementation, relevant-component-names]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Your name or "Claude"]
type: handoff
---

# Handoff: TEAM-XXXX [very concise description]

## Task(s)
[Description of tasks you were working on with status: completed, work in progress, planned/discussed. Reference the plan document and/or research documents you are working from.]

## Critical References
[List 2-3 most important specification documents, architectural decisions, or design docs that must be followed. Leave blank if none.]

## Recent Changes
[Describe recent changes made to the codebase in file:line syntax]

## Learnings
[Important patterns, root causes of bugs, or other information the next agent should know. Include explicit file paths.]

## Artifacts
[Exhaustive list of artifacts you produced or updated as filepaths and/or file:line references - plans, research docs, etc.]

## Action Items & Next Steps
[List of action items and next steps for the next agent based on task statuses]

## Other Notes
[Other references or useful information - relevant codebase sections, documents, or learnings that don't fit above categories]
```

### 3. Approve and Sync

1. Ask the user to review and approve the document
2. If they request changes, make them and ask for approval again
3. Once approved, sync the document:
   ```bash
   thoughts sync -m "Add handoff for TEAM-XXXX: [brief description]"
   ```

4. Respond to the user with:
   ```
   Handoff created and synced! You can resume from this handoff in a new session with:

   /resume_handoff thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD_HH-MM-SS-handoff-description.md
   ```

   Or if using ticket number:
   ```
   /resume_handoff TEAM-XXXX
   ```

## Additional Notes & Instructions

- **More information, not less** - This template defines the minimum; include more if necessary
- **Be thorough and precise** - Include both top-level objectives and lower-level details
- **Avoid excessive code snippets** - Prefer file:line references over large code blocks
- **Reference implementation phase** - If working on a plan, call out which phase you're on
