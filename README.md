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

### Option 1: From GitHub Marketplace (Recommended)

**Step 1:** Register the marketplace in `~/.claude/plugins/known_marketplaces.json`:

```json
"dev-flow-marketplace": {
  "source": {
    "source": "github",
    "repo": "helloWorldTT-user/dev-flow-plugin"
  },
  "installLocation": "~/.claude/plugins/marketplaces/dev-flow-marketplace",
  "lastUpdated": "2026-05-22T00:00:00.000Z"
}
```

**Step 2:** Register the plugin in `~/.claude/plugins/installed_plugins.json`, add inside `"plugins": {`:

```json
"dev-flow@dev-flow-marketplace": [
  {
    "scope": "user",
    "installPath": "~/.claude/plugins/marketplaces/dev-flow-marketplace/dev-flow",
    "version": "1.0.0",
    "installedAt": "2026-05-22T00:00:00.000Z",
    "lastUpdated": "2026-05-22T00:00:00.000Z"
  }
]
```

**Step 3:** Restart Claude Code, then verify:

```
/dev-flow test it out
```

### Option 2: Local Directory Install

```bash
# 1. Copy files
mkdir -p ~/.claude/local-marketplace
git clone https://github.com/helloWorldTT-user/dev-flow-plugin.git /tmp/dev-flow-plugin
cp -r /tmp/dev-flow-plugin/.claude-plugin /tmp/dev-flow-plugin/marketplace.json /tmp/dev-flow-plugin/dev-flow ~/.claude/local-marketplace/

# 2. Add to known_marketplaces.json
# 3. Add to installed_plugins.json (set installPath to local-marketplace/dev-flow)
# 4. Restart Claude Code
```

See [安装指南.md](./安装指南.md) for detailed step-by-step instructions.

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

1. Remove `dev-flow@dev-flow-marketplace` from `installed_plugins.json`
2. Remove `dev-flow-marketplace` from `known_marketplaces.json`
3. Delete the local files
4. Restart Claude Code

## License

MIT
