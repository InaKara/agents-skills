---
description: Central function registry — document every function, check before writing new ones
applyTo: "**/*.py"
---

## Purpose

Maintain a **central function registry** at `docs/function-registry.md` that serves as
the single source of truth for all reusable functions in the project. Agents use this
registry to discover existing functionality before writing new code and to keep the
documentation in sync with the codebase at all times.

## Registry File Location

```
<project-root>/docs/function-registry.md
```

If the file or directory does not exist, create them.

## Registry Format

The registry is a Markdown file with one table per module/file. Each row documents a
single function.

```markdown
# Function Registry

> Auto-maintained by Copilot. Do not remove entries manually — update them through code changes.

## `module_name.py`

| Function | Description |
|---|---|
| `calculate_discount(price, rate, min_price)` | Apply a discount rate to a price, clamped to a floor value. |
| `validate_order(order_dict)` | Check that all required order fields are present and well-formed. |

## `utils/helpers.py`

| Function | Description |
|---|---|
| `retry(fn, attempts, delay)` | Retry a callable up to N times with exponential back-off. |
```

### Column rules

- **Function** — function name with parameter names (no types, no defaults). Use backtick formatting.
- **Description** — one sentence, imperative mood, ≤ 120 characters. Describes *what* the function does, not *how*.

## Mandatory Agent Behaviors

### 1. Check Before Writing

Before implementing **any** new function, the agent **must**:

1. Read `docs/function-registry.md`.
2. Search for functions that already solve the same or a similar problem.
3. If a match exists, **reuse** the existing function (import it) instead of creating a duplicate.
4. If the existing function almost fits, **extend** it (add a parameter, generalize) rather than creating a near-copy.

### 2. Document on Creation

When a new function is added to **any** Python file:

1. Add a row to the appropriate module table in `docs/function-registry.md`.
2. If the module table does not exist yet, create a new `## \`<filename>\`` section.
3. The entry must be added in the **same response / edit batch** as the code change — never defer it.

### 3. Update on Change

When an existing function is **renamed, moved, or has its signature/behavior changed**:

1. Update the corresponding row in `docs/function-registry.md` immediately.
2. If the function was moved to a different module, move the row to the correct section.
3. If the function was deleted, remove the row.

### 4. Never Skip

- These rules apply to **every** code change that adds, modifies, or removes a Python function.
- A task is **not complete** until the registry is consistent with the code.
- If the agent is unsure whether a function is already registered, it must check the registry — do not guess.

## Anti-Patterns

- Creating a new utility function without checking the registry first.
- Finishing a code change and forgetting to update `docs/function-registry.md`.
- Documenting only "important" functions — **all** public functions must be registered.
- Letting the registry drift out of sync with the actual codebase.
