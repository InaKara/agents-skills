---
description: Python coding standards — clean, modular, DRY code
applyTo: "**/*.py"
---

## Core Principles

1. **DRY (Don't Repeat Yourself)** — Never duplicate logic. If the same block of code appears (or would appear) in more than one place, extract it into a reusable function or class method immediately.
2. **Single Responsibility** — Every function and class does exactly one thing. If a function name needs "and" to describe it, split it.
3. **Small Functions** — Keep functions short (aim for ≤ 25 lines of logic). Long functions are a sign that extraction is needed.

## Function Design

- Prefer **pure functions** (no side effects, deterministic output) wherever possible.
- Accept explicit parameters instead of relying on global state.
- Return values instead of mutating inputs. If mutation is unavoidable, document it clearly.
- Use **default parameter values** and `**kwargs` sparingly — only when they genuinely simplify the caller's life.

## Module & File Structure

- Group related functions into modules; group related modules into packages.
- Use a flat import style (`from mypackage.utils import helper`) over deep nesting.
- Keep a clear separation between:
  - **Core logic** (pure computation, no I/O)
  - **I/O / integration** (file, network, database)
  - **Entry points / CLI** (`if __name__ == "__main__"` or click/typer commands)
- Place shared utilities in a dedicated `utils.py` or `helpers.py` rather than scattering them across feature modules.

## Naming Conventions

- `snake_case` for functions, methods, variables, and module names.
- `PascalCase` for classes.
- `UPPER_SNAKE_CASE` for module-level constants.
- Prefix private helpers with a single underscore (`_internal_helper`).
- Names must be descriptive: `calculate_total_price` over `calc`, `user_records` over `data`.

## Code Reuse Patterns

- **Extract, don't copy.** Before writing new code, search the codebase for existing functions that solve the same problem.
- Use **composition over inheritance** — small, focused functions composed together beat deep class hierarchies.
- When multiple functions share setup/teardown logic, use **context managers** (`with` statements) or decorators.
- For repeated data transformations, consider building a small **pipeline** of composable functions.

## Error Handling

- Handle errors at the **boundary** (entry points, I/O layers), not deep inside pure logic.
- Raise specific exception types (`ValueError`, `FileNotFoundError`) instead of bare `Exception`.
- Never use bare `except:` — always catch specific exceptions.
- Use `try/except` blocks narrowly around the exact call that may fail.

## Imports

- Standard library → third-party → local application (separated by blank lines).
- Avoid wildcard imports (`from module import *`).
- Prefer absolute imports over relative imports.

## Testing Alignment

- Write functions that are easy to test: accept inputs, return outputs, minimal side effects.
- If a function is hard to test, that is a design signal — refactor it.
- Keep test helpers in a shared `conftest.py` or `test_utils.py` to avoid duplication in tests too.

## Anti-Patterns to Reject

- Copy-pasted blocks of code with minor variations.
- "God functions" that do everything (fetch, process, format, save).
- Hardcoded values that should be parameters or constants.
- Deeply nested `if/else` trees — flatten with early returns or guard clauses.
- Mixing business logic with I/O in the same function.
