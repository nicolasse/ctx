---
name: ctx-pull-repos
description: Clone all repos from config.json into repositories/. Skips repos that already exist locally. Use to sync your workspace with the configured repo list.
metadata:
  author: nicolasse
  version: "1.0.0"
---

# Pull Repos

Reads the repo list from `features/config.json` at the workspace root and clones any missing repos into `repositories/`.

## Workflow

### 1. Read config

Read `features/config.json`. Parse the `repos` array. Each entry has:
- `name` — directory name inside `repositories/`
- `url` — git clone URL

If `features/config.json` doesn't exist or has no `repos` key, tell the user and stop.

If the `repos` array is empty, tell the user there are no repos configured and suggest they add entries to `features/config.json`.

### 2. Clone missing repos

For each repo in the array:

- Check if `repositories/{name}` already exists
- If it exists, mark as **skipped**
- If it doesn't exist, run `git clone {url} repositories/{name}` and mark as **cloned**

If a clone fails, report the error but continue with the remaining repos.

### 3. Report

Print a summary:

- **Cloned**: list of repos that were cloned
- **Skipped** (already existed): list of repos that were skipped
- **Failed**: list of repos that failed to clone, with error details

If everything was skipped, say all repos are already present.
