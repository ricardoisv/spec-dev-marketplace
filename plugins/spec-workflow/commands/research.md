---
description: Conductor-native research workflow for Linear tickets - fetches ticket, conducts research, documents findings
argument-hint: [ticket-number]
---

# Research Workflow

Conductor-native research workflow. Fetches a Linear ticket, conducts codebase and web research, documents findings, and updates ticket status. Works directly in the current Conductor workspace.

## PART I - DETERMINE WHAT TO RESEARCH

### If a ticket number is provided:

1. Use `linear` MCP (`mcp__linear__get_issue`) to fetch the ticket and save it to `./thoughts/shared/issues/TEAM-xxxx/ticket.md`
2. Read the ticket and use `mcp__linear__list_comments` to read all comments
3. Read any linked documents from the ticket's `links` section to understand existing context
4. **Check for an issue contract**:
   - Check ticket `links` for a contract document URL
   - Search `thoughts/shared/issues/TEAM-XXXX/` for files matching `*contract*`
   - If found, read the contract's **Locked Objective** and **Acceptance Criteria**
   - Use these to scope the research — sub-agents should focus on code relevant to the contract's functional criteria and regression concerns
5. If doubts, ask user for clarification on their intent and wait.

### If no ticket is provided:

5. If the user specified a research question or job, proceed to Part II following their instructions.
6. If the user provided neither a ticket nor a research question, ask for clarification and wait.

## PART II - CONDUCT RESEARCH

think deeply about what needs to be researched

7. Confirm research scope is clear, then move the ticket to "Research" using Linear MCP tools (skip if no ticket).

8. Conduct codebase research using parallel sub-agents:
   - **codebase-locator** — find WHERE relevant files and components live
   - **codebase-analyzer** — understand HOW specific code works
   - **codebase-pattern-finder** — find examples of existing patterns to document
   - **thoughts-locator** — discover existing documents about the topic
   - **thoughts-analyzer** — extract insights from relevant documents

   If an issue contract was found in Part I, scope each sub-agent's prompt: "Focus research on code relevant to these outcomes: [contract criteria summary]". This prevents broad, unfocused research and reduces context bloat.

9. If the ticket or user suggests web research is needed, also spawn:
   - **web-search-researcher** — research external solutions, APIs, or best practices

10. Wait for ALL sub-agents to complete before proceeding.

think deeply about the findings

11. Read all key files identified by the sub-agents directly into context. Cross-reference with ticket requirements.

## PART III - DOCUMENT FINDINGS

12. Gather metadata:
    ```bash
    git rev-parse HEAD
    git branch --show-current
    basename $(git rev-parse --show-toplevel)
    date -Iseconds
    ```

13. Write research document to `thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-research-description.md`:
    - YYYY-MM-DD is today's date
    - TEAM-XXXX is the ticket number (omit if no ticket)
    - description is a brief kebab-case description
    - Use full YAML frontmatter (date, researcher, git_commit, branch, repository, topic, tags, status)
    - Be unbiased — document how systems work today, don't propose an implementation plan
    - Include specific file paths and line numbers for all references
    - Document cross-component connections and data flows

14. Sync the research document:
    ```bash
    thoughts sync -m "Add research for TEAM-XXXX: [brief description]"
    ```

15. Write a brief summary of findings to `.context/notes.md`:
    ```bash
    mkdir -p .context
    ```
    - Include: ticket ID, research document path, 3-5 key findings
    - This makes findings visible in the Conductor UI for other agents

## PART IV - UPDATE TICKET AND REPORT

16. If a ticket exists:
    a. Attach the research document to the ticket using `mcp__linear__save_issue` with proper link formatting (GitHub URL: `https://github.com/your-org/thoughts/blob/main/shared/issues/TEAM-XXXX/YYYY-MM-DD-research-description.md`)
    b. Add a comment via `mcp__linear__create_comment` summarizing the research outcomes
    c. Leave the ticket in "Research" status (the next workflow step will advance it)

17. If a contract was found, note in the summary that research was scoped by the contract.

18. Print a summary for the user (replace placeholders with actual values, make sure major findings are related to the issue at hand, mention any open questions if any):

```
Research complete for TEAM-XXXX: [ticket title]

Research document: thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-research-description.md

Key findings:
- [Major finding 1]
- [Major finding 2]
- [Major finding 3]

Open questions [if any]:
- [Open question 1]
- [Open question 2]

Next steps:
- Review the research document
- Run /plan TEAM-XXXX to create an implementation plan

View the ticket: https://linear.app/your-org/issue/TEAM-XXXX/[ticket-slug]
```

## HANDLING FOLLOW-UPS

If the user asks follow-up questions after research is complete:
- Append to the same research document (do not create a new one)
- Update frontmatter: `last_updated`, `last_updated_by`, add `last_updated_note`
- Add a new section: `## Follow-up Research [timestamp]`
- Spawn new sub-agents as needed
- Sync again after updates
