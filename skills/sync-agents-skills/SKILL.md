---
name: sync-agents-skills
description: "Sync personal Copilot agents, skills, instructions, and prompts with the remote GitHub repo at https://github.com/InaKara/agents-skills.git. Use when you want to pull updates from another machine, or commit and push newly added/modified agents, skills, instructions, or prompts. IMPORTANT: this skill always operates on the ~/.copilot/ directory, regardless of which workspace is currently open."
argument-hint: "[pull | push [commit message]]"
user-invocable: true
---

# Sync Agents & Skills with GitHub

This skill syncs the personal Copilot customizations repo at `~/.copilot/` with the remote at `https://github.com/InaKara/agents-skills.git`.

> **CRITICAL**: All git operations in this skill MUST be performed inside the `~/.copilot/` directory — NOT in the user's currently open workspace. Always `cd` to `~/.copilot/` first and verify you are in the correct repo before running any git command.

---

## Step 1: Always start by navigating to the correct repo

```bash
# Windows (PowerShell)
Set-Location "$env:USERPROFILE\.copilot"

# Linux / macOS
cd ~/.copilot
```

Then verify you are in the right place:

```bash
git remote -v
# Expected output must include: https://github.com/InaKara/agents-skills.git
```

If the remote does not match, stop and do not proceed. The user is not in the correct repo.

---

## Step 2: Determine the user's intent

Ask the user (or infer from the argument) which operation to perform:

- **"pull"** (or "sync", "update", "get latest") → proceed to [Pull updates](#pull-pull-updates-from-github)
- **"push"** (or "commit", "save", "publish", "share") → proceed to [Commit and push](#push-commit-and-push-changes-to-github)

---

## Pull: Pull updates from GitHub

Use this to receive changes made on another machine.

```bash
# From inside ~/.copilot/
git status                   # show what, if anything, is locally modified
git pull origin main         # pull latest from GitHub
```

After pulling, inform the user which files were updated.

---

## Push: Commit and push changes to GitHub

Use this after adding or modifying agents, skills, instructions, or prompts.

```bash
# From inside ~/.copilot/
git status                   # show what has changed
git add .                    # stage all changes (agents/, skills/, instructions/, prompts/, README.md)
git status                   # confirm what is staged
```

Then commit with a descriptive message. If the user provided a commit message, use it. If not, generate a short one that summarizes the changes (e.g. "add: security-review agent" or "update: copilot-base instructions"):

```bash
git commit -m "<message>"
git push origin main
```

Confirm success and list what was pushed.

---

## Safety rules

- NEVER run any git command (add, commit, push, pull, reset) without first confirming `pwd` / `Get-Location` shows `~/.copilot/` and `git remote -v` shows `https://github.com/InaKara/agents-skills.git`
- NEVER push to the user's current workspace repo
- If the user asks to push "everything", stage with `git add .` (scoped to `~/.copilot/`) — do NOT use `git add` from a different working directory
- The `skills/motivator/` directory is in `.gitignore` and will not be committed — this is intentional
