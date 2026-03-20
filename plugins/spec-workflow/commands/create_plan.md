---
description: Create detailed implementation plans through interactive, iterative process with TDD
argument-hint: [ticket-file or TEAM-XXX]
---

# Implementation Plan

Create detailed implementation plans through an interactive, iterative process. Be skeptical, thorough, and work collaboratively with the user to produce high-quality technical specifications. Follow test-driven-development practices.

## Initial Response

When this command is invoked:

1. **Check if parameters were provided**:
   - If a file path or ticket reference was provided, skip the default message
   - Immediately read any provided files FULLY
   - Begin the research process

2. **If no parameters provided**, respond with:
```
I'll help you create a detailed implementation plan. Let me start by understanding what we're building.

Please provide:
1. The task/ticket description (or reference to a ticket file)
2. Any relevant context, constraints, or specific requirements
3. Links to related research or previous implementations

I'll analyze this information and work with you to create a comprehensive plan.

Tip: You can also invoke this command with a ticket file directly: `/create_plan thoughts/shared/issues/TEAM-XXX/ticket.md`
```

Then wait for the user's input.

## Process Steps

### Step 1: Context Gathering & Initial Analysis

1. **Read all mentioned files immediately and FULLY**:
   - Ticket files (e.g., `thoughts/shared/issues/TEAM-XXX/ticket.md`)
   - Research documents
   - Related implementation plans
   - **IMPORTANT**: Use the Read tool WITHOUT limit/offset parameters
   - **CRITICAL**: DO NOT spawn sub-tasks before reading these files yourself

2. **Spawn initial research tasks to gather context**:
   - Use **codebase-locator** agent to find all files related to the task
   - Use **codebase-analyzer** agent to understand current implementation
   - Use **thoughts-locator** agent to find existing thoughts documents

3. **Read all files identified by research tasks**:
   - After research tasks complete, read ALL files they identified as relevant
   - Read them FULLY into the main context

4. **Analyze and verify understanding**:
   - Cross-reference requirements with actual code
   - Identify discrepancies or misunderstandings
   - Note assumptions that need verification

5. **Present informed understanding and focused questions**:
   ```
   Based on the ticket and my research of the codebase, I understand we need to [accurate summary].

   I've found that:
   - [Current implementation detail with file:line reference]
   - [Relevant pattern or constraint discovered]
   - [Potential complexity or edge case identified]

   Questions that my research couldn't answer:
   - [Specific technical question that requires human judgment]
   - [Business logic clarification]
   ```

### Step 2: Research & Discovery

After getting initial clarifications:

1. **If the user corrects any misunderstanding**:
   - Spawn new research tasks to verify the correct information
   - Only proceed once you've verified the facts yourself

2. **Create a research todo list** using TodoWrite

3. **Spawn parallel sub-tasks for comprehensive research**:
   - **codebase-locator** - Find specific files
   - **codebase-analyzer** - Understand implementation details
   - **codebase-pattern-finder** - Find similar features to model after
   - **thoughts-locator** - Find existing research/plans about this area

4. **Wait for ALL sub-tasks to complete** before proceeding

5. **Present findings and design options**:
   ```
   Based on my research, here's what I found:

   **Current State:**
   - [Key discovery about existing code]
   - [Pattern or convention to follow]

   **Design Options:**
   1. [Option A] - [pros/cons]
   2. [Option B] - [pros/cons]

   Which approach aligns best with your vision?
   ```

### Step 3: Plan Structure Development

Once aligned on approach:

1. **Create initial plan outline**:
   ```
   Here's my proposed plan structure:

   ## Overview
   [1-2 sentence summary]

   ## Implementation Phases:
   1. [Phase name] - [what it accomplishes]
   2. [Phase name] - [what it accomplishes]
   3. [Phase name] - [what it accomplishes]

   Does this phasing make sense?
   ```

2. **Get feedback on structure** before writing details

### Step 4: Detailed Plan Writing

After structure approval:

1. **Write the plan** to `thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-plan-description.md`
   - Format: `YYYY-MM-DD-plan-description.md` where:
     - YYYY-MM-DD is today's date
     - TEAM-XXXX is the ticket number (omit if no ticket)
     - description is a brief kebab-case description

2. **Use this template structure**:

```markdown
# [Feature/Task Name] Implementation Plan

## Overview
[Brief description of what we're implementing and why]

## Current State Analysis
[What exists now, what's missing, key constraints discovered]

## Desired End State
[Specification of the desired end state and how to verify it]

### Key Discoveries:
- [Important finding with file:line reference]
- [Pattern to follow]
- [Constraint to work within]

## What We're NOT Doing
[Explicitly list out-of-scope items to prevent scope creep]

## Implementation Approach
[High-level strategy and reasoning]

## Testing Strategy
[TDD - failing and passing tests are written first]

### Unit Tests:
- [What to test]
- [Key edge cases]

### Integration Tests:
- [End-to-end scenarios]

### Manual Testing Steps:
1. [Specific step to verify feature]
2. [Another verification step]

## Phase 1: [Descriptive Name]

### Overview
[What this phase accomplishes]

### Changes Required:

#### 1. [Component/File Group]
**File**: `path/to/file.ext`
**Changes**: [Summary of changes]

### Success Criteria:

#### Automated Verification:
- [ ] Tests pass: `bun run lint` / `make test`
- [ ] Type checking passes
- [ ] Linting passes

#### Manual Verification:
- [ ] Feature works as expected when tested via UI
- [ ] No regressions in related features

**Implementation Note**: After completing this phase and all automated verification passes, pause for manual confirmation before proceeding.

---

## Phase 2: [Descriptive Name]
[Similar structure...]

---

## Performance Considerations
[Any performance implications]

## Migration Notes
[If applicable]

## References
- Original ticket: `thoughts/shared/issues/TEAM-XXXX/ticket.md`
- Related research: `thoughts/shared/issues/TEAM-XXXX/[relevant-research].md`
```

### Step 5: Sync and Review

1. **Sync the thoughts directory**:
   ```bash
   thoughts sync -m "Add implementation plan for TEAM-XXXX: [brief description]"
   ```

2. **Present the draft plan location**:
   ```
   I've created the initial implementation plan at:
   `thoughts/shared/issues/TEAM-XXXX/YYYY-MM-DD-plan-description.md`

   Please review it and let me know:
   - Are the phases properly scoped?
   - Are the success criteria specific enough?
   - Any technical details that need adjustment?
   ```

3. **Iterate based on feedback** - after making changes, run `thoughts sync` again

## Important Guidelines

1. **Be Skeptical**: Question vague requirements, identify issues early, verify with code
2. **Be Interactive**: Don't write the full plan in one shot, get buy-in at each step
3. **Be Thorough**: Read all context files COMPLETELY, include specific file paths
4. **Be Practical**: Focus on incremental, testable changes
5. **No Open Questions in Final Plan**: Research or ask for clarification immediately

## Success Criteria Guidelines

**Always separate into two categories:**

1. **Automated Verification** (can be run by agents):
   - Commands: `bun run lint`, `bun test`, etc.
   - Code compilation/type checking
   - Automated test suites

2. **Manual Verification** (requires human testing):
   - UI/UX functionality
   - Performance under real conditions
   - Edge cases hard to automate
