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

```bash
# 1. Add this repo as a marketplace
claude plugins marketplace add https://github.com/helloWorldTT-user/dev-flow-plugin

# 2. Install the plugin
claude plugins install dev-flow

# 3. Reload plugins in Claude Code (or restart session)
/reload-plugins
```

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

## Uninstall

```bash
claude plugins uninstall dev-flow
claude plugins marketplace remove dev-flow-marketplace
```

## Project Structure

```
dev-flow-plugin/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace index
├── dev-flow/
│   ├── .claude-plugin/
│   │   └── plugin.json           # Plugin metadata
│   ├── agents/
│   │   └── dev-flow-driver.md    # Agent definition
│   └── commands/
│       └── dev-flow.md           # /dev-flow command
├── README.md
└── 安装指南.md                    # Chinese installation guide
```

## License

MIT
