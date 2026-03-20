---
description: Initialize thoughts in the current workspace. Apply automatically when the thoughts/ directory is missing and the workspace is a git repo.
user-invocable: false
---

# Thoughts Initialization

When working in a git repository that doesn't have a `thoughts/` directory (or it exists but has no symlinks), initialize it by running:

```bash
thoughts init
```

If `thoughts/shared` exists as a real directory (not a symlink), the CLI will migrate its contents to the central repo and replace it with a symlink when run with `--force`:

```bash
thoughts init --force
```

## Detection

- Check if `thoughts/shared` or `thoughts/shared/issues` exists as a valid symlink
- If missing or is a plain directory (not a symlink), the workspace needs thoughts initialized

## Worktree/Workspace Behavior

- The CLI auto-detects worktrees and reuses the main repo's thoughts mapping via `resolveRepoMapping()`, which falls back to the main repo's mapping — so the repo name will be correct even in worktrees
- No special flags needed — just run `thoughts init`
- It will create symlinks pointing to the same central thoughts repo

## Do NOT run if:

- `thoughts/shared` is already a valid symlink
- The user explicitly opted out of thoughts
- Not in a git repository
