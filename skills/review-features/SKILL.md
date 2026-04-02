---
name: review-features
description: Interactively select one or more features and choose which review layers (product, engineering, implementation drift) to run against their contracts. Read-only.
---

# Review Features

Lets the developer pick one or more features and choose which contract review layers to run. Spawns the contract-reviewer agent for each selected feature and presents a combined report. Read-only — no code or context file modifications.

## Workflow

### 1. Discover Features

List all directories under `_ctx/` (excluding `_template/`). Filter to only those that contain all three context files (`product.md`, `engineering.md`, `implementation.md`). These are the reviewable features.

If no reviewable features are found, inform the user and stop.

### 2. Feature Selection

Use the `AskUserQuestion` tool to present the available features and let the user select which ones to review.

Format the question clearly:

```
Which features would you like to review? (comma-separated numbers, or "all")

1. feature-one
2. feature-two
3. feature-three
```

Parse the user's response. Accept:
- A single number (e.g., "1")
- Comma-separated numbers (e.g., "1, 3")
- "all" to select every feature

If the response is unclear, ask again.

### 3. Layer Selection

Use the `AskUserQuestion` tool to ask which review layers to run:

```
Which review layers should I run?

1. Product contract only
2. Product + Engineering contracts
3. All three (product + engineering + implementation drift)
```

Parse the user's response. Default to option 3 if unclear.

Map the selection:
- **Option 1:** Run only steps 4 (Product Contract Review) from the contract-reviewer workflow
- **Option 2:** Run steps 4 and 5 (Product + Engineering Contract Review)
- **Option 3:** Run steps 4, 5, and 6 (Product + Engineering + Implementation Drift)

### 4. Run Reviews

For each selected feature, spawn the **contract-reviewer agent** (from `agents/contract-reviewer.md`) with:
- The feature name
- Instructions to run only the selected review layers (from step 3)

If multiple features were selected, spawn one agent per feature using the `Agent` tool so they run in parallel.

Each agent should produce its standard structured report, limited to the selected layers.

### 5. Aggregate and Report

Collect all review reports. Present a combined summary to the user:

```
## Contract Review Summary

### {feature-one}
#### Product Contract
- UC-001: Pass
- UC-002: Warning — ...

#### Engineering Contract
- Interfaces: Pass
- ...

### {feature-two}
...

---

**Totals:**
- Violations: N
- Warnings: N
- Suggestions: N (implementation drift)
```

Only include sections for the layers the user selected in step 3.

If any violations were found, highlight them at the top of the summary.

## Constraints

- This skill is strictly **read-only** — it must never modify code or context files.
- Do not create, edit, or delete any files.
- If a feature is missing one or more context files, skip it and note why in the report.
