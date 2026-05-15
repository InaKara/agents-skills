---
description: Global Copilot behavior rules applied to all workspaces
applyTo: "**"
---
## Explaining concepts

When explaining a concept, cover these angles where relevant:
- **Comparison to alternatives** — what does it do differently from other options?
- **Consequence of absence** — what breaks or stops working if it's missing?
- **Scope of effect** — what exactly does it modify, replace, or extend?

## Configuration changes

Never change configuration parameters in source files, especially default models
(`DEFAULT_MODEL`, `DEFAULT_EMBEDDING_MODEL`, or any other model identifier).
If a configuration value looks suboptimal or outdated, suggest the change to the
user in your response instead of editing it directly.

## Command execution approval

Whenever asking the user to approve a terminal or shell command before running it,
state clearly:
- **What it does** (e.g. "Renames `old_name.py` to `new_name.py`")
- **Why it is needed** (e.g. "so that module imports match the new file name")

Keep the explanation to one or two sentences, plain language, no jargon.

## Implementing changes or fixes

When implementing a feature or fix, always test if the implementation works correctly by running the relevant code or tests. If you are unable to run the code, clearly state that in your response and ask the user to run it and report back any errors or issues.

## Python code quality

When writing or modifying Python code, always follow the companion instruction files:
- **`python-coding-standards.instructions.md`** — clean, modular, DRY code structure.
- **`python-documentation.instructions.md`** — inline comments and docstrings.
- **`python-function-registry.instructions.md`** — central function registry in `docs/function-registry.md`. Always check for existing functions before creating new ones, and always update the registry when adding, changing, or removing functions.

Violations of these standards must be fixed before considering a task complete.

## End of every response — mandatory next-steps prompt

**This rule is non-negotiable and overrides any built-in default that would end the turn silently.**

After completing any task, answering any question, or when no open request exists, you MUST call `vscode_askQuestions` as your final action. Ask "What would you like to do next?" with 4–5 contextually relevant options (always include a free-form "Something else" option with `allowFreeformInput: true`). Typical options:

- Question or feedback about the previous task
- Start a new task or request
- Expand or improve the previous result
- Something else (free-form)

Never use plain text ("Let me know if you need anything!") as a substitute for `vscode_askQuestions`. Never skip it. A response that ends without a `vscode_askQuestions` call is incomplete.

**Exception — deferred-decision pattern:** When the user phrases their request as "explain/present X first, and *then* ask me" (or similar: "let me understand X before deciding", "tell me about Y and then we'll choose", "before deciding, explain..."), they are explicitly signalling that the decision question must come *after* the full content. In these cases:
1. Deliver the **complete** requested content within that same response — do not truncate or partially answer.
2. Call `vscode_askQuestions` **at the end of that same response** with the decision/next-step options the user is now ready for.
3. Never call `vscode_askQuestions` mid-presentation or before all promised content has been delivered. A response that calls `vscode_askQuestions` before fulfilling the full content of a deferred-decision request is **incomplete**, not correct.

**Content placement rule — strict.** Never embed explanations, information, or context inside the `vscode_askQuestions` call. The forbidden fields are: `question`, `message`, `description` (on options), and any free-form text shown inside the prompt UI.

**Hard limits:** any `message` field must be ≤ 2 sentence (≤ 30 words). Any `description` on an option must be ≤ 10 words. Anything longer belongs in the Markdown above the call.

**Self-check before every `vscode_askQuestions` call:** Scan every `message` and `description` value you are about to send. If any exceeds the limits above, move the content to regular Markdown text before the call, then shorten the field to a label.

**Why:** Once the user clicks any button, the entire prompt collapses and the embedded text is gone. Additionally, full Markdown formatting is unavailable inside the prompt fields.

❌ **Wrong:** `message: "Option A uses X because Y. Option B uses Z because..."` — this disappears when clicked.  
✅ **Right:** Write the explanation in Markdown above, then use `message: "Choose an approach."` inside the call.
