---
name: ctx-add-index
description: Scan workspace repos and generate context/index.md — a comprehensive big-picture map of the entire project including domain glossary, services, flows, and dependencies.
metadata:
  author: nicolasse
  version: "1.0.0"
---

# Generate Project Index

Scans all repos in the workspace and generates `context/index.md` — a comprehensive big-picture map of the entire project. This index is the entry point for understanding the system before creating new features.

## When to use

- After `ctx-init` to create the first full map
- When the project has evolved significantly and you need a fresh snapshot
- Before creating a new feature, to understand the landscape

## Output

A single file: `context/index.md` — overwritten each time this skill runs.

## Workflow

### 1. Discover repos

List all directories at the workspace root. Each directory with a `.git` folder is a repo. Skip `context/` and any non-repo directories (no `.git`).

Record the full list of repos found.

### 2. Deep scan all repos

**Use a single agent** to explore all repos thoroughly. The agent must build a complete understanding of:

**Per repo:**
- What it actually does (read code, not just README)
- Tech stack (language, framework, runtime)
- Entry points (main files, handlers, route definitions)
- Events/triggers it receives (HTTP, gRPC, Kafka, SQS, cron, webhooks)
- Events/actions it emits (publishes to queues, calls other services, writes to DB)
- User-facing configurations that affect behavior
- Key internal structure (folders, architecture patterns)

**Cross-repo:**
- Shared databases (same connection strings, same tables)
- Event bus (Kafka topics, SQS queues — who produces, who consumes)
- HTTP/RPC calls between services
- Shared libraries and proto/contract files
- DNS/proxy routing

**Domain concepts:**
- Identify the core domain entities (the nouns of the system)
- For each: what it is, possible states/types, where it lives (source of truth), which repos consume it

**End-to-end flows:**
- Identify the main flows a user or external system would trigger
- Trace each flow step-by-step across repos
- Note which domain concepts and configurations affect each flow

**Exploration instructions for the agent:**
- Read key source files — don't just list directories
- Check config files, environment variables, queue names, topic names
- Follow imports to understand how code flows across boundaries
- Look at event handlers, consumers, producers to map the bus
- Check package.json / pyproject.toml / go.mod for stack and dependencies
- Look at proto files, shared types, API contracts
- Read tests to understand expected behavior

### 3. Generate index.md

From the deep scan, generate `context/index.md` following this exact structure. Write in **English**. Be thorough but concise — every line should add information.

```markdown
# {Project Name} — Context Map

> Auto-generated. Last updated: {YYYY-MM-DD}
> Repos analyzed: {list of all repo names}

---

## 1. Domain Glossary

### {ENTITY_NAME}
- **What it is**: {One sentence definition}
- **Possible states/types**: {list, or "N/A" if stateless}
- **Where it lives**: {repo(s) that own/define it}
- **Consumed by**: {repo(s) that use it}

{Repeat for each core domain entity. Order from most central to most peripheral.}

---

## 2. Services

### {service-name}
- **Main responsibility**: {What this service actually does — one sentence}
- **Stack**: {language, framework, runtime, DB}
- **Receives**: {events, triggers, HTTP calls it handles — be specific about topics, queues, RPC methods}
- **Emits**: {events it publishes, services it calls, queues it writes to}
- **User configs that affect behavior**: {settings, flags, env vars that change what it does}
- **Key cross-repo dependencies**: {which other repos it talks to and how}

{Repeat for each service/repo. For monorepos, list sub-services individually.
 For groups of similar services (e.g., connectors), use a summary + table format:}

### {group-name} — Summary

{One paragraph explaining the shared pattern.}

| Service | Stack | Specifics |
|---|---|---|
| `service-a` | Node.js | {one-line differentiator} |
| `service-b` | Go | {one-line differentiator} |

{Also list shared libraries and tools as tables if they exist:}

### Shared Libraries

| Library | Stack | Responsibility |
|---|---|---|
| `lib-a` | Go | {one line} |

---

## 3. Flows

### Flow: {Flow name}
- **Trigger**: {What initiates this flow}
- **Steps**:
  1. [{repo/service}] → {what happens}
  2. [{repo/service}] → {what happens}
  3. [{repo/service}] → {what happens}
- **Repos involved**: {list}
- **Related domain terms**: {list of glossary entries}
- **User configs that affect this flow**: {list}

{Repeat for each major end-to-end flow. Include both happy-path user flows and system/cron flows.}

---

## 4. Dependency Map

| Source repo | Target repo | Communication type | Context |
|---|---|---|---|
| `repo-a` | `repo-b` | gRPC | {method or purpose} |
| `repo-b` | `repo-c` | Kafka | {topic name} |
| `repo-a` | `repo-d` | HTTP REST | {endpoint or purpose} |
| `repo-c` | external-service | HTTP | {what for} |

{Include ALL cross-repo and cross-service communication. Also include shared DB, shared queues, proxy routing.}

---

## 5. Additional Details per Repo

### {repo-name} — {aspect}

{Deeper structural information that doesn't fit in the Services section but is valuable for understanding the system. Examples:}

- API route maps
- Event handler lists
- Architecture patterns (hexagonal, layered, etc.)
- Lambda/function inventories
- Key internal modules

{Only include details that add value. Skip repos that are fully covered by their Services entry.}

---

## 6. Implementation Notes

### Key Patterns
{Numbered list of architectural patterns used across the system (event sourcing, CQRS, saga, etc.) with one-line explanation each.}

### Conventions
{Notable naming conventions, tagging systems, internal protocols.}

### Critical Environment Variables
```
VAR_NAME    # what it does
VAR_NAME    # what it does
```
{Only the variables that are essential for understanding how services connect and behave.}
```

### 4. Review with user

Show the generated `index.md` to the user. Iterate if they want changes. Write the final version.

### 5. Summary

Tell the user:
- How many repos were scanned
- How many domain entities, services, flows were documented
- Any gaps or areas that need manual review (e.g., repos with sparse code, unclear responsibilities)
