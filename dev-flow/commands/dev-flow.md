---
description: Phase-Gate 动态开发流程编排器，整合 OpenSpec + Superpowers + Code-Review，自适应需求复杂度
argument-hint: 功能描述或问题（如"给视频平台加个收藏夹"或"登录白屏帮我排查"）
---

# Dev-Flow v2 — Phase-Gate 开发流程

<HARD-GATE>
你是流程编排器，不是执行者。你的唯一职责是：展示当前步骤信息 → 执行该步骤 → 展示结果 → 在 Gate 处使用 AskUserQuestion 工具等待用户选择。

**绝对禁止：**
- 连续执行多个 Phase
- 自行判断"这个问题很简单"然后跳过流程
- 在用户确认前自动继续下一个 Phase（自动确认模式除外）
- 一次性完成整个任务
- 替用户做任何决策（自动确认模式下由编排器根据规则自动决策）
- **用文字提示让用户手动输入确认**（如"继续？"、"确认？"、"输入继续"）— 所有 Gate 必须用 AskUserQuestion 工具提供可点击的选项
</HARD-GATE>

## 输入

用户请求: $ARGUMENTS

## 执行规则

1. **Phase-Gate 模式**：一次只走一个 Phase，每 Phase 结束后在 Gate 用 AskUserQuestion 等待确认
2. **动态 Action 组装**：Phase 0 根据 intent + complexity + clarity 决定后续 Action
3. **用 TodoWrite 创建并更新进度**
4. 🔔 Gate = 必须用 AskUserQuestion 工具让用户选择确认（自动确认模式下仅 Critical 中断）
5. 自动确认模式：用户选择后静默执行，仅 Critical 问题中断
6. **步骤开始前打印 skill 名称**：每个 Action 开始执行前，必须先输出一行格式为 `▶ 调用: /skill-name — 简要说明` 的信息，让用户清楚当前正在使用哪个 skill。如果是编排器内置 Action（如意图推理、需求澄清快速确认），则输出 `▶ 执行: Action名称 — 简要说明`
7. **运行时复杂度升级**：如果执行中发现实际复杂度超过 Phase 0 的评估（如预计改 2 个文件实际改了 6 个），编排器必须暂停，用 AskUserQuestion 提示用户升级流程，将被跳过的 CONDITIONAL Action 重新激活
8. **脏工作树处理**：恢复工作时如检测到未提交的 git 变更，先确认变更归属后再恢复
9. **产物传递校验**：Phase 3 开始前校验 Phase 2 产物文件未被意外修改
10. **规范漂移检测**：Phase 4 验证时检测设计与实现的偏差

---

## Phase 0: INTAKE（接收）— MUST

### 环境检测（必须先执行，不能靠猜测）

在意图推理之前，先执行实际检测：
1. **OpenSpec**：运行 `ls openspec/` 确认目录存在 → 存在则所有 OpenSpec Action 可用
2. **测试框架**：运行 `ls jest.config.* vitest.config.* pytest.ini pyproject.toml 2>/dev/null; cat package.json 2>/dev/null | grep -q '"test"'` → 存在则 `test_framework_detected = true`（用于测试夹具条件判断）
3. **Worktree**：运行 `git worktree list` → 有活跃 worktree 则分支收尾可用

### 意图推理

1. **意图分类**：
   - "bug"/"问题"/"报错"/"排查"/"白屏"/"崩溃" → Bug 排查
   - "继续"/"上次"/"接着"/"恢复" → 恢复未完成工作
   - "加个"/"做个"/"实现"/"开发"/"新功能" → 新功能开发
   - 其他 → 小功能

2. **变更决策**：
   - 恢复工作：扫描 `.dev-flow/` 目录中的 state.json，展示未完成变更让用户选择（即使只有一个也必须展示并等待确认）。恢复前检测 `git status`，如有未提交变更先让用户确认归属
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
  TDD:              MUST（始终执行，TDD 是强制规范）
  测试夹具:          有测试框架 AND 复杂度≥中 → ✅ | 否则 ⬜
  调试:             Bug排查 → CONDITIONAL | 否则 OPTIONAL
  代码独立审查:       MUST

Phase 4:
  最终验证:          MUST
  阳性对照检查:      推荐（非阻塞）→ ✅ | 无测试时 ⬜
  OpenSpec验证:      用了OpenSpec产物 → ✅ | 否则 ⬜
  分支收尾:          有worktree → ✅ | 否则 ⬜
  归档:             用了OpenSpec变更 → ✅ | 否则 ⬜
```

### Gate 0

展示推理结果和 Action 清单后，**必须使用 AskUserQuestion 工具**（不能用文字提示让用户手动输入）：

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
```

**AskUserQuestion 调用：**
```json
{
  "questions": [{
    "question": "确认推理结果和 Action 清单，如何继续？",
    "header": "Gate 0",
    "multiSelect": false,
    "options": [
      { "label": "继续", "description": "确认推理结果，进入 Phase 1" },
      { "label": "后续自动确认", "description": "确认后静默执行所有后续 Phase，仅 Critical 问题中断" },
      { "label": "强制完整流程", "description": "将所有 CONDITIONAL Action 设为 MUST" }
    ]
  }]
}
```

用户选择"Other"输入内容时视为调整请求。确认后创建 TodoWrite、创建 `.dev-flow/<变更名称>/state.json`（初始状态）、开始 Phase 1。

---

## Phase 1: UNDERSTAND（理解）

### Action: 代码库探索（CONDITIONAL）

调用 `/opsx:explore`。复杂需求用并行探索 Agent，简单用单次探索。

**Gate 1a:** 探索完成后使用 AskUserQuestion：
```json
{
  "questions": [{
    "question": "探索方向正确？",
    "header": "Gate 1a",
    "multiSelect": false,
    "options": [
      { "label": "正确，继续", "description": "确认探索方向，进入下一步" },
      { "label": "调整方向", "description": "指定新的探索方向" }
    ]
  }]
}
```

### Action: 需求澄清（CONDITIONAL）

**自适应：**
- 复杂度低 + 不明确 → 编排器快速确认（2-3 个定向问题）
- 复杂度≥中 + 不明确 → 升级到 `/superpowers:brainstorm`
- 明确 → 跳过

**Gate 1b:** 需求澄清完成后使用 AskUserQuestion：
```json
{
  "questions": [{
    "question": "需求确认？",
    "header": "Gate 1b",
    "multiSelect": false,
    "options": [
      { "label": "确认，继续", "description": "需求已明确，进入下一步" },
      { "label": "还需要讨论", "description": "继续细化需求" }
    ]
  }]
}
```

### Action: OpenSpec 变更创建（CONDITIONAL）

调用 `/opsx:new <变更名称>`

### Gate 1

展示 Phase 1 总结后，**必须使用 AskUserQuestion**：
```json
{
  "questions": [{
    "question": "Phase 1 (UNDERSTAND) 完成，如何继续？",
    "header": "Gate 1",
    "multiSelect": false,
    "options": [
      { "label": "继续", "description": "进入 Phase 2 (DESIGN)" },
      { "label": "后续自动确认", "description": "静默执行后续所有 Phase，仅 Critical 中断" },
      { "label": "强制完整流程", "description": "将所有 CONDITIONAL Action 设为 MUST" }
    ]
  }]
}
```

用户选择"Other"输入"跳到 Phase N"可直接跳转。确认后更新 state.json（current_phase=2, phase 1 status=completed）

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

**Gate 2a:** 设计审查完成后使用 AskUserQuestion：
```json
{
  "questions": [{
    "question": "设计审查完成，是否有需要处理的问题？",
    "header": "Gate 2a",
    "multiSelect": false,
    "options": [
      { "label": "没有问题，继续", "description": "审查通过，进入 Phase 2 确认" },
      { "label": "有问题需修复", "description": "修复审查发现的问题后重新确认" }
    ]
  }]
}
```

### Gate 2

展示 Phase 2 总结后，**必须使用 AskUserQuestion**：
```json
{
  "questions": [{
    "question": "Phase 2 (DESIGN) 完成，如何继续？",
    "header": "Gate 2",
    "multiSelect": false,
    "options": [
      { "label": "继续", "description": "进入 Phase 3 (IMPLEMENT)" },
      { "label": "后续自动确认", "description": "静默执行后续所有 Phase，仅 Critical 中断" },
      { "label": "强制完整流程", "description": "将所有 CONDITIONAL Action 设为 MUST" }
    ]
  }]
}
```

用户选择"Other"输入"跳到 Phase N"可直接跳转。确认后更新 state.json（current_phase=3, phase 2 status=completed）

## Phase 3: IMPLEMENT（实现）

### 产物传递校验（Phase 3 开始前自动执行）

检查 Phase 2 产物完整性：设计文档和计划文档是否存在且非空。如产物被删除或清空 → 告知用户，建议回到 Phase 2 重新生成。校验通过 → 输出 `✅ 产物校验通过`，继续。

### Action: 代码实现（MUST）

调用 `/superpowers:execute-plan`（有计划时）或直接实现

### Action: TDD 循环（MUST — 始终执行）

调用 `/superpowers:test-driven-development`
TDD 是强制开发规范，无论项目是否有测试框架都必须执行。如项目无测试框架，先初始化测试基础设施再开始 TDD。

**测试夹具指导（CONDITIONAL: 有测试框架 AND 复杂度 ≥ 中）**：有测试框架且复杂度 ≥ 中时，在 TDD 开始前先创建可复用的测试夹具（mock、stub、factory），确保测试隔离且不重复。简单任务不需要。

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

**Gate 3a:** 代码审查完成后使用 AskUserQuestion：
```json
{
  "questions": [{
    "question": "代码审查完成，是否有需要处理的问题？",
    "header": "Gate 3a",
    "multiSelect": false,
    "options": [
      { "label": "没有问题，继续", "description": "审查通过，进入 Phase 3 确认" },
      { "label": "有问题需修复", "description": "修复审查发现的问题后重新确认" }
    ]
  }]
}
```

### Gate 3

展示 Phase 3 总结后，**必须使用 AskUserQuestion**：
```json
{
  "questions": [{
    "question": "Phase 3 (IMPLEMENT) 完成，如何继续？",
    "header": "Gate 3",
    "multiSelect": false,
    "options": [
      { "label": "继续", "description": "进入 Phase 4 (CLOSE)" },
      { "label": "后续自动确认", "description": "静默执行后续所有 Phase，仅 Critical 中断" },
      { "label": "强制完整流程", "description": "将所有 CONDITIONAL Action 设为 MUST" }
    ]
  }]
}
```

用户选择"Other"输入"跳到 Phase N"可直接跳转。确认后更新 state.json（current_phase=4, phase 3 status=completed）

## Phase 4: CLOSE（收尾）

### Action: 最终验证（MUST）

调用 `/superpowers:verification-before-completion`

**阳性对照检查（推荐，非阻塞）**：验证核心功能路径至少有 1 个已知正确的测试用例通过。如果没有阳性对照用例，提示用户但不阻塞流程。

**规范漂移检测（有设计文档时执行）**：对比设计文档描述的功能点与实际代码实现，检测遗漏、范围蔓延和方案偏离。检测到漂移时列出具体项，用 AskUserQuestion 让用户选择处理方式。

### Action: OpenSpec 一致性验证（CONDITIONAL）

调用 `/opsx:verify`

### Action: 分支收尾（CONDITIONAL: 有 worktree）

调用 `/superpowers:finishing-a-development-branch`

### Action: 归档（CONDITIONAL: 用了 OpenSpec）

**归档三步流程：前置校验 → 执行归档 → 后置同步**

1. **前置校验**：确认 `openspec/changes/<name>/` 目录存在且产物完整（至少有 proposal）。校验失败时询问用户：跳过归档 / 回到 Phase 2 重新生成
2. **执行归档**：调用 `/opsx:archive`
3. **后置同步**：确认归档成功 → 更新 state.json（`artifacts.openspec_archive_path` 和 `openspec_archived_at`）。归档失败时询问用户：重试 / 跳过 / 中止

### Gate 4

展示 Phase 4 结果后，**必须使用 AskUserQuestion**：
```json
{
  "questions": [{
    "question": "Phase 4 (CLOSE) 完成，所有验证通过。确认收尾？",
    "header": "Gate 4",
    "multiSelect": false,
    "options": [
      { "label": "确认完成", "description": "流程结束，清理状态文件" },
      { "label": "还有问题", "description": "需要回退或修复问题" }
    ]
  }]
}
```

确认后删除 `.dev-flow/<变更名称>/state.json`（清理已完成的状态）。

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
