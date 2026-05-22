# Dev-Flow Plugin for Claude Code

OpenSpec + Superpowers + Code-Review 联合开发流程编排器，提供结构化的分步开发工作流。

## 功能特点

- **完整模式（13 步）** — 新功能开发，覆盖全生命周期
- **排查模式（8 步）** — Bug 调查与修复
- **快速模式（6 步）** — 小功能和改动

Dev-Flow 会自动推理用户意图，选择合适的工作流模式。

## 前置条件

- 已安装 [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)（v1.0+）
- 推荐安装的依赖插件：
  - `openspec` — 结构化变更管理
  - `superpowers` — TDD、调试、执行工作流
  - `code-review` — 多维度代码审查

## 安装

```bash
# 1. 添加 marketplace
claude plugins marketplace add https://github.com/helloWorldTT-user/dev-flow-plugin

# 2. 安装插件
claude plugins install dev-flow

# 3. 在 Claude Code 中重载插件
/reload-plugins
```

## 使用

```
/dev-flow 给视频平台加个收藏夹功能
/dev-flow 帮我排查登录白屏的问题
/dev-flow 给设置页加个深色模式开关
```

Dev-Flow 会自动：
1. 推理意图（新功能 / Bug 修复 / 小改动）
2. 选择对应的工作流模式
3. 逐步引导，每步等待确认

## 卸载

```bash
claude plugins uninstall dev-flow
claude plugins marketplace remove dev-flow-marketplace
```

## 项目结构

```
dev-flow-plugin/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace 索引
├── dev-flow/
│   ├── .claude-plugin/
│   │   └── plugin.json           # 插件元数据
│   ├── agents/
│   │   └── dev-flow-driver.md    # Agent 定义
│   └── commands/
│       └── dev-flow.md           # /dev-flow 命令
└── README.md
```

## License

MIT
