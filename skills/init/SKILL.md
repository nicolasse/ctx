---
name: init
description: Initialize a ctx workspace — name the project, explore all repos, and generate a workspace map. Use when setting up a new multi-repo project with ctx.
---

# Initialize Workspace

Sets up a ctx workspace: names the project, scans all repos, and generates the workspace map.

Run this once after cloning your repos into the workspace root.

## Workspace Setup

Before scanning repos, ensure the workspace has the required structure. If any of these are missing, create them:

- `_ctx/` — where feature context files live
- `_ctx/README.md` — short description of the _ctx directory
- `_ctx/_template/` — with the three template files (product.md, engineering.md, implementation.md)
- `_ctx/CLAUDE.md` — agent instructions for the _ctx directory
- `CLAUDE.md` — workspace-level instructions

The templates and instructions are bundled with this skill. If the files don't exist, generate them from the templates below.

### _ctx/README.md

```markdown
Feature context for Claude Code — product definitions, engineering contracts, and implementation docs across repos.

Created with [ctx](https://github.com/nicolasse/ctx).
```

### _ctx/CLAUDE.md

```markdown
# Context — Agent Instructions

This directory contains the product, engineering, and implementation context for each feature in the system. These files are the source of truth that AI agents use to understand, implement, and validate changes.

**Read this file before operating on any feature.**

## Structure

_ctx/
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

### _ctx/_template/product.md

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

### _ctx/_template/engineering.md

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

### _ctx/_template/implementation.md

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

This is a multi-repo workspace managed with the ctx plugin.

## Structure

- Repos live at the workspace root. Each subdirectory with a `.git` folder is a repo.
- `_ctx/` — Context layer. Product, engineering, and implementation docs for features. NOT a code directory.

To discover repos, list directories at the workspace root and check for `.git` folders. Do not hardcode repo names.

## Default Workflow

**Every task follows this workflow. Do not skip steps.**

### 1. Load context first

Before doing anything else:

1. Read `_ctx/WORKSPACE.md` if it exists — this is the architecture map.
2. Read `_ctx/index.md` if it exists — this is the domain glossary and service map.
3. List `_ctx/` directories to see which features have context.

Never explore repos from scratch when context files already exist.

### 2. Resolve the feature

Match the user's request to a feature directory in `_ctx/`. Match semantically — users may use aliases, abbreviations, or different languages.

- Feature matches → read its three context files (`product.md`, `engineering.md`, `implementation.md`) before proceeding.
- Multiple features involved → load all relevant ones.
- No match → this is either a new feature or unmapped existing code. Route accordingly in step 3.

### 3. Route through a skill — do not code directly

Feature work MUST go through ctx skills. They handle context loading, agent spawning, contract validation, and context updates. Coding directly skips all of that.

| The user wants to... | Skill |
|---|---|
| **Change** an existing mapped feature | `ctx:modify-feature` |
| **Add a use case** to an existing feature | `ctx:add-use-case` |
| **Build** something new that doesn't exist | `ctx:add-feature` |
| **Map** existing code that has no context yet | Ask which files: `ctx:map-product`, `ctx:map-engineering`, `ctx:map-implementation` |
| **Review** contracts against current code | `ctx:review-features` |

Do NOT wait for the user to type a slash command. Determine the right skill automatically based on intent.

### 4. Validate after changes

After any code change to a mapped feature, validate against contracts:

- **Product contract**: All use cases still work? Business rules enforced?
- **Engineering contract**: Interfaces respected? Patterns followed?

If the skill already includes validation (modify-feature, add-use-case, add-feature do), this is covered. Otherwise, run `ctx:review-features`.

### 5. Keep context in sync

After implementing, context files must reflect the new state:

- `implementation.md` — always update after code changes
- `product.md` / `engineering.md` — update only if contracts changed

The ctx skills handle this automatically. If you bypassed a skill, update manually.

## When you can skip the full workflow

Some tasks don't need a skill:

- **Quick lookups** — "Where is X defined?" → search directly, but still check `_ctx/` first for a faster answer
- **Single-file bug fixes** — read the feature context first for understanding, but a full modifier agent isn't needed
- **Non-feature work** — CI, dependencies, tooling, infrastructure

Even for these, **always read relevant context from `_ctx/` before exploring repos**.

## Where to work

- **Code changes** → repo folders at the workspace root. Never in `_ctx/`.
- **Context changes** → only through ctx skills (`ctx:map-*`, `ctx:modify-feature`, etc.). Do not edit `_ctx/` files directly.

## Workspace setup

| Situation | Skill |
|---|---|
| No `_ctx/WORKSPACE.md` exists | `ctx:init` |
| Need to generate or refresh the project index | `ctx:add-index` |
| Need to export config for sharing | `ctx:export` |
| Need to clone repos from config | `ctx:pull-repos` |
| Update scaffolding after plugin update | `ctx:update` |
```

## Workflow

### 1. Project Name

Ask the user: **"What's the name of this project?"**

Rename the root folder of the workspace to the project name (lowercase, kebab-case). For example, if the user says "Darwin AI", rename the folder to `darwin-ai/`.

If the folder is already named something other than the default, confirm with the user before renaming.

### 2. Init context git

If `_ctx/` doesn't have a `.git` directory, run `git init` inside it so the project context is its own repo from the start.

### 3. Discover repos

List all directories at the workspace root. Each directory with a `.git` folder is a repo. Skip `_ctx/` — it's the context repo, not a code repo.

### 4. Deep scan each repo

For each repo, explore thoroughly to truly understand what it does:

- Read the README if it exists, but don't stop there
- Look at the project structure, entry points, route definitions, event handlers
- Read key source files — services, controllers, processors, models
- Check config files for queue names, event topics, API URLs, DB connections
- Look at tests to understand expected behavior
- Check package.json / pyproject.toml / go.mod for dependencies that reveal purpose
- Follow imports and references to understand how code flows

Use a single agent to explore all repos — cross-repo context matters more than parallelism.

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

Write to `_ctx/WORKSPACE.md`.

### 8. Summary

Tell the user:
- What you found
- How many repos, how many features identified
- Suggest which features to map first (the most cross-cutting or complex ones)

## Output

Show the user the generated map and ask for feedback before writing.
