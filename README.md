# Personal GitHub Copilot Agents, Skills & Customizations

A personal repository of GitHub Copilot customizations for VS Code: custom agents, skills, instruction files, and prompt files.

---

## Repository Structure

```
.copilot/
├── agents/               # Custom agents (.agent.md) — specialized AI personas
├── skills/               # Agent skills (SKILL.md) — reusable domain capabilities
├── instructions/         # Instruction files (.instructions.md) — always-on AI rules
└── prompts/              # Prompt files (.prompt.md) — reusable slash commands
```

### What's in each folder

| Folder | File type | VS Code concept |
|---|---|---|
| `agents/` | `*.agent.md` | Custom agents — persistent personas with tool restrictions and handoffs |
| `skills/` | `SKILL.md` (inside named subdirectory) | Agent skills — portable, multi-file capabilities per the [agentskills.io](https://agentskills.io) standard |
| `instructions/` | `*.instructions.md` | File-based instructions — always-on or conditionally applied coding rules |
| `prompts/` | `*.prompt.md` | Prompt files — reusable slash commands for common tasks |

---

## Setup: Two Cases

### Case 1 — Clone directly to `~/.copilot/` (Recommended, minimum configuration)

`~/.copilot/` is VS Code's native personal customization folder. When you clone here, **agents, skills, and instructions are detected automatically** — no VS Code settings needed for those three types.

#### Steps

```bash
# Windows (PowerShell)
git clone https://github.com/InaKara/agents-skills.git "$env:USERPROFILE\.copilot"

# Linux / macOS
git clone https://github.com/InaKara/agents-skills.git ~/.copilot
```

> **If `~/.copilot/` already exists** (VS Code may have created it), clone into a temp folder and move the contents manually, or do a `git init` + `git remote add` + `git pull` inside the existing folder.

#### After cloning — what works automatically

| Type | Auto-detected? | Reason |
|---|---|---|
| Skills (`skills/`) | Native + setting recommended | `~/.copilot/skills/` is a VS Code native location |
| Agents (`agents/`) | Native + setting recommended | `~/.copilot/agents/` is a VS Code native location |
| Instructions (`instructions/`) | Native + setting recommended | `~/.copilot/instructions/` is a VS Code native location |
| Prompts (`prompts/`) | Setting required | No native `~/.copilot/prompts/` path exists |

#### VS Code settings needed — for all four types

Even though `~/.copilot/skills/`, `~/.copilot/agents/`, and `~/.copilot/instructions/` are native VS Code locations, explicitly declaring them in `settings.json` is recommended — it ensures VS Code treats them as active, and offers them as creation targets when you run "New Agent" or "New Skill" from the UI.

Add all four to your `settings.json` (`Ctrl+Shift+P` → "Open User Settings (JSON)"):

```jsonc
// Windows — replace YOUR_USERNAME
"chat.agentFilesLocations": {
  "C:\\Users\\YOUR_USERNAME\\.copilot\\agents": true
},
"chat.agentSkillsLocations": {
  "C:\\Users\\YOUR_USERNAME\\.copilot\\skills": true
},
"chat.instructionsFilesLocations": {
  "C:\\Users\\YOUR_USERNAME\\.copilot\\instructions": true
},
"chat.promptFilesLocations": {
  "C:\\Users\\YOUR_USERNAME\\.copilot\\prompts": true
}

// Linux / macOS
"chat.agentFilesLocations": {
  "~/.copilot/agents": true
},
"chat.agentSkillsLocations": {
  "~/.copilot/skills": true
},
"chat.instructionsFilesLocations": {
  "~/.copilot/instructions": true
},
"chat.promptFilesLocations": {
  "~/.copilot/prompts": true
}
```

Replace `YOUR_USERNAME` with your actual Windows username.

#### Avoiding duplicates

VS Code checks **both** `~/.copilot/agents/` and its internal `AppData\Roaming\Code\User\prompts\` folder for agents and instructions. If you previously had agents or instructions in the `AppData` folder, you will see **duplicate entries** in the Copilot UI. Once you've verified everything works from the cloned repo, delete the originals from:
- Windows: `%APPDATA%\Code\User\prompts\`
- Linux: `~/.config/Code/User\prompts/`
- macOS: `~/Library/Application Support/Code/User/prompts/`

---

### Case 2 — Clone to an arbitrary folder

Use this if you cannot or do not want to clone to `~/.copilot/`. You'll need to tell VS Code where to find each type via settings.

#### Steps

```bash
git clone https://github.com/InaKara/agents-skills.git /path/to/your/folder
```

#### VS Code settings needed

Add all four of these to your `settings.json` (`Ctrl+Shift+P` → "Open User Settings (JSON)"), replacing the paths with wherever you cloned the repo:

```jsonc
// Windows example — replace C:\\path\\to\\your\\clone with your actual path
"chat.agentFilesLocations": {
  "C:\\path\\to\\your\\clone\\agents": true
},
"chat.agentSkillsLocations": {
  "C:\\path\\to\\your\\clone\\skills": true
},
"chat.instructionsFilesLocations": {
  "~/.copilot/instructions": true,         // keep the default
  "C:\\path\\to\\your\\clone\\instructions": true
},
"chat.promptFilesLocations": {
  "C:\\path\\to\\your\\clone\\prompts": true
}

// Linux / macOS example
"chat.agentFilesLocations": {
  "/path/to/your/clone/agents": true
},
"chat.agentSkillsLocations": {
  "/path/to/your/clone/skills": true
},
"chat.instructionsFilesLocations": {
  "~/.copilot/instructions": true,
  "/path/to/your/clone/instructions": true
},
"chat.promptFilesLocations": {
  "/path/to/your/clone/prompts": true
}
```

---

## Day-to-day usage: sync across machines

Use the built-in **`/sync-agents-skills`** skill to pull updates or commit and push changes — from any VS Code workspace.

```
/sync-agents-skills pull          → pull latest changes from GitHub to this machine
/sync-agents-skills push          → commit and push new/modified files to GitHub
/sync-agents-skills push add security-review agent   → push with a custom commit message
```

The skill always operates on `~/.copilot/` (regardless of which repo you currently have open) and verifies the git remote before running any commands. See [`skills/sync-agents-skills/SKILL.md`](./skills/sync-agents-skills/SKILL.md) for full details.

---

## VS Code Concepts Cheat Sheet

| Concept | When to use it |
|---|---|
| **Agent** (`.agent.md`) | You want a persistent AI persona with specific tool restrictions and optional handoffs to other agents |
| **Skill** (`SKILL.md`) | You want a reusable, portable capability that can include scripts and works across Copilot CLI, cloud agent, and VS Code |
| **Instructions** (`.instructions.md`) | You want always-on coding rules or conventions applied automatically to every chat request |
| **Prompt** (`.prompt.md`) | You want a reusable slash command for a specific task, invoked manually with `/` in chat |

### Native VS Code personal paths (no settings required)

| Path | Content |
|---|---|
| `~/.copilot/skills/` | Personal skills |
| `~/.copilot/agents/` | Personal custom agents |
| `~/.copilot/instructions/` | Personal instruction files |
| `~/.claude/skills/`, `~/.agents/skills/` | Alternative skill locations (Claude / agentskills.io standard) |

---

## Related Resources

- [VS Code Custom Agents docs](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [VS Code Agent Skills docs](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [VS Code Custom Instructions docs](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [VS Code Prompt Files docs](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [agentskills.io specification](https://agentskills.io/)
- [github/awesome-copilot — community skills, agents, prompts](https://github.com/github/awesome-copilot)
