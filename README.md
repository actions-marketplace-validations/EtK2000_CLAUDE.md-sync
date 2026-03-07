# CLAUDE.md-sync

> **Warning:** This project is vibecoded slop. Proceed with caution.

A GitHub Action that keeps `CLAUDE.md` in sync across repos. This repo holds the canonical source-of-truth; consumer repos use the action to automatically open PRs when the file drifts.

## Quick Start

Add this workflow to any consumer repo at `.github/workflows/sync-claude-md.yml`:

```yaml
name: Sync CLAUDE.md
on:
  schedule:
    - cron: '0 9 * * 1'   # Every Monday at 9 AM UTC
  workflow_dispatch:        # Manual trigger

permissions:
  contents: write
  pull-requests: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: EtK2000/CLAUDE.md-sync@v1
        with:
          source-repo: EtK2000/CLAUDE.md
```

## Inputs

| Input           | Default                               | Description                                             |
|-----------------|---------------------------------------|---------------------------------------------------------|
| `source-repo`   | **required**                          | `owner/repo` holding the canonical file                 |
| `source-ref`    | `master`                              | Branch or tag to fetch from                             |
| `source-path`   | `CLAUDE.md`                           | Path in the source repo                                 |
| `target-path`   | `CLAUDE.md`                           | Where to place the file locally                         |
| `target-branch` | *(auto-detected)*                     | Branch the PR targets (defaults to repo default branch) |
| `pr-title`      | `chore: sync CLAUDE.md from upstream` | PR title                                                |
| `branch-name`   | `claude-md-sync/update`               | Branch used for the sync PR                             |
| `pr-labels`     | `automated`                           | Comma-separated PR labels                               |
| `token`         | `${{ github.token }}`                 | GitHub token                                            |

## Required Permissions

The token needs:
- `contents: write` — to push the sync branch
- `pull-requests: write` — to create/comment on PRs

The default `GITHUB_TOKEN` works if you set the `permissions` block as shown above.

## Behavior

- **Files match** — no-op, exits cleanly.
- **Files differ, no open PR** — creates a new branch, commits the updated file, opens a PR.
- **Files differ, PR already open** — force-pushes the branch and comments on the existing PR.
- **Source fetch fails** — exits with an error.

## Setup (Consumer Repos)

In each consumer repo, go to **Settings > Actions > General > Workflow permissions** and enable:
- **"Allow GitHub Actions to create and approve pull requests"**

Without this, the action will fail when trying to open PRs.

## Development

Run ShellCheck locally:

```bash
shellcheck sync.sh
```

### Security

This repo has **"Require actions to be pinned to a full-length commit SHA"** enabled. When adding or updating action dependencies, always pin to a commit SHA:

```yaml
# Good
- uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4

# Bad
- uses: actions/checkout@v4
```