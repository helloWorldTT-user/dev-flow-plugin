# Dev-Flow Plugin for Claude Code

Phase-Gate 动态开发流程编排器，整合 OpenSpec + Superpowers + Code-Review + Feature-Dev 四个工具的优点。

## 功能特点

- **Phase-Gate 架构** — 5 个 Phase（INTAKE → UNDERSTAND → DESIGN → IMPLEMENT → CLOSE），每个 Phase 有独立的质量门控
- **动态 Action 组装** — 根据需求复杂度和意图自动组装 Action 清单，不固定步骤
- **双重独立审查** — 设计审查（实现前）+ 代码审查（实现后），多维度并行 + 置信度过滤
- **自适应需求澄清** — 简单需求快速确认，复杂需求升级到 Brainstorm
- **自动确认模式** — 用户可在任意 Gate 选择"后续自动确认"，编排器自动执行
- **特殊路径支持** — Bug 排查（偏重探索+调试）、恢复工作（从断点继续）

Dev-Flow 会自动推理用户意图、评估复杂度，动态组装最合适的流程。

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
1. 推理意图和复杂度
2. 动态组装 Action 清单
3. 逐 Phase 执行，每 Phase 在 Gate 等待确认

## 流程架构

```
Phase 0: INTAKE（接收）
  意图推理 → 复杂度评估 → Action 组装 → Gate 0 用户确认

Phase 1: UNDERSTAND（理解）
  代码探索 → 需求澄清 → OpenSpec 变更 → Gate 1

Phase 2: DESIGN（设计）
  产物生成 → 技术方案 → 执行计划 → 设计独立审查 → Gate 2

Phase 3: IMPLEMENT（实现）
  代码实现 → TDD → 调试 → 代码独立审查 → Gate 3

Phase 4: CLOSE（收尾）
  最终验证 → 一致性验证 → 分支收尾 → 归档 → Gate 4
```

## 项目结构

```
dev-flow-plugin/
├── .claude-plugin/
│   └── marketplace.json
├── dev-flow/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── agents/
│   │   └── dev-flow-driver.md
│   └── commands/
│       └── dev-flow.md
├── docs/superpowers/
│   ├── specs/
│   │   └── 2026-05-28-dev-flow-v2-design.md
│   └── plans/
│       └── 2026-05-28-dev-flow-v2.md
└── README.md
```

## 卸载

```bash
claude plugins uninstall dev-flow
claude plugins marketplace remove dev-flow-marketplace
```

## License

MIT
