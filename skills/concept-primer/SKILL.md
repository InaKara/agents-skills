---
name: concept-primer
description: 'Generates a concise, self-contained .md explanation of a single technical concept. Saves to docs/concepts/. ALWAYS invoke via runSubagent — never inline — to isolate concept generation from the calling session context.'
argument-hint: 'Concept name and user familiarity level (e.g. "Never heard of it")'
keywords: [concept, explanation, documentation, learning, primer, teaching, technical-writing, knowledge-base]
tools:
  - file_search
  - create_file
---

# Concept Primer Skill

## Purpose

Generate a concise, general-purpose explanation of a single technical concept and save it as a `.md` file in `docs/concepts/`. Files are reusable across projects and sessions — once generated, they persist as a personal concept library.

**Always invoked via `runSubagent`.** Context isolation is the core design constraint: concept generation must never consume the calling session's context window.

---

## Interface

You will receive two inputs in the invocation prompt:

- **concept**: the concept name (e.g., `"async/await"`, `"Dependency Injection"`, `"Context Manager"`)
- **familiarity**: the user's stated familiarity (e.g., `"Never heard of it"`, `"Heard of it but haven't used it"`, `"Used it a bit"`)

---

## Execution

### Step 1 — Normalize the file name

Convert the concept name to a slug: lowercase, spaces and slashes → hyphens, remove special characters.

| Input | Slug |
|---|---|
| `"async/await"` | `async-await` |
| `"Dependency Injection"` | `dependency-injection` |
| `"REST vs GraphQL"` | `rest-vs-graphql` |
| `"__init__.py"` | `init-py` |

Target path: `docs/concepts/<slug>.md`

### Step 2 — Check for an existing file

Use `file_search` to check if `docs/concepts/<slug>.md` already exists.

If it **exists**: stop immediately. Return: `"Skipped: docs/concepts/<slug>.md already exists."`

### Step 3 — Write the explanation

Generate the explanation following the **File Template** section below.

Length target: **350–550 words** of body text (not counting headings and table markup). Do not exceed 650 words.

Calibrate depth to the user's familiarity:

- `"Never heard of it"` → explain from zero; lead with a strong real-world analogy before any technical detail
- `"Heard of it but haven't used it"` → assume they know the name and rough purpose; focus on mechanics and how it looks in code
- `"Used it a bit"` → focus on mental model, comparison to alternatives, and misconceptions

### Step 4 — Save

Use `create_file` to save the file at `docs/concepts/<slug>.md`. The directory will be created automatically if it does not exist.

### Step 5 — Return

Return exactly one line: `"Saved: docs/concepts/<slug>.md"`

---

## File Template

Every generated file must follow this structure exactly. Do not add, remove, or reorder sections.

```
---
keywords: [<comma-separated lowercase terms that describe or relate to the concept, 5-10 words>]
---

# `<Concept Name>`

> **In one sentence:** <what it is and what problem it solves, max 25 words>

## The Problem It Solves

<2–4 sentences. Describe a concrete scenario where absence of this concept causes pain. No abstract theory — make it tangible.>

## The Mental Model

<3–5 sentences. Build intuition using one concrete analogy from everyday life. Introduce technical terms only after the analogy lands.>

## How It Works

<Mechanics in plain prose (3–5 sentences). Then one minimal, self-contained, runnable code example.>

```python
# example
```

## Comparison to Alternatives

<1–2 sentences framing what you would do without this concept. Then:>

| Approach | Typical use | Downside |
|---|---|---|
| This concept | ... | ... |
| Without it (alternative) | ... | ... |

## Common Misconceptions

- <misconception 1 — state the wrong belief, then correct it>
- <misconception 2>
- <misconception 3 if applicable>
```

---

## Sample Output

The following is a correctly filled-in example for the concept **"Context Manager"**. Use it as the quality and length reference for all generated files.

---

---
keywords: [context-manager, with, enter, exit, resource-cleanup, try-finally, contextlib, python, file-handling, exception-safety]
---

# `Context Manager`

> **In one sentence:** A context manager wraps a block of code and guarantees that setup and cleanup both run, even if the block raises an exception.

## The Problem It Solves

Working with resources — files, database connections, network sockets, locks — requires cleanup after use. If you open a file and an exception fires before `file.close()`, the file stays open for the rest of the program's life. Wrapping every resource access in `try/finally` manually is verbose and easy to omit.

## The Mental Model

Think of a hotel room key card. Checking in (entering the `with` block) hands you the card and turns on the AC. Checking out (leaving the block — whether you walk out normally or the fire alarm goes off) triggers automatic room cleanup and card deactivation. You never need to remember the checkout steps yourself; the hotel guarantees them regardless of why you left.

## How It Works

Python's `with` statement calls `__enter__` when entering the block and `__exit__` when leaving — on both normal exit and exceptions. The most common use is opening files, but the mechanism is general.

```python
with open("data.txt") as f:
    content = f.read()
# f is guaranteed closed here, even if f.read() raised an exception
```

You can define your own context manager using `contextlib.contextmanager`:

```python
from contextlib import contextmanager

@contextmanager
def timer():
    start = time.time()
    yield                          # code inside the `with` block runs here
    print(f"Elapsed: {time.time() - start:.2f}s")

with timer():
    do_something_slow()
```

## Comparison to Alternatives

Without a context manager, you write explicit `try/finally` at every call site that uses the resource:

```python
f = open("data.txt")
try:
    content = f.read()
finally:
    f.close()   # must be repeated at every single call site
```

| Approach | Typical use | Downside |
|---|---|---|
| `with` (context manager) | Any resource needing guaranteed setup/teardown | Object must implement `__enter__`/`__exit__` |
| `try/finally` | One-off cleanup without a reusable abstraction | Verbose; cleanup logic duplicated at every call site |
| Manual close (no guard) | — | Any exception permanently skips cleanup |

## Common Misconceptions

- **"`with` catches exceptions."** It does not. `__exit__` is called regardless, but exceptions still propagate unless `__exit__` explicitly returns `True`.
- **"Context managers are only for files."** No — they apply to locks, database transactions, temporary directories, network connections, and any resource with a paired setup/teardown lifecycle.
- **"`as f` is required."** It is optional — `with open("data.txt"):` is valid when you do not need the file handle returned by `__enter__`.
