---
description: Python documentation standards — docstrings and inline comments
applyTo: "**/*.py"
---

## General Rule

Every public module, class, function, and method **must** have a docstring. Inline comments are required wherever the intent is not immediately obvious from the code itself.

## Docstring Format (Google Style)

Use **Google-style docstrings** consistently across the entire codebase.

### Functions and Methods

```python
def calculate_discount(price: float, rate: float, min_price: float = 0.0) -> float:
    """Calculate the discounted price, clamped to a minimum.

    Applies the given discount rate to the original price and ensures
    the result does not fall below min_price.

    Args:
        price: Original price before discount.
        rate: Discount rate as a decimal (e.g., 0.15 for 15%).
        min_price: Floor value for the discounted price. Defaults to 0.0.

    Returns:
        The discounted price, no lower than min_price.

    Raises:
        ValueError: If rate is not between 0 and 1.
    """
```

### Classes

```python
class OrderProcessor:
    """Processes customer orders from validation through fulfillment.

    Coordinates validation, payment, and shipping steps for a single
    order. Each step is handled by a dedicated method to keep the
    pipeline testable and composable.

    Attributes:
        order_id: Unique identifier for the order being processed.
        status: Current processing status (e.g., "pending", "paid", "shipped").
    """
```

### Modules

Place a module-level docstring at the very top of the file (after any shebang or encoding line):

```python
"""Utility functions for date and time manipulation.

Provides timezone-aware helpers used across the scheduling and
reporting modules. All functions accept and return UTC datetimes
unless stated otherwise.
"""
```

## Docstring Rules

1. **First line** — a single imperative sentence summarizing the purpose (≤ 79 chars). End with a period.
2. **Blank line** after the summary, then an optional extended description.
3. **Args** — list every parameter with its type (if not annotated) and meaning.
4. **Returns** — describe the return value and its type.
5. **Raises** — list exceptions the function intentionally raises.
6. Omit sections that don't apply (e.g., skip `Raises` if none are raised).
7. Use type annotations in the signature; do **not** duplicate types in the docstring when annotations are present.

## Inline Comments

- Use inline comments to explain **why**, not **what**.
  ```python
  # Retry up to 3 times because the upstream API is flaky under load
  for attempt in range(3):
      ...
  ```
- Place the comment on the line **above** the code it explains, not at the end of the line (unless the comment is very short and directly tied to that expression).
- Do **not** state the obvious:
  ```python
  # BAD: Increment counter
  counter += 1

  # GOOD: Compensate for the off-by-one in the API's pagination index
  counter += 1
  ```

## When to Document

| Situation | Required |
|---|---|
| Public function / method / class | Always |
| Private helper (`_helper`) | Docstring if logic is non-trivial |
| Module-level constant | Inline comment explaining purpose |
| Complex conditional or algorithm | Inline comment explaining reasoning |
| Workaround or hack | Inline comment with context and link to issue if applicable |
| Trivial one-liner property | May omit docstring if name is self-explanatory |

## Documentation at Creation Time

- Write docstrings and comments **while writing the code**, not as a separate pass.
- When modifying an existing function, update its docstring if the behavior, parameters, or return value changed.
- When extracting a new function (DRY refactor), always add a docstring immediately.

## Anti-Patterns to Reject

- Empty or placeholder docstrings (`"""TODO"""`, `"""..."""`).
- Docstrings that only repeat the function name (`"""Calculate discount."""` on `calculate_discount`).
- Commented-out code left without explanation — remove it or add a justification comment.
- Excessive inline comments that narrate every line instead of explaining intent.
