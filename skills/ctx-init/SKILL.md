---
name: ctx-init
description: Initialize a ctx workspace — name the project, explore all repos, and generate a workspace map. Use when setting up a new multi-repo project with ctx.
metadata:
  author: nicolasse
  version: "1.0.0"
---

# Initialize Workspace

Sets up a ctx workspace: names the project, scans all repos, and generates the workspace map.

Run this once after cloning and adding your repos to `repositories/`.

## Workspace Setup

Before scanning repos, ensure the workspace has the required structure. If any of these are missing, create them:

- `repositories/` — where all repos live
- `features/` — where feature context files live
- `features/_template/` — with the three template files (product.md, engineering.md, implementation.md)
- `features/CLAUDE.md` — agent instructions for the features directory
- `CLAUDE.md` — workspace-level instructions

The templates and instructions are bundled with this skill. If the files don't exist, generate them from the templates below.

### features/CLAUDE.md

```markdown
# Features Context — Agent Instructions

This directory contains the product, engineering, and implementation context for each feature in the system. These files are the source of truth that AI agents use to understand, implement, and validate changes.

**Read this file before operating on any feature.**

## Structure

features/
  _template/          # Templates for creating new features
  {feature-name}/
    product.md        # WHAT: product definition, use cases, business rules
    engineering.md    # HOW: technical contracts, interfaces, constraints
    implementation.md # CURRENT STATE: how it's built today, key files, flows

## Conventions

- Directory names use kebab-case: `follow-up/`, `web-chat/`, `voice-pipeline/`
- Each feature MUST have all three files
- `product.md` is the product contract — changes here mean the feature's behavior changes
- `engineering.md` is the technical contract — changes here may be breaking
- `implementation.md` is descriptive, not prescriptive — it reflects reality, not aspirations
- `_template/` contains starter files for new features, do not modify

## For Agents

When resolving a feature name from a user request, list the directories here and match semantically. Users may refer to features by aliases, abbreviations, or in different languages. If uncertain, ask.
```

### features/_template/product.md

```markdown
# {Feature Name}

{One or two sentences: what this does and why it matters.}

## Use Cases

- **UC-001: {Name}** — {One sentence describing the behavior.}
- **UC-002: {Name}** — {One sentence describing the behavior.}

## Business Rules

- {Rule in plain language}
- {Rule in plain language}

## Out of Scope

- {What this feature does NOT do}
```

### features/_template/engineering.md

```markdown
# {Feature Name} — Engineering

## Architecture

{Which repos/services are involved, 2-3 sentences max.}

## Key Interfaces

{Only the interfaces that matter — the ones that if you change them, things break. Keep it short.}

## Constraints

- {Technical constraint in plain language}
- {Technical constraint in plain language}

## Patterns

- {How things are done in this feature, one line each}
```

### features/_template/implementation.md

```markdown
# {Feature Name} — Implementation

## Current State

{One or two sentences: what's built, how mature is it.}

## Key Files

| Repo | Path | What it does |
|---|---|---|
| `repo-a` | `src/...` | {one line} |
| `repo-b` | `src/...` | {one line} |

## Flows

### UC-001: {Name}

1. {Step} → 2. {Step} → 3. {Step}

### UC-002: {Name}

1. {Step} → 2. {Step} → 3. {Step}

## Data Model

{Only the key tables/collections. Schema snippet only if it's short.}

## Known Limitations

- {What's missing or broken}

## Recent Changes

- {date}: {what changed}
```

### CLAUDE.md (workspace root)

```markdown
# Workspace

This is a multi-repo workspace.

## Structure

- `repositories/` — All repos live here. Each subdirectory is a repo.
- `features/` — Product, engineering, and implementation context for each feature. See `features/CLAUDE.md`.

## Repos

The repos are all directories inside `repositories/`. To discover available repos, list that directory. Do not hardcode repo names — they may change.

## Features Context

The `features/` directory contains the source of truth for understanding, implementing, and validating changes across repos.

When working on a feature, always check `features/` for existing context before exploring repos from scratch.
```

## Workflow

### 1. Project Name

Ask the user: **"What's the name of this project?"**

Rename the root folder of the workspace to the project name (lowercase, kebab-case). For example, if the user says "Darwin AI", rename the folder to `darwin-ai/`.

If the folder is already named something other than the default, confirm with the user before renaming.

### 2. Init features git

If `features/` doesn't have a `.git` directory, run `git init` inside it so the project context is its own repo from the start.

### 3. Discover repos

List all directories inside `repositories/`.

### 4. Deep scan each repo

For each repo, explore thoroughly to truly understand what it does:

- Read the README if it exists, but don't stop there
- Look at the project structure, entry points, route definitions, event handlers
- Read key source files — services, controllers, processors, models
- Check config files for queue names, event topics, API URLs, DB connections
- Look at tests to understand expected behavior
- Check package.json / pyproject.toml / go.mod for dependencies that reveal purpose
- Follow imports and references to understand how code flows

Use subagents to explore repos in parallel when possible.

**The goal is to understand what each repo actually does, not what its README says it does.** READMEs are often outdated. The code is the truth.

### 5. Map connections

From your deep exploration, identify how repos communicate:

- Shared databases (same DB connection strings, same table names)
- Event bus (Kafka topics, SQS queues — producer and consumer sides)
- HTTP/RPC calls (API clients calling other services)
- Shared libraries (common packages imported across repos)
- Proto/contract files (gRPC definitions, shared schemas)

### 6. Identify features

From what you've seen across repos, identify the main product features — the things a user of the product would recognize. A feature typically spans multiple repos. Group repos by which features they participate in.

### 7. Generate WORKSPACE.md

Distill everything into a concise map. You explored deep — now write short. Only the things that matter.

**Under 60 lines.** Think "napkin sketch of the architecture". Someone reads this in 2 minutes and gets the big picture.

```markdown
# {Project Name}

{One sentence: what this product/system is.}

## Repos

| Repo | Tech | What it does |
|---|---|---|
| `repo-a` | TS/Next.js | Frontend + BFF |
| `repo-b` | Go | Event processor |

## Connections

{How repos talk to each other. Keep it short — a few lines or a simple list.}

## Features

| Feature | Repos involved | Has context |
|---|---|---|
| Follow-ups | repo-a, repo-b, repo-c | No |
| Payments | repo-a, repo-d | No |
```

Write to `features/WORKSPACE.md`.

### 8. Summary

Tell the user:
- What you found
- How many repos, how many features identified
- Suggest which features to map first (the most cross-cutting or complex ones)

## Output

Show the user the generated map and ask for feedback before writing.
