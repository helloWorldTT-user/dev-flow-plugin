---
description: Phase-Gate 动态开发流程编排器（v2.5 — 流程不变量 + 技能协同 + 收尾 Checklist）
argument-hint: 功能描述或问题（如"给视频平台加个收藏夹"或"登录白屏帮我排查"）
---

# Dev-Flow v2.5 入口

本命令通过 `dev-flow-driver` agent 执行。完整契约见 `agents/dev-flow-driver.md`。

## v2.5 核心特性

- **流程不变量（FLOW-INVARIANT）**：消息分类协议 + 类型 C 强制兜底，无论用户如何操作都不脱离流程边界
- **技能协同**：OpenSpec specs/ 与 Superpowers design.md 强制交叉引用
- **收尾单一出口**：10 步有序 Checklist，含文档同步、中间 commit、最终 commit、产物清单核对

## 五个 Phase

| Phase | 名称 | 关键产出 |
|-------|------|----------|
| 0 | INTAKE（接收） | 意图/复杂度/Action 清单 + state.json |
| 1 | UNDERSTAND（理解） | 代码探索 + 需求澄清 + OpenSpec 变更 |
| 2 | DESIGN（设计） | OpenSpec 产物 + design.md + 执行计划 |
| 3 | IMPLEMENT（实现） | 代码 + 测试 + 审查 + 中间 commit |
| 4 | CLOSE（收尾） | 验证 + 文档同步 + 归档 + 最终 commit + 产物清单 |

## 触发示例

- 新功能：`/dev-flow:dev-flow 给视频平台加个收藏夹功能`
- Bug 排查：`/dev-flow:dev-flow 登录页面偶尔白屏帮我排查`
- 恢复工作：`/dev-flow:dev-flow 继续上次没做完的收藏夹功能`
- 小改动：`/dev-flow:dev-flow 给设置页加个深色模式开关`

agent 启动后会自动扫描 `.dev-flow/*/state.json` 检查是否有未完成流程。
