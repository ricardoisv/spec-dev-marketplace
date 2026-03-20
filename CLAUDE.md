# Spec-Driven Development Marketplace

## Versioning

This repo uses **automated version bumping**. A GitHub Actions workflow (`.github/workflows/version-bump.yml`) bumps the minor version on every PR merge to `main`.

- **Do NOT manually edit version numbers** in `.claude-plugin/marketplace.json` or `plugins/spec-workflow/.claude-plugin/plugin.json` — the workflow handles this automatically.
- The version bump commit uses `[skip ci]` and is pushed directly to `main` by `github-actions[bot]`.
- If you see a version conflict, pull latest `main` — the workflow likely bumped it after your branch was created.
- Both `marketplace.json` and `plugin.json` versions are kept in sync automatically.

## Installation

Requires **Claude Code v1.0.33+**.

### For team repositories

Add the marketplace to your project's `.claude/settings.json` so team members get prompted to install automatically:

```json
{
  "extraKnownMarketplaces": {
    "klira-spec-dev-marketplace": {
      "source": {
        "source": "github",
        "repo": "KliraAI/klira-cc-public-marketplace"
      }
    }
  },
  "enabledPlugins": {
    "klira@klira-spec-dev-marketplace": true
  }
}
```

When a team member opens the project and trusts the folder, Claude Code will prompt them to install the marketplace and plugin. Note: this only works in interactive mode, not headless (`claude -p`).

### Manual install

```bash
/plugin marketplace add KliraAI/klira-cc-public-marketplace
/plugin install klira@klira-spec-dev-marketplace
```

After installation, run `/reload-plugins` to activate without restarting Claude Code.

## Updating the Cache

After merging changes to this repo, team members need to refresh their local cache to pick up the new version:

```bash
/plugin marketplace update klira-spec-dev-marketplace
```

This is handled automatically when the version number changes (which happens on every PR merge via the version bump workflow). Claude Code checks the version in `marketplace.json` and re-fetches when it detects a new version.

If commands aren't updating after a merge:
1. Confirm the version bump commit landed on `main` (`git log --oneline -3` on this repo)
2. Run `/plugin marketplace update klira-spec-dev-marketplace` to force a refresh
3. Restart Claude Code if the update still doesn't appear

## Repository Structure

- `.claude-plugin/marketplace.json` — Root marketplace manifest (canonical version source)
- `plugins/spec-workflow/` — Plugin with commands and skills
- `.github/workflows/version-bump.yml` — Auto version bump workflow
- `thoughts/` — Shared research, plans, and contracts (managed by `thoughts` CLI)
