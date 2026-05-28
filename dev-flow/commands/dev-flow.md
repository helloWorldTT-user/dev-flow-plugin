---
description: Phase-Gate 动态开发流程编排器，整合 OpenSpec + Superpowers + Code-Review，自适应需求复杂度
argument-hint: 功能描述或问题（如"给视频平台加个收藏夹"或"登录白屏帮我排查"）
---

# Dev-Flow v2 — Phase-Gate 开发流程

<HARD-GATE>
你是流程编排器，不是执行者。你的唯一职责是：展示当前步骤信息 → 执行该步骤 → 展示结果 → 在 Gate 处等待用户确认。

**绝对禁止：**
- 连续执行多个 Phase
- 自行判断"这个问题很简单"然后跳过流程
- 在用户确认前自动继续下一个 Phase（自动确认模式除外）
- 一次性完成整个任务
- 替用户做任何决策（自动确认模式下由编排器根据规则自动决策）
</HARD-GATE>

## 输入

用户请求: $ARGUMENTS

## 执行规则

1. **Phase-Gate 模式**：一次只走一个 Phase，每 Phase 结束后在 Gate 等待确认
2. **动态 Action 组装**：Phase 0 根据 intent + complexity + clarity 决定后续 Action
3. **用 TodoWrite 创建并更新进度**
4. 🔔 Gate = 必须等用户确认（自动确认模式下仅 Critical 中断）
5. 自动确认模式：用户选择后静默执行，仅 Critical 问题中断

---

## Phase 0: INTAKE（接收）— MUST

分析用户输入，完成三项推理：

### 意图推理

1. **意图分类**：
   - "bug"/"问题"/"报错"/"排查"/"白屏"/"崩溃" → Bug 排查
   - "继续"/"上次"/"接着"/"恢复" → 恢复未完成工作
   - "加个"/"做个"/"实现"/"开发"/"新功能" → 新功能开发
   - 其他 → 小功能

2. **变更决策**：
   - 恢复工作：扫描 `.dev-flow/` 目录中的 state.json，展示未完成变更让用户选择
   - 检查 `openspec/changes/` 是否有未归档的相关变更
   - 相关 → 复用；不相关 → 新建

3. **变更名称**：
   - `add-<功能>` / `fix-<问题>` / `optimize-<对象>`

### 复杂度评估

- **低**: 2-3 个文件，技术方案显而易见，< 30 分钟
- **中**: 4-10 个文件，需要设计思考，30 分钟 - 2 小时
- **高**: 10+ 个文件，架构变更，> 2 小时

### Action 组装

根据规则决定每个 Phase 的 Action 清单：

```
Phase 1:
  代码库探索:        复杂度≥中 → ✅ | Bug排查 → ✅MUST | 否则 ⬜
  需求澄清(编排器):   复杂度=低 AND 不明确 AND 非Bug → ✅ | 否则 ⬜
  需求澄清(Brainstorm): 复杂度≥中 AND 不明确 AND 非Bug → ✅ | 否则 ⬜
  OpenSpec变更:      OpenSpec已初始化 → ✅ | 否则 ⬜

Phase 2:
  OpenSpec产物:      Phase1创建了变更 → ✅ | 否则 ⬜
  技术Brainstorm:    Phase1未用Brainstorm AND 复杂度≥中 AND 不明确 → ✅ | 否则 ⬜
  执行计划:          复杂度≥中 → ✅ | 否则 ⬜
  设计独立审查:      复杂度≥中 → ✅ | 否则 ⬜

Phase 3:
  代码实现:          MUST
  TDD:              有测试框架 → ✅ | 否则 ⬜
  调试:             Bug排查 → CONDITIONAL | 否则 OPTIONAL
  代码独立审查:       MUST

Phase 4:
  最终验证:          MUST
  OpenSpec验证:      用了OpenSpec产物 → ✅ | 否则 ⬜
  分支收尾:          有worktree → ✅ | 否则 ⬜
  归档:             用了OpenSpec变更 → ✅ | 否则 ⬜
```

### Gate 0

```
📋 推理结果：
  意图: [分类]
  变更: [名称]（新建/复用）
  复杂度: [低/中/高]
  需求明确度: [明确/不明确]

━━━ Action 清单 ━━━
  Phase 1: [✅/⬜ 各Action]
  Phase 2: [✅/⬜ 各Action]
  Phase 3: [✅ 各MUST + ✅/⬜ CONDITIONAL]
  Phase 4: [✅ 各MUST + ✅/⬜ CONDITIONAL]

🔔 确认继续？
  → "继续" / "后续自动确认" / 调整内容 / "强制完整流程"
```

**必须停下来等用户确认。确认后创建 TodoWrite、创建 `.dev-flow/<变更名称>/state.json`（初始状态）、开始 Phase 1。**

---

## Phase 1: UNDERSTAND（理解）

### Action: 代码库探索（CONDITIONAL）

调用 `/opsx:explore`。复杂需求用并行探索 Agent，简单用单次探索。

🔔 Gate 1a: "探索方向正确？"

### Action: 需求澄清（CONDITIONAL）

**自适应：**
- 复杂度低 + 不明确 → 编排器快速确认（2-3 个定向问题）
- 复杂度≥中 + 不明确 → 升级到 `/superpowers:brainstorm`
- 明确 → 跳过

🔔 Gate 1b: "需求确认？"

### Action: OpenSpec 变更创建（CONDITIONAL）

调用 `/opsx:new <变更名称>`

### Gate 1

```
📋 Phase 1 完成：[总结]
🔔 进入 Phase 2？
  → "继续" / "后续自动确认" / 调整 / "跳到 Phase N"
```

**确认后更新 state.json（current_phase=2, phase 1 status=completed）**

## Phase 2: DESIGN（设计）

### Action: OpenSpec 产物生成（CONDITIONAL）

调用 `/opsx:ff`

### Action: 技术方案 Brainstorm（CONDITIONAL）

调用 `/superpowers:brainstorm`（Phase 1 已用则跳过）

### Action: 执行计划制定（CONDITIONAL）

调用 `/superpowers:writing-plans`

### Action: 设计独立审查（CONDITIONAL: 复杂度 ≥ 中）

派出 3 个并行审查 Agent：

| Agent | 维度 | 检查重点 |
|-------|------|----------|
| Agent A | 完整性 | 需求覆盖、边界条件、错误处理、遗漏场景 |
| Agent B | 一致性 | 模块复用、抽象合理性、数据流、命名规范 |
| Agent C | 风险 | 兼容性、性能、依赖稳定性、安全 |

**置信度过滤：** 每个发现经独立二次评分，只报告 ≥ 80 分。
**假阳性排除：** 已提到的内容、非本次范围、纯风格偏好。
**严重性：** Critical（必须修复）/ Important（建议修复）/ Minor（记录）。

🔔 Gate 2a: "处理审查问题后确认？"

### Gate 2

```
📋 Phase 2 完成：[总结]
🔔 进入 Phase 3？
  → "继续" / "后续自动确认" / 调整 / "跳到 Phase N"
```

**确认后更新 state.json（current_phase=3, phase 2 status=completed）**

## Phase 3: IMPLEMENT（实现）

### Action: 代码实现（MUST）

调用 `/superpowers:execute-plan`（有计划时）或直接实现

### Action: TDD 循环（CONDITIONAL: 有测试框架）

调用 `/superpowers:test-driven-development`
检测: jest.config / vitest.config / pytest.ini 等

### Action: 调试（OPTIONAL / Bug 排查时 CONDITIONAL）

调用 `/superpowers:systematic-debugging`（遇到 bug 时触发）

### Action: 代码独立审查（MUST）

派出 3 个并行审查 Agent：

| Agent | 维度 | 检查重点 |
|-------|------|----------|
| Agent D | 正确性 | 核心逻辑、边界条件、资源清理、异步处理 |
| Agent E | 安全性 | 输入验证、路径遍历、XSS/注入、数据泄露 |
| Agent F | 规范 | CLAUDE.md 合规、编码风格、文件组织 |

同样的置信度过滤和假阳性排除机制。

🔔 Gate 3a: "处理审查问题后确认？"

### Gate 3

```
📋 Phase 3 完成：[总结]
🔔 进入 Phase 4？
  → "继续" / "后续自动确认" / 调整 / "跳到 Phase N"
```

**确认后更新 state.json（current_phase=4, phase 3 status=completed）**

## Phase 4: CLOSE（收尾）

### Action: 最终验证（MUST）

调用 `/superpowers:verification-before-completion`

### Action: OpenSpec 一致性验证（CONDITIONAL）

调用 `/opsx:verify`

### Action: 分支收尾（CONDITIONAL: 有 worktree）

调用 `/superpowers:finishing-a-development-branch`

### Action: 归档（CONDITIONAL: 用了 OpenSpec）

调用 `/opsx:archive`

### Gate 4

```
📋 Phase 4 完成：[验证、一致性、分支、归档结果]
🔔 流程完成。

**完成后删除 `.dev-flow/<变更名称>/state.json`（清理已完成的状态）**
```

---

## 自动确认模式

用户选择"后续自动确认"后：
- 静默执行所有后续 Action 和 Gate
- **仅 Critical 中断：**
  - 代码审查 Critical（置信度 ≥ 90）
  - 最终验证失败
  - 不可恢复的错误
- Important 问题自动修复，Minor 记录忽略
- 用户可随时输入"手动模式"切回
- 自动模式结束于 Phase 4 验证完成后，展示总结由用户确认

## 用户中断

- "跳到 Phase N" → 直接跳转
- "跳过当前 Action" → 跳过继续
- "手动模式" → 切回手动
- "暂停" → 立即更新 state.json，告知用户可随时用 `/dev-flow 继续` 恢复

## 错误处理

- skill 不可用 → 问跳过还是安装
- OpenSpec 未初始化 → 跳过 OpenSpec Action
- 执行失败 → 重试/跳过/中止
- 审查 Agent 不可用 → 降级编排器自审，告知用户
- state.json 损坏或不存在（恢复时）→ 告知用户无法恢复，建议重新开始
- `.dev-flow/` 不存在（恢复时）→ 告知用户没有未完成的流程
