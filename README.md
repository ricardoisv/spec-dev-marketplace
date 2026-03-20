# Spec-Driven Development Marketplace for Claude Code

A Claude Code plugin marketplace that provides a complete spec-driven development workflow. These commands guide Claude through a structured pipeline: from understanding intent, through research and planning, to implementation, validation, and code review.

## What's Included

### Core Pipeline Commands

Run these in order for a complete development workflow:

| Command | Description |
|---------|-------------|
| `/issue_contract` | Understand user intent for a ticket and create a verifiable completion contract |
| `/research` | Conduct codebase research scoped by the contract |
| `/plan` | Create a phased implementation plan mapped to contract criteria |
| `/implement_plan` | Execute the plan, verify contract criteria before committing |
| `/validate_plan` | Validate the implementation against plan and contract |
| `/code-review` | Multi-agent code review with confidence scoring |

### Supporting Commands

| Command | Description |
|---------|-------------|
| `/create_plan` | Interactive plan creation with TDD (standalone, outside the pipeline) |
| `/research_codebase` | Standalone codebase research and documentation |
| `/create_handoff` | Create a handoff document for session continuity |
| `/resume_handoff` | Resume work from a previous handoff |
| `/local_review` | Set up workspace to review a colleague's branch |

### Skills (Always Available)

| Skill | Description |
|-------|-------------|
| **commit** | Clean git commits with focused messages (always active) |
| **linear** | Linear ticket management with workflow status tracking |
| **thoughts-init** | Auto-initialize the thoughts directory in new workspaces |

## Installation

### Option 1: Add to your project's settings

Add the marketplace to `.claude/settings.json` in your repository:

```json
{
  "extraKnownMarketplaces": {
    "spec-dev-marketplace": {
      "source": {
        "source": "github",
        "repo": "your-org/your-marketplace-repo"
      }
    }
  },
  "enabledPlugins": {
    "spec-workflow@spec-dev-marketplace": true
  }
}
```

### Option 2: Manual install via Claude Code

```bash
/plugin marketplace add your-org/your-marketplace-repo
/plugin install spec-workflow@spec-dev-marketplace
```

## Customization

### Linear Integration

The Linear skill (`plugins/spec-workflow/skills/linear/SKILL.md`) needs to be configured with your team's actual Linear workspace details. Look for `<!-- TODO: ... -->` comments and replace:

1. **Workspace name and team UUID** - Your Linear workspace identifier and team ID
2. **Team member names, emails, and UUIDs** - Your actual team members
3. **Status UUIDs** - Your Linear workflow status IDs (the status names can stay as-is if you set up matching statuses)
4. **Label UUIDs** - Your Linear label IDs
5. **GitHub org** in URL templates - Your actual GitHub organization name

### Ticket Prefix

The commands use `TEAM-XXXX` as the ticket prefix. To change this:

1. Find and replace `TEAM-` with your project prefix (e.g., `ENG-`, `PROJ-`) across all command files
2. Update the regex patterns in `validate_plan.md` and `code-review.md` that match `TEAM-\d+`

### Thoughts Directory

The workflow uses a `thoughts/` directory for storing research, plans, contracts, and handoffs. This is managed by a separate CLI tool. If you don't use this pattern, you can:

1. Replace `thoughts/shared/issues/` paths with your preferred documentation location
2. Remove or modify the `thoughts sync` commands
3. Remove the `thoughts-init` skill if not needed

## Updating

After merging changes to this repo, team members refresh their local cache:

```bash
/plugin marketplace update spec-dev-marketplace
```

This happens automatically when the version number changes (bumped on every PR merge via GitHub Actions).

## How It Works

The pipeline is designed around **contracts** — verifiable definitions of "done" that scope all downstream work:

1. **Contract** defines acceptance criteria (what success looks like)
2. **Research** explores the codebase, scoped by the contract
3. **Plan** creates phased implementation steps mapped to contract criteria
4. **Implementation** executes the plan and verifies contract criteria
5. **Validation** independently verifies everything against plan + contract
6. **Code Review** uses multiple agents with confidence scoring to catch real issues

Each step produces artifacts in the `thoughts/` directory and updates the Linear ticket, creating a complete audit trail.

## Versioning

This repo uses automated version bumping via GitHub Actions. A workflow bumps the minor version on every PR merge to `main`. Do not manually edit version numbers in `marketplace.json` or `plugin.json`.
