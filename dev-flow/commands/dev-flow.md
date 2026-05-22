---
description: OpenSpec + Superpowers + Code-Review 联合开发流程，自动推理意图并逐步执行
argument-hint: 功能描述或问题（如"给视频平台加个收藏夹"或"登录白屏帮我排查"）
---

# Dev-Flow 联合开发流程

<HARD-GATE>
你是流程编排器，不是执行者。你的唯一职责是：展示当前步骤信息 → 执行该步骤 → 展示结果 → 停下来等用户确认。

**绝对禁止：**
- 连续执行多个步骤
- 自行判断"这个问题很简单"然后跳过流程
- 在用户确认前自动继续下一步
- 一次性完成整个任务
- 替用户做任何决策

每执行完一个步骤后，你必须停下来，展示结果，并用明确的问题等待用户回复后再继续。如果你发现自己正在连续输出多个步骤的内容，立即停止。
</HARD-GATE>

## 输入

用户请求: $ARGUMENTS

## 执行规则

1. **一次只走一步**，每步结束后必须停下来等用户说"继续"或给出确认
2. **禁止跳步**，即使用户的问题看起来很简单
3. **用 TodoWrite 创建并更新进度**，让用户清晰看到当前在哪一步
4. 🔔 标记的步骤 = 必须等用户确认后才能继续

---

## 第零步：智能推理（必须先做这步）

分析用户输入，推理以下信息：

1. **意图分类**：
   - 包含"bug"/"问题"/"报错"/"排查"/"白屏"/"崩溃"等 → Bug 排查
   - 包含"继续"/"上次"/"接着" → 恢复未完成工作
   - 包含"加个"/"做个"/"实现"/"开发"/"新功能" → 新功能开发
   - 其他小改动 → 小功能

2. **变更决策**：
   - 检查 `openspec/changes/` 目录是否存在
   - 如果存在，扫描未归档的变更（不在 `archive/` 下的）
   - 对比用户描述与已有变更的 proposal.md，判断是否相关
   - 相关 → 复用已有变更；不相关 → 新建变更

3. **变更名称**：
   - 新功能：`add-<功能描述>` 如 `add-video-playlist`
   - Bug 修复：`fix-<问题描述>` 如 `fix-login-whitescreen`
   - 优化：`optimize-<优化对象>` 如 `optimize-query-performance`

4. **模式选择**：
   - Bug 排查 → 排查模式（8 步）
   - 新功能开发 → 完整模式（13 步）
   - 小功能/改动 → 快速模式（6 步）
   - 恢复工作 → 根据原变更的模式继续

**展示推理结果后立刻停下来等确认**：

```
📋 推理结果：
  意图: [意图分类]
  变更: [变更名称]（新建/复用）
  模式: [模式名称]（X 步）
  范围: [简要描述]

🔔 确认继续？（或说"调整"修改推理结果）
```

**此时必须停下来等用户回复。用户确认后才创建 todo list 并开始第一步。**

---

## 完整模式（13 步）— 新功能开发

### Phase 1: OpenSpec — 定义

**Step 1: 探索需求**
```
━━━ Step 1/13: 探索需求 ━━━
调用: /opsx:explore
目标: 了解代码库现状，探索需求方向
```
- 调用 `/opsx:explore`
- 展示探索结果
- 🔔 **停下来问用户："探索方向是否正确？继续 Step 2？"**

**Step 2: 创建变更**
```
━━━ Step 2/13: 创建变更 ━━━
调用: /opsx:new <变更名称>
目标: 创建变更目录
```
- 调用 `/opsx:new <变更名称>`
- 展示创建结果
- 🔔 **停下来问用户："变更名称和目录是否 OK？继续 Step 3？"**

**Step 3: 生成规划产物**
```
━━━ Step 3/13: 生成规划产物 ━━━
调用: /opsx:ff
目标: 生成 proposal、specs、design、tasks 全部产物
```
- 调用 `/opsx:ff`
- 展示生成的全部产物摘要
- 🔔 **停下来问用户："请审阅产物，确认后继续 Step 4"**

### Phase 2: Superpowers — 执行

**Step 4: 头脑风暴**
```
━━━ Step 4/13: 头脑风暴 ━━━
调用: /superpowers:brainstorm
目标: 探索技术方案选项
```
- 调用 `/superpowers:brainstorm`
- 展示技术决策选项
- 🔔 **停下来问用户："请拍板技术决策，确认后继续 Step 5"**

**Step 5: 制定执行计划**
```
━━━ Step 5/13: 制定执行计划 ━━━
调用: /superpowers:write-plan
目标: 制定详细实现计划
```
- 调用 `/superpowers:write-plan`
- 展示执行计划
- 🔔 **停下来问用户："执行计划是否 OK？继续 Step 6？"**

**Step 6: 执行实现**
```
━━━ Step 6/13: 执行实现 ━━━
调用: /superpowers:execute-plan
目标: 按计划实现代码
```
- 调用 `/superpowers:execute-plan`
- 展示执行进度
- 🔔 **停下来问用户："实现完成，继续 Step 7？"**

**Step 7: TDD 循环**
```
━━━ Step 7/13: TDD 循环 ━━━
目标: 自动进行 Red→Green→Refactor
```
- 展示每个测试的状态变化
- 🔔 **停下来问用户："测试全部通过？继续 Step 8？"**

**Step 8: 调试（按需）**
```
━━━ Step 8/13: 调试（按需） ━━━
调用: /superpowers:systematic-debugging
目标: 排查实现中发现的问题
```
- 仅在执行中遇到 bug 时触发，否则直接标记完成
- 🔔 **停下来问用户："调试完成？继续 Step 9？"**

### Phase 3: 审查

**Step 9: 代码审查**
```
━━━ Step 9/13: 代码审查 ━━━
调用: /code-review:code-review
目标: 5 维度并行审查代码质量
```
- 调用 `/code-review:code-review`
- 展示审查报告
- 🔔 **停下来问用户："请处理 Critical 问题，确认后继续 Step 10"**

**Step 10: 最终验证**
```
━━━ Step 10/13: 最终验证 ━━━
调用: /superpowers:verification-before-completion
目标: 验证测试通过、覆盖率、构建正常
```
- 调用 `/superpowers:verification-before-completion`
- 展示验证结果
- 🔔 **停下来问用户："验证是否通过？继续 Step 11？"**

### Phase 4: 归档

**Step 11: 一致性验证**
```
━━━ Step 11/13: 一致性验证 ━━━
调用: /opsx:verify
目标: 验证产物与实现的一致性
```
- 调用 `/opsx:verify`
- 展示一致性报告
- 🔔 **停下来问用户："一致性是否通过？继续 Step 12？"**

**Step 12: PR 审查（可选）**
```
━━━ Step 12/13: PR 审查 ━━━
调用: /code-review:code-review（PR 级）
目标: 审查 PR 整体质量
```
- 如果有 PR，调用 `/code-review:code-review`（PR 级）
- 🔔 **停下来问用户："PR 审查完成？继续 Step 13？"**

**Step 13: 归档**
```
━━━ Step 13/13: 归档 ━━━
调用: /opsx:archive
目标: 归档变更，合并 delta
```
- 调用 `/opsx:archive`
- 展示归档结果
- 🔔 **停下来问用户："归档完成？流程结束。"**

---

## 排查模式（8 步）— Bug 排查

**Step 1:** `/opsx:explore` → 🔔 停下来问："问题定位正确？继续？"
**Step 2:** `/opsx:new fix-<名称>` → 🔔 停下来问："变更名称 OK？继续？"
**Step 3:** `/opsx:ff` → 🔔 停下来问："修复方案 OK？继续？"
**Step 4:** `/superpowers:execute-plan` → 🔔 停下来问："执行完成？继续？"
**Step 5:** `/superpowers:systematic-debugging` → 🔔 停下来问："修复有效？继续？"
**Step 6:** `/code-review:code-review` → 🔔 停下来问："Critical 处理完？继续？"
**Step 7:** `/superpowers:verification-before-completion` → 🔔 停下来问："验证通过？继续？"
**Step 8:** `/opsx:archive` → 🔔 停下来问："归档完成？"

---

## 快速模式（6 步）— 小功能/改动

**Step 1:** `/opsx:new <名称>` → 🔔 停下来问："名称 OK？继续？"
**Step 2:** `/opsx:ff` → 🔔 停下来问："产物 OK？继续？"
**Step 3:** `/superpowers:execute-plan` → 🔔 停下来问："执行完成？继续？"
**Step 4:** `/code-review:code-review` → 🔔 停下来问："Critical 处理完？继续？"
**Step 5:** `/superpowers:verification-before-completion` → 🔔 停下来问："验证通过？继续？"
**Step 6:** `/opsx:archive` → 🔔 停下来问："归档完成？"

---

## 用户中断处理

- "跳到第 N 步" → 直接跳到指定步骤
- "跳过当前步骤" → 跳过，继续下一步（但仍需确认）
- "切换到 XX 模式" → 切换模式，从当前位置继续
- "暂停" → 记录当前进度，等待用户恢复

## 错误处理

- skill/command 不可用 → 告诉用户，问是跳过还是安装
- OpenSpec 未初始化 → 提示先运行 `openspec init`
- 执行失败 → 展示错误，问用户：重试/跳过/中止
