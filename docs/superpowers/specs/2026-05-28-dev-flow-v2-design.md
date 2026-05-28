# Dev-Flow v2.0 设计文档

> 日期: 2026-05-28
> 状态: 待审核
> 范围: dev-flow-plugin 的架构重构

## 背景

Dev-Flow v1.0 是一个线性步骤串联器，将 OpenSpec 和 Superpowers 用固定步骤编号串联。三种模式（13步/8步/6步）的区别仅在于"跳过了哪些步骤"。

### v1.0 的核心问题

1. **缺少需求澄清环节** — 用户说"做个下载器"，直接进入代码探索，没有确认细节
2. **缺少设计独立审查** — 设计缺陷直接流入实现，修复成本高
3. **缺少代码独立审查** — 单次 code-review 无置信度过滤，假阳性多
4. **固定模式僵化** — 3 种模式无法覆盖所有场景（重构、探索、实验等）
5. **工具耦合过紧** — OpenSpec 的步骤即使项目没用 OpenSpec 也出现在流程中

## 设计目标

1. 用 Phase-Gate 结构替代固定步骤模式
2. 在 Phase 内动态组装 Action，按需触发
3. 增加设计独立审查和代码独立审查
4. 整合 OpenSpec + Superpowers + Code-Review + Feature-Dev 四个工具的优点
5. 自适应需求澄清（简单用编排器，复杂升级到 Brainstorm）

## 架构

### Phase-Gate 结构

```
Phase 0: INTAKE（接收）
  → Gate 0: 用户确认推理结果 + Action 清单

Phase 1: UNDERSTAND（理解）
  → Gate 1a: 探索方向确认（如有代码探索）
  → Gate 1b: Brainstorm/需求确认（如有）
  → Gate 1: Phase 完成确认

Phase 2: DESIGN（设计）
  → Gate 2a: 设计审查结果确认（如有设计审查）
  → Gate 2: Phase 完成确认

Phase 3: IMPLEMENT（实现）
  → Gate 3a: 代码审查结果确认
  → Gate 3: Phase 完成确认

Phase 4: CLOSE（收尾）
  → Gate 4: 流程完成确认
```

### Phase 内 Action 定义

#### Phase 0: INTAKE

| Action | 类型 | 说明 |
|--------|------|------|
| 意图推理 | MUST | 分析用户输入，分类意图 |
| 复杂度评估 | MUST | 评估需求复杂度（低/中/高） |
| Action 组装 | MUST | 根据意图+复杂度决定 Action 清单 |

**意图分类规则：**
- 包含"bug"/"问题"/"报错"/"排查"/"白屏"/"崩溃"等 → Bug 排查
- 包含"继续"/"上次"/"接着" → 恢复未完成工作
- 包含"加个"/"做个"/"实现"/"开发"/"新功能" → 新功能开发
- 其他小改动 → 小功能

**复杂度评估标准：**
- **低**: 单文件或 2-3 个文件的小改动，技术方案显而易见，预计 < 30 分钟
- **中**: 涉及 4-10 个文件，需要一定设计思考，预计 30 分钟 - 2 小时
- **高**: 涉及 10+ 个文件，需要深度设计思考或架构变更，预计 > 2 小时

**需求明确度判断标准：**
- **明确**: 用户描述了具体的功能行为和约束（如"加个深色模式开关"）
- **不明确**: 用户只给了大方向（如"做个下载器"），需要确认细节

### 特殊路径：Bug 排查

当意图推理为"Bug 排查"时，流程自动调整：
- Phase 1: 代码探索 MUST（定位问题是排查的核心），需求澄清通常不需要
- Phase 2: 跳过技术方案 Brainstorm，OpenSpec 产物生成聚焦于修复方案而非新功能设计
- Phase 3: 调试从 OPTIONAL 升级为 CONDITIONAL（Bug 排查场景下大概率需要调试）
- 整体偏重 Phase 1（探索定位）和 Phase 3（修复+验证）

### 特殊路径：恢复未完成工作

当意图推理为"恢复工作"时：
- 扫描 `openspec/changes/` 找到未归档的变更
- 读取变更的 tasks.md 确定上次进度
- 从上次中断的 Phase 继续，跳过已完成的 Action

#### Phase 1: UNDERSTAND

| Action | 类型 | 触发条件 |
|--------|------|----------|
| 代码库探索 | CONDITIONAL | 复杂度 ≥ 中（Bug 排查时为 MUST） |
| 需求澄清（编排器） | CONDITIONAL | 复杂度 = 低 AND 需求不明确 AND 非 Bug 排查 |
| 需求澄清（Brainstorm） | CONDITIONAL | 复杂度 ≥ 中 AND 需求不明确 AND 非 Bug 排查 |
| OpenSpec 变更创建 | CONDITIONAL | OpenSpec 已初始化 |

**代码探索方式：** 复杂需求使用 feature-dev 的并行探索模式（同时派出 2-3 个探索 Agent，各自负责不同维度）。简单需求使用单次 `/opsx:explore`。

**需求澄清自适应逻辑：**
- 需求明确 → 跳过
- 需求不明确 + 复杂度低 → 编排器快速确认（2-3 个定向问题）
- 需求不明确 + 复杂度 ≥ 中 → 升级到 `/superpowers:brainstorm`（合并需求澄清和技术方案探索）

**Brainstorm 路径的产出：**
- 设计文档: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
- 此产出同时满足需求澄清和技术方案探索两个目标

#### Phase 2: DESIGN

| Action | 类型 | 触发条件 |
|--------|------|----------|
| OpenSpec 产物生成 | CONDITIONAL | 使用了 OpenSpec 变更创建 |
| 技术方案 Brainstorm | CONDITIONAL | Phase 1 未用 Brainstorm AND 复杂度 ≥ 中 AND 技术方案不明确 |
| 执行计划制定 | CONDITIONAL | 复杂度 ≥ 中 |
| 设计独立审查 | CONDITIONAL | 复杂度 ≥ 中 |

**自适应跳过逻辑：**
- Phase 1 已用 Brainstorm → Phase 2 跳过技术方案 Brainstorm
- Phase 1 未用 Brainstorm → Phase 2 根据复杂度决定是否触发 Brainstorm

**执行计划产出：** `docs/superpowers/plans/YYYY-MM-DD-<topic>-plan.md`

**设计独立审查**见"独立审查"章节。

#### Phase 3: IMPLEMENT

| Action | 类型 | 触发条件 |
|--------|------|----------|
| 代码实现 | MUST | 始终执行 |
| TDD 循环 | CONDITIONAL | 项目存在可运行的测试框架 |
| 调试 | OPTIONAL | 实现中遇到 bug 时触发（Bug 排查时升级为 CONDITIONAL） |
| 代码独立审查 | MUST | 始终执行 |

**TDD 触发逻辑：**
- 项目有 `jest.config.*` / `vitest.config.*` / `pytest.ini` 等测试配置 → 有测试框架
- 无测试框架 → 跳过 TDD，但必须通过 verification-before-completion 做其他验证
- 有测试框架但测试跑不起来 → 先修复测试环境

**代码独立审查**见"独立审查"章节。

#### Phase 4: CLOSE

| Action | 类型 | 触发条件 |
|--------|------|----------|
| 最终验证 | MUST | 始终执行 |
| OpenSpec 一致性验证 | CONDITIONAL | 使用了 OpenSpec 产物生成 |
| 分支收尾 | CONDITIONAL | 检测到 git worktree |
| 归档 | CONDITIONAL | 使用了 OpenSpec 变更创建 |

### 独立审查

#### 设计独立审查（Phase 2）

**触发条件:** 复杂度 ≥ 中

**执行方式:** 派出 3 个并行 Agent，每个只负责一个维度

| Agent | 维度 | 输入 | 检查重点 |
|-------|------|------|----------|
| Agent A | 完整性 | 需求 + 设计文档 + OpenSpec 产物 | 每个需求有对应方案？边界条件？错误处理？遗漏场景？ |
| Agent B | 一致性 | 设计文档 + 代码库架构 | 复用已有模块？不必要抽象？数据流一致？命名规范？ |
| Agent C | 风险 | 设计文档 + 技术方案 + 代码库现状 | 兼容性？性能瓶颈？依赖稳定性？安全风险？ |

**置信度评分:** 每个发现经独立 Agent 二次评分，只报告 ≥ 80 分的问题。

**假阳性排除清单:**
- 设计中已经提到但审查 Agent 没注意到的
- 非本次需求的范围
- 纯编码风格偏好

**严重性分级:**
- Critical: 必须修复才能继续
- Important: 应该修复但可以商议
- Minor: 记录但不阻塞

#### 代码独立审查（Phase 3）

**触发条件:** MUST（所有场景）

**执行方式:** 派出 3 个并行 Agent

| Agent | 维度 | 输入 | 检查重点 |
|-------|------|------|----------|
| Agent D | 正确性 | 代码 diff + 设计文档 | 核心逻辑？边界条件？资源清理？异步处理？ |
| Agent E | 安全性 | 代码 diff | 输入验证？路径遍历？XSS/注入？数据泄露？ |
| Agent F | 规范 | 代码 diff + CLAUDE.md + 项目配置 | CLAUDE.md 合规？编码风格？文件组织？错误处理？ |

**同样的置信度评分、假阳性排除和严重性分级机制。**

### 工具介入点

#### OpenSpec

| Phase | Action | 命令 | 职责 |
|-------|--------|------|------|
| Phase 1 | 代码库探索 | `/opsx:explore` | 探索需求方向 |
| Phase 1 | 变更创建 | `/opsx:new <name>` | 创建变更目录 |
| Phase 2 | 产物生成 | `/opsx:ff` | 生成 proposal/specs/design/tasks |
| Phase 4 | 一致性验证 | `/opsx:verify` | 产物与实现一致性 |
| Phase 4 | 归档 | `/opsx:archive` | 归档变更 |

#### Superpowers

| Phase | Action | 命令 | 职责 |
|-------|--------|------|------|
| Phase 1 | Brainstorm | `/superpowers:brainstorm` | 需求+技术方案（升级路径） |
| Phase 2 | 技术方案 Brainstorm | `/superpowers:brainstorm` | 技术方案探索（独立路径） |
| Phase 2 | 执行计划 | `/superpowers:write-plan` | 制定实现计划 |
| Phase 3 | 代码实现 | `/superpowers:execute-plan` | 按计划实现 |
| Phase 3 | TDD | `/superpowers:test-driven-development` | Red→Green→Refactor |
| Phase 3 | 调试 | `/superpowers:systematic-debugging` | 系统化调试 |
| Phase 4 | 最终验证 | `/superpowers:verification-before-completion` | 全面验证 |
| Phase 4 | 分支收尾 | `/superpowers:finishing-a-development-branch` | 合并/PR/清理 |

#### Code-Review

介入 Phase 3 的代码独立审查，使用其并行 Agent + 置信度评分模式。

#### Feature-Dev（模式借鉴）

- Phase 1 的并行探索 Agent 模式（code-explorer）
- Phase 2/3 的并行审查维度多样性
- "Ask Early, Decide Late" 模式（先探索再提问）

### 交互规范

**Gate 确认采用问答形式：**

```
📋 Phase X 完成：
  [总结内容]

🔔 确认继续？
  → "继续" — 进入下一个 Phase
  → "后续自动确认" — 切换为自动模式，直到验证完成
  → 输入调整内容 — 编排器应用调整后重新确认
  → "跳到 Phase N" — 直接跳转
  → "强制完整流程" — 将所有 CONDITIONAL Action 设为 MUST
```

**自动确认模式：**

用户在任意 Gate 选择"后续自动确认"后，编排器进入自动模式：

- **默认行为：** 编排器自行选择最合适选项，静默执行后续所有 Action 和 Gate，不等待用户输入
- **Critical 中断：** 仅在以下情况暂停并通知用户：
  - 代码独立审查发现 Critical 级别问题（置信度 ≥ 90）
  - 最终验证失败（测试不通过、构建失败）
  - 执行遇到不可恢复的错误
- **自动模式下的决策逻辑：**
  - CONDITIONAL Action：编排器根据触发条件自动判断是否执行
  - 设计审查/代码审查的 Important 级别问题：编排器自动修复后继续，不中断
  - Minor 级别问题：记录但忽略
- **用户可随时恢复手动模式：** 输入"手动模式"即可切回
- **自动模式结束于 Phase 4 最终验证完成后** — 归档前展示完整流程总结，由用户确认收尾

### 最小流程保障

任何场景都至少包含：
1. Phase 0: 意图推理 + 复杂度评估 + 用户确认
2. Phase 3: 代码实现 + 代码独立审查
3. Phase 4: 最终验证

三层质量保障：
1. Phase 0 用户确认（防止理解偏差）
2. Phase 3 代码独立审查（防止代码缺陷）
3. Phase 4 最终验证（防止功能回归）

### 产出物清单

**持久化（提交到 git）：**
1. 设计文档: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
2. 执行计划: `docs/superpowers/plans/YYYY-MM-DD-<topic>-plan.md`
3. OpenSpec 变更产物: `openspec/changes/<name>/`（proposal/specs/design/tasks）
4. 源代码变更
5. 测试代码
6. Git 提交 + PR
7. OpenSpec 归档: `openspec/archive/`

**临时（仅在对话上下文中）：**
1. Action 组装报告
2. 代码探索报告
3. 设计审查报告
4. 代码审查报告
5. 验证报告
6. 一致性验证报告

## 错误处理

- skill/command 不可用 → 告诉用户，问是跳过还是安装
- OpenSpec 未初始化 → 跳过所有 OpenSpec 相关 Action，不影响流程
- 执行失败 → 展示错误，问用户：重试/跳过/中止
- 审查 Agent 不可用 → 降级为编排器自审（无置信度过滤），告知用户

## 与 v1.0 的兼容性

- v2.0 完全替代 v1.0 的 `dev-flow-driver.md` 和 `dev-flow.md`
- 用户使用方式不变：`/dev-flow <描述>`
- v1.0 的三种模式在 v2.0 中自动被动态组装覆盖：
  - 原"完整模式" ≈ 复杂度=高 + 所有 CONDITIONAL 触发
  - 原"排查模式" ≈ Bug 排查意图 + 中等复杂度路径
  - 原"快速模式" ≈ 复杂度=低 + 最小 Action 集合
