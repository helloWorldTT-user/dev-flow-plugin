# Dev-Flow Plugin for Claude Code

Phase-Gate 动态开发流程编排器，整合 OpenSpec + Superpowers + Code-Review + Feature-Dev 四个工具的优点。

## v2.5 重大更新（流程鲁棒性升级）

修复用户反复遇到的三个痛点：

1. **流程不变量（FLOW-INVARIANT）** — 新增"消息分类协议"，无论用户如何操作（中途插入新需求、跑题提问、随意跳转），编排器都把用户保持在当前 Phase 边界内
   - 每次收到用户消息先分类（Gate 选项 / 流程内追问 / 流程外消息 / 元指令）
   - 流程外消息触发"强制兜底"：作为子任务 / 暂存 / 开新流程 三选一
   - 启动强制路径：扫描 `.dev-flow/` 检查未完成流程，必须显式选择恢复或新建
   - 移除无条件"跳到 Phase N"漏洞，跳转必须经过依赖校验

2. **技能协同（OpenSpec × Superpowers）** — 两套工具的产物强制交叉引用，不再是独立路径
   - `/opsx:ff` 生成的 design.md 必须引用 Superpowers design.md
   - `/superpowers:writing-plans` 输入校验：必须同时包含 design.md + OpenSpec specs
   - Phase 3 入口校验：检查交叉引用是否存在（向后兼容旧流程，警告不阻塞）
   - Brainstorm 职责分离：Phase 1 聚焦需求，Phase 2 聚焦技术方案，可并存

3. **收尾单一出口 Checklist** — Phase 4 重构为 10 步有序 Checklist，强制覆盖文档/提交/归档
   - 新增 Action 4.5 文档同步：检查 README/docs/CLAUDE.md 是否需要更新
   - 新增 Action 3.5 中间 git 提交：Phase 3 代码实现完成后固化状态
   - 新增 Action 4.8 最终 git 提交：Phase 4 收尾后提交文档和归档变更
   - 新增 Action 4.9 产物清单核对：11 项产物逐项打勾确认
   - 归档失败回滚机制：检测状态不一致，禁止静默继续

## 功能特点

- **流程不变量** — 消息分类协议 + 强制兜底，无论用户如何操作都不脱离流程边界（v2.5）
- **技能协同** — OpenSpec specs 与 Superpowers design.md 强制交叉引用（v2.5）
- **收尾 Checklist** — 10 步有序收尾，含文档同步、git 提交、产物清单核对（v2.5）
- **Phase-Gate 架构** — 5 个 Phase（INTAKE → UNDERSTAND → DESIGN → IMPLEMENT → CLOSE），每个 Phase 有独立的质量门控
- **动态 Action 组装** — 根据需求复杂度和意图自动组装 Action 清单，不固定步骤
- **双重独立审查** — 设计审查（实现前）+ 代码审查（实现后），多维度并行 + 置信度过滤
- **自适应需求澄清** — 简单需求快速确认，复杂需求升级到 Brainstorm
- **自动确认模式** — 用户可在任意 Gate 选择"后续自动确认"，编排器自动执行
- **特殊路径支持** — Bug 排查（偏重探索+调试）、恢复工作（从断点继续）
- **中断恢复** — 流程状态自动持久化到 `.dev-flow/<变更名>/state.json`，关机重启后可从断点继续，支持多变更并行
- **环境自动检测** — Phase 0 通过 shell 命令实际检测 OpenSpec、测试框架、worktree，不靠 LLM 猜测
- **运行时复杂度升级** — 执行中发现实际复杂度超预期时，主动提示用户升级流程
- **产物传递校验** — Phase 3 开始前校验 Phase 2 设计产物完整性 + 交叉引用（v2.5）
- **规范漂移检测** — Phase 4 对比设计文档与实际代码，发现遗漏、范围蔓延、方案偏离
- **脏工作树处理** — 恢复工作时检测未提交变更，确认归属后才能继续
- **上下文压缩恢复** — state.json 包含 recovery_context 摘要，压缩后快速恢复理解

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
/dev-flow 继续上次的收藏夹功能      # 从断点恢复
/dev-flow 恢复                      # 列出所有未完成变更
```

Dev-Flow 会自动：
1. 推理意图和复杂度
2. 动态组装 Action 清单
3. 逐 Phase 执行，每 Phase 在 Gate 等待确认

## 流程架构

```
Phase 0: INTAKE（接收）
  环境检测 → 意图推理 → 复杂度评估 → Action 组装 → Gate 0 用户确认

Phase 1: UNDERSTAND（理解）
  代码探索 → 需求澄清 → OpenSpec 变更 → Gate 1

Phase 2: DESIGN（设计）
  产物生成 → 技术方案 → 执行计划 → 设计独立审查 → Gate 2

Phase 3: IMPLEMENT（实现）
  产物校验 → 代码实现 → TDD → 调试 → 代码独立审查 → 中间 git 提交 → Gate 3

Phase 4: CLOSE（收尾 Checklist）
  最终验证 → 阳性对照 → 漂移检测 → 一致性验证 → 文档同步
  → 归档 → 最终 git 提交 → 分支收尾 → 产物清单核对 → Gate 4
```

## 流程步骤与 Skill 命令对照

```
Phase 0: INTAKE（接收）
  ① 环境检测           — shell 命令检测 OpenSpec/测试框架/worktree
  ② 意图推理           — 编排器内置（关键词匹配 + 复杂度评估）
  ③ Action 组装        — 编排器内置（动态决定后续步骤）
  🔔 Gate 0: 用户确认推理结果和 Action 清单

Phase 1: UNDERSTAND（理解）
  ④ 代码库探索         /opsx:explore                  [CONDITIONAL: 复杂度≥中 / Bug排查时MUST]
  ⑤ 需求澄清（简单）    — 编排器快速确认（2-3 个定向问题）[CONDITIONAL: 低复杂度+不明确]
  ⑤ 需求澄清（复杂）    /superpowers:brainstorm         [CONDITIONAL: 中高复杂度+不明确]
  ⑥ OpenSpec 变更创建   /opsx:new <名称>                [CONDITIONAL: OpenSpec 已初始化]
  🔔 Gate 1: 用户确认理解正确

Phase 2: DESIGN（设计）
  ⑦ OpenSpec 产物生成   /opsx:ff                        [CONDITIONAL: 使用了变更创建]
  ⑧ 技术方案 Brainstorm /superpowers:brainstorm          [CONDITIONAL: Phase1 未用 + 复杂度≥中]
  ⑨ 执行计划制定        /superpowers:writing-plans       [CONDITIONAL: 复杂度≥中]
  ⑩ 设计独立审查        — 3 并行 Agent（完整性/一致性/风险）[CONDITIONAL: 复杂度≥中]
  🔔 Gate 2a: 审查结果确认
  🔔 Gate 2: 用户确认设计通过

Phase 3: IMPLEMENT（实现）
  ⑪ 产物传递校验        — 检查设计/计划文档完整性 + 交叉引用  [自动执行]
  ⑫ 代码实现           /superpowers:execute-plan        [MUST]
  ⑬ TDD 循环           /superpowers:test-driven-development  [MUST when 复杂度≥中, CONDITIONAL when 复杂度=低]
  ⑭ 调试               /superpowers:systematic-debugging     [OPTIONAL / Bug排查时CONDITIONAL]
  ⑮ 代码独立审查        — 3 并行 Agent（正确性/安全性/规范） [MUST]
  ⑯ 中间 git 提交       — Phase 3 实现完成后固化代码状态    [MUST, v2.5]
  🔔 Gate 3a: 审查结果确认
  🔔 Gate 3: 用户确认实现通过

Phase 4: CLOSE（收尾 Checklist）
  ⑰ 最终验证           /superpowers:verification-before-completion  [MUST]
  ⑱ 阳性对照检查        — 核心路径至少 1 个已知正确测试通过   [有测试时 MUST]
  ⑲ 规范漂移检测        — 对比设计文档 vs 实际代码          [有设计文档时 MUST]
  ⑳ 一致性验证         /opsx:verify                     [CONDITIONAL: 使用了 OpenSpec 产物]
  ㉑ 文档同步          — README/docs/CLAUDE.md 按需更新   [MUST, v2.5]
  ㉒ 归档               /opsx:archive + 失败回滚           [CONDITIONAL: 使用了 OpenSpec 变更]
  ㉓ 最终 git 提交       — 提交文档和归档变更              [MUST, v2.5]
  ㉔ 分支收尾           /superpowers:finishing-a-development-branch [CONDITIONAL: 有 worktree]
  ㉕ 产物清单核对        — 11 项产物逐项打勾确认           [MUST, v2.5]
  🔔 Gate 4: 流程完成
```

> 标注说明：**MUST** = 始终执行 | **CONDITIONAL** = 满足条件时执行 | **OPTIONAL** = 按需触发
> 独立审查采用并行 Agent + 置信度评分（≥ 80 分报告）+ 假阳性过滤机制。

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

## 中断恢复

流程状态自动持久化到 `.dev-flow/<变更名>/state.json`，关机重启后可从断点继续。

- 支持多个变更并行追踪（每个变更独立状态文件）
- 恢复时列出所有未完成变更，用户显式选择要继续哪个（即使只有一个也展示选择）
- 从上次中断的 Phase + Action 继续，跳过已完成的步骤
- 不依赖 OpenSpec，以 `.dev-flow/` 为唯一真相源
- **脏工作树处理**：恢复前检测未提交 git 变更，确认归属后才能继续
- **上下文压缩恢复**：state.json 包含 `recovery_context` 摘要字段，压缩后快速恢复理解
- Phase 4 归档后自动清理状态文件

## 运行时优化

- **复杂度升级**：执行中发现实际复杂度超 Phase 0 评估时，暂停提示用户升级流程
- **产物传递校验**：Phase 3 开始前校验 Phase 2 设计文档和计划文档未被意外修改
- **规范漂移检测**：Phase 4 对比设计文档与实际代码，检测遗漏、范围蔓延和方案偏离

## 卸载

```bash
claude plugins uninstall dev-flow
claude plugins marketplace remove dev-flow-marketplace
```

## License

MIT
