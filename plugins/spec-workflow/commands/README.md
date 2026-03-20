# Spec Workflow Commands

This directory contains all the spec workflow commands (slash commands).

## Command Structure

Each command should be a separate `.md` file with the following frontmatter:

```markdown
---
description: Brief description of what the command does
argument-hint: [arg1] [arg2]
---

# Command Name

Detailed instructions for Claude on how to execute this command.

Include:
- What the command does
- Required parameters
- Expected behavior
- Example usage
```

## Available Commands

### Completed

**Core Pipeline (in order):**
- [x] `/issue_contract` - Understand user intent, create verifiable completion contract
- [x] `/research` - Ticket research workflow (scoped by contract)
- [x] `/plan` - Implementation planning workflow (maps to contract criteria)
- [x] `/implement_plan` - Implementation workflow (verifies contract before commit)
- [x] `/validate_plan` - Plan and contract validation
- [x] `/code-review` - Multi-agent code review

**Supporting Commands:**
- [x] `/create_plan` - Interactive plan creation with TDD
- [x] `/local_review` - Set up workspace for reviewing a colleague's branch
- [x] `/create_handoff` - Create handoff document for session continuity
- [x] `/resume_handoff` - Resume from a handoff document
- [x] `/research_codebase` - Standalone codebase research

### To Be Added

- [ ] `/describe_pr` - PR description generation
- [ ] `/debug` - Debugging workflow
- [ ] And others...

**Note:** Commands should be adapted to work with the spec workflow before being added here.
