---
description: Resume work from a handoff document
argument-hint: [TEAM-XXX or path/to/handoff.md]
---

# Resume Handoff

Resume work from a handoff document through an interactive process. These handoffs contain critical context, learnings, and next steps from previous work sessions.

## Initial Response

When this command is invoked:

1. **If a handoff document path was provided**:
   - Immediately read the handoff document FULLY
   - Read any research or plan documents it links to under `thoughts/shared/issues/TEAM-XXXX/`
   - DO NOT use a sub-agent to read these critical files
   - Begin the analysis process
   - Propose a course of action and confirm with user

2. **If a ticket number (like TEAM-XXX) was provided**:
   - Locate the most recent handoff document for the ticket at `thoughts/shared/issues/TEAM-XXX/`
   - List the directory contents
   - If zero files or directory doesn't exist: ask the user for the path
   - If one file: proceed with that handoff
   - If multiple files: use the most recent one (based on YYYY-MM-DD_HH-MM-SS in filename)
   - Read the handoff document FULLY
   - Read any linked research or plan documents
   - Propose a course of action and confirm

3. **If no parameters provided**, respond with:
   ```
   I'll help you resume work from a handoff document.

   Which handoff would you like to resume from?

   Tip: You can invoke this command with:
   - A handoff path: `/resume_handoff thoughts/shared/issues/TEAM-XXX/YYYY-MM-DD_HH-MM-SS-handoff-description.md`
   - A ticket number: `/resume_handoff TEAM-XXX` (uses most recent handoff for that ticket)
   ```

## Process Steps

### Step 1: Read and Analyze Handoff

1. **Read handoff document completely** (no limit/offset)

2. **Extract all sections**:
   - Task(s) and their statuses
   - Recent changes
   - Learnings
   - Artifacts
   - Action items and next steps
   - Other notes

3. **Spawn focused research tasks** to verify current state:

   ```
   Task 1 - Verify recent changes:
   Check if the recent changes mentioned in the handoff still exist.
   Return: Current state of recent changes with file:line references
   ```

   ```
   Task 2 - Validate current codebase state:
   Verify the current state against what's described in the handoff.
   Return: Validation results and any discrepancies found
   ```

   ```
   Task 3 - Gather artifact context:
   Read all artifacts mentioned in the handoff.
   Return: Summary of artifact contents and key decisions
   ```

4. **Wait for ALL sub-tasks to complete**

5. **Read critical files identified** from the "Learnings" and "Recent changes" sections

### Step 2: Synthesize and Present Analysis

Present comprehensive analysis:

```
I've analyzed the handoff from [date]. Here's the current situation:

**Original Tasks:**
- [Task 1]: [Status from handoff] → [Current verification]
- [Task 2]: [Status from handoff] → [Current verification]

**Key Learnings Validated:**
- [Learning with file:line reference] - [Still valid/Changed]

**Recent Changes Status:**
- [Change 1] - [Verified present/Missing/Modified]

**Artifacts Reviewed:**
- [Document 1]: [Key takeaway]

**Recommended Next Actions:**
Based on the handoff's action items and current state:
1. [Most logical next step]
2. [Second priority action]

**Potential Issues Identified:**
- [Any conflicts or regressions found]

Shall I proceed with [recommended action 1], or would you like to adjust the approach?
```

### Step 3: Create Action Plan

1. **Use TodoWrite to create task list**:
   - Convert action items from handoff into todos
   - Add any new tasks discovered during analysis
   - Prioritize based on dependencies

2. **Present the plan** and get confirmation

### Step 4: Begin Implementation

1. Start with the first approved task
2. Reference learnings from handoff throughout
3. Apply documented patterns and approaches
4. Update progress as tasks are completed

## Guidelines

1. **Be Thorough in Analysis**:
   - Read the entire handoff document first
   - Verify ALL mentioned changes still exist
   - Check for regressions or conflicts
   - Read all referenced artifacts

2. **Be Interactive**:
   - Present findings before starting work
   - Get buy-in on the approach
   - Allow for course corrections

3. **Leverage Handoff Wisdom**:
   - Pay special attention to "Learnings" section
   - Apply documented patterns
   - Avoid repeating mentioned mistakes

4. **Track Continuity**:
   - Use TodoWrite to maintain task continuity
   - Reference the handoff document in commits
   - Consider creating a new handoff when done

5. **Validate Before Acting**:
   - Never assume handoff state matches current state
   - Verify all file references still exist
   - Check for breaking changes since handoff

## Common Scenarios

### Scenario 1: Clean Continuation
- All changes present, no conflicts
- Proceed with recommended actions

### Scenario 2: Diverged Codebase
- Some changes missing or modified
- Reconcile differences, adapt plan

### Scenario 3: Incomplete Handoff Work
- Tasks marked "in_progress"
- Complete unfinished work first

### Scenario 4: Stale Handoff
- Significant time has passed
- Re-evaluate strategy based on current state
