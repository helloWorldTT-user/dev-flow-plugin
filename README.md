# Dev-Flow Plugin for Claude Code

A structured development workflow orchestrator that combines **OpenSpec + Superpowers + Code-Review** into a step-by-step guided process.

## Features

- **Full Mode (13 steps)** — New feature development with complete lifecycle
- **Debug Mode (8 steps)** — Bug investigation and fix workflow
- **Quick Mode (6 steps)** — Small features and minor changes

Dev-Flow automatically infers your intent from the description and selects the appropriate workflow mode.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) v1.0+
- Recommended dependency plugins:
  - `openspec` — for structured change management
  - `superpowers` — for TDD, debugging, and execution workflows
  - `code-review` — for multi-dimensional code review

## Installation

### Option 1: Local Marketplace (Recommended)

```bash
# 1. Create directory and copy files
mkdir -p ~/.claude/local-marketplace
cp -r .claude-plugin marketplace.json dev-flow ~/.claude/local-marketplace/

# 2. Register marketplace in ~/.claude/plugins/known_marketplaces.json
# Add this entry (replace <username> with your actual username):
```

```json
"dev-flow-marketplace": {
  "source": {
    "source": "directory",
    "path": "/Users/<username>/.claude/local-marketplace"
  },
  "installLocation": "/Users/<username>/.claude/local-marketplace",
  "lastUpdated": "2026-05-22T00:00:00.000Z"
}
```

```bash
# 3. Restart Claude Code, then install:
/install-plugin dev-flow

# 4. Verify (restart session first):
/dev-flow test it out
```

### Option 2: Direct Path

```
/install-plugin /path/to/dev-flow
```

### Option 3: From GitHub

After pushing to GitHub, add to `known_marketplaces.json`:

```json
"dev-flow-marketplace": {
  "source": {
    "source": "github",
    "repo": "<your-github-username>/dev-flow-plugin"
  },
  "installLocation": "~/.claude/plugins/marketplaces/dev-flow-marketplace",
  "lastUpdated": "2026-05-22T00:00:00.000Z"
}
```

Then restart Claude Code and run `/install-plugin dev-flow`.

## Usage

```
/dev-flow add a favorites feature to the video platform
/dev-flow investigate the login page white screen issue
/dev-flow add a dark mode toggle to settings
```

Dev-Flow will automatically:
1. Infer your intent (new feature / bug fix / small change)
2. Select the appropriate workflow mode
3. Guide you through each step with confirmation prompts

## Project Structure

```
dev-flow-plugin/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace index
├── marketplace.json              # Compatibility index
├── dev-flow/
│   ├── .claude-plugin/
│   │   └── plugin.json           # Plugin metadata
│   ├── agents/
│   │   └── dev-flow-driver.md    # Agent definition
│   └── commands/
│       └── dev-flow.md           # /dev-flow command
├── README.md                     # This file
└── 安装指南.md                    # Chinese installation guide
```

## Uninstall

```
/uninstall-plugin dev-flow
```

Then remove the `dev-flow-marketplace` entry from `known_marketplaces.json`.

## License

MIT
