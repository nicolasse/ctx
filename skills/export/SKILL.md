---
name: export
description: Scan workspace repos and export context/config.json with repo list and context link. Use to share or rebuild the workspace.
---

# Export Context

Scans root-level directories for git repos and creates/updates `context/config.json` so anyone can recreate the workspace with `/ctx-pull-repos`.

## Workflow

### 1. Ensure context directory

If `context/` doesn't exist, create it.

### 2. Scan repos

List all directories at the workspace root. For each directory:

- Skip `context/` — it's the context repo, not a code repo
- Skip `node_modules/`, `.git/`, and other common non-repo directories
- Check if the directory contains a `.git` folder
- If it does, read its git remote origin URL (`git -C {dir} remote get-url origin`)
- If it has no remote, warn the user and skip it

Collect the remote URL string for each found repo.

### 3. Ask for context link

Ask the user: **"What's the GitHub URL for the context repo? (leave empty if you don't have one yet)"**

- If the user provides a URL, use it
- If the user skips or says they don't know, set `context` to `""` and tell them: "Remember to set the `context` field in `context/config.json` once you push the context repo."

### 4. Load existing config

If `context/config.json` already exists, read it. Merge:

- **Repos**: replace the entire `repos` array with the fresh scan (the filesystem is the source of truth)
- **Context**: keep the existing value unless the user provided a new one

### 5. Write config.json

Write `context/config.json`:

```json
{
  "context": "{url or empty string}",
  "repos": [
    "https://github.com/org/repo-a.git",
    "https://github.com/org/repo-b.git"
  ]
}
```

Sort repos alphabetically by URL.

### 6. Report

Print a summary:

- **Context URL**: the value set (or a reminder to configure it)
- **Repos found**: list of repos with their URLs
- **Skipped**: directories that were skipped and why (no `.git`, no remote, etc.)

If `context` is empty, remind the user: "Don't forget to set the `context` URL in `context/config.json` after pushing the context repo to GitHub."
