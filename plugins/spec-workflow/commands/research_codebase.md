---
description: Conduct comprehensive codebase research with parallel sub-agents
argument-hint: [research question or TEAM-XXX]
---

# Research Codebase

Conduct comprehensive research across the codebase to answer questions by spawning parallel sub-agents and synthesizing findings.

## CRITICAL: YOUR ONLY JOB IS TO DOCUMENT AND EXPLAIN THE CODEBASE AS IT EXISTS TODAY

- DO NOT suggest improvements or changes unless explicitly asked
- DO NOT perform root cause analysis unless explicitly asked
- DO NOT propose future enhancements unless explicitly asked
- DO NOT critique the implementation or identify problems
- ONLY describe what exists, where it exists, how it works, and how components interact
- You are creating a technical map/documentation of the existing system

## Initial Setup

When this command is invoked, respond with:
```
I'm ready to research the codebase. Please provide your research question or area of interest, and I'll analyze it thoroughly by exploring relevant components and connections.
```

Then wait for the user's research query.

## Steps

### 1. Read Any Directly Mentioned Files First

- If the user mentions specific files, read them FULLY first
- Use the Read tool WITHOUT limit/offset parameters
- Read these files yourself in the main context before spawning sub-tasks

### 2. Analyze and Decompose the Research Question

- Break down the query into composable research areas
- Think deeply about patterns, connections, and architectural implications
- Identify specific components, patterns, or concepts to investigate
- Create a research plan using TodoWrite

### 3. Spawn Parallel Sub-Agent Tasks

Create multiple Task agents to research different aspects concurrently:

**For codebase research:**
- **codebase-locator** agent - Find WHERE files and components live
- **codebase-analyzer** agent - Understand HOW specific code works (without critiquing)
- **codebase-pattern-finder** agent - Find examples of existing patterns (without evaluating)

**For thoughts directory:**
- **thoughts-locator** agent - Discover what documents exist about the topic
- **thoughts-analyzer** agent - Extract key insights from relevant documents

**For web research (only if user explicitly asks):**
- **web-search-researcher** agent - External documentation and resources
- Include LINKS with findings in final report

**For Linear tickets (if relevant):**
- Use Linear MCP tools to get ticket details or find related tickets

Guidelines:
- Start with locator agents to find what exists
- Then use analyzer agents on promising findings
- Run multiple agents in parallel when searching for different things
- Remind agents they are documenting, not evaluating

### 4. Wait for All Sub-Agents and Synthesize

- IMPORTANT: Wait for ALL sub-agent tasks to complete
- Compile all results (codebase and thoughts findings)
- Prioritize live codebase findings as primary source of truth
- Use thoughts/ findings as supplementary historical context
- Include specific file paths and line numbers
- Highlight patterns, connections, and architectural decisions

### 5. Gather Metadata

```bash
git rev-parse HEAD          # commit hash
git branch --show-current   # branch name
basename $(git rev-parse --show-toplevel)  # repo name
date -Iseconds              # current timestamp
```

Filename:
- With ticket: `thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-research-description.md`
- Without ticket: `thoughts/shared/issues/{slug}/YYYY-MM-DD-research-description.md`

Where:
- YYYY-MM-DD is today's date
- TEAM-XXXX is the ticket number
- {slug} is a kebab-case topic name for standalone research
- description is a brief kebab-case description

Examples:
- With ticket: `thoughts/shared/issues/TEAM-320/2026-01-26-research-clickhouse-integration.md`
- Without ticket: `thoughts/shared/issues/authentication-flow/2026-01-26-research-authentication-flow.md`

### 6. Generate Research Document

```markdown
---
date: [ISO timestamp with timezone]
researcher: [Researcher name or "Claude"]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "[User's Question/Topic]"
tags: [research, codebase, relevant-component-names]
status: complete
last_updated: [YYYY-MM-DD]
last_updated_by: [Researcher name]
---

# Research: [User's Question/Topic]

**Date**: [Current date and time]
**Researcher**: [Name]
**Git Commit**: [hash]
**Branch**: [branch]
**Repository**: [repo]

## Research Question
[Original user query]

## Summary
[High-level documentation of what was found]

## Detailed Findings

### [Component/Area 1]
- Description of what exists ([file.ext:line](link))
- How it connects to other components
- Current implementation details (without evaluation)

### [Component/Area 2]
...

## Code References
- `path/to/file.py:123` - Description of what's there
- `another/file.ts:45-67` - Description of the code block

## Architecture Documentation
[Current patterns, conventions, and design implementations]

## Historical Context (from thoughts/)
[Relevant insights from thoughts/ directory with references]

## Related Research
[Links to other research documents in thoughts/shared/issues/TEAM-XXXX/ or related issue dirs]

## Open Questions
[Any areas that need further investigation]
```

### 7. Sync and Present Findings

```bash
thoughts sync -m "Add research for TEAM-XXXX: [brief description]"
```

Present a concise summary to the user with key file references.

### 8. Handle Follow-up Questions

- Append to the same research document
- Update frontmatter: `last_updated`, `last_updated_by`
- Add: `last_updated_note: "Added follow-up research for [description]"`
- Add new section: `## Follow-up Research [timestamp]`
- Spawn new sub-agents as needed
- Sync again after updates

## Important Notes

- Always use parallel Task agents to maximize efficiency
- Always run fresh codebase research - never rely solely on existing documents
- Research documents should be self-contained with all necessary context
- Document cross-component connections
- **CRITICAL**: You are a documentarian, not an evaluator
- **REMEMBER**: Document what IS, not what SHOULD BE
- **NO RECOMMENDATIONS**: Only describe the current state
