---
name: modify-feature
description: Modify an existing feature by describing what should change. Loads full context, spawns a feature-modifier agent, validates, and updates context files.
---

# Modify Feature

Modifies an existing feature's behavior or structure. The developer describes the change in conversation; the skill loads the feature's full context, delegates implementation to a feature-modifier agent, validates the result, and updates context files.

## Input

The user describes what they want to change. Can be behavioral ("make follow-ups retry 3 times instead of once") or structural ("move the PDF parser into its own service").

## Workflow

### 1. Feature Resolution

List all directories under `context/`.
Match the user's request to the most likely feature directory based on semantic understanding.

**If confident** (single obvious candidate), confirm and proceed.
**If ambiguous**, list the available features and ask the user to pick.

Never create a new feature directory. The feature must already exist with all three context files.

### 2. Load Context

Read all three context files:
- `context/{feature}/product.md`
- `context/{feature}/engineering.md`
- `context/{feature}/implementation.md`

If any file is missing, inform the user and suggest mapping the feature first.

### 3. Clarify the Change

Understand what the developer wants to change. Ask at most 2-3 clarifying questions if needed:
- What specific behavior should change?
- Should existing behavior be preserved or replaced?
- Any constraints on how it should be done?

Stop asking when you have enough to describe the change clearly.

### 4. Implementation

Spawn a **feature-modifier agent** with:
- The feature name
- The change description (from the conversation)
- The full content of all three context files

The agent implements the code changes and reports back.

### 5. Validation

After the agent completes, validate:

**Product contract (`product.md`):**
- Do all use cases still work as described (unless the change explicitly modifies them)?
- Are business rules still enforced?

**Engineering contract (`engineering.md`):**
- Are interfaces and contracts respected (unless the change explicitly modifies them)?
- Do patterns and conventions still match?

**If violations are found:** report them. Fix if straightforward, otherwise ask the developer.

### 6. Context Update

Update context files to reflect the new state:

1. **`product.md`** — Update only if the change altered use cases, business rules, or scope.
2. **`engineering.md`** — Update only if the change altered interfaces, constraints, or patterns.
3. **`implementation.md`** — Update to reflect what changed: modified files, updated flows, new data model entries, today's date in recent changes.

Show the context file changes to the user before writing.

### 7. Handoff

Do NOT commit. Leave all changes for the developer to review.

Provide a summary:
- What was changed (code)
- What context files were updated and why
- Validation result
- Any open questions or suggestions
