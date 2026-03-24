---
name: dispatching-parallel-agents
description: "Use when facing 2+ independent tasks that can be worked on without shared state or sequential dependencies. 面对 2+ 个可以在无共享状态或顺序依赖的情况下处理的独立任务时使用。"
metadata:
  openclaw:
    emoji: "🔀"
---

# Dispatching Parallel Agents（派遣并行 Agents）

## 概述

当你有多个不相关的失败（不同测试文件、不同子系统、不同 bug），顺序调查浪费时间。每个调查是独立的，可以并行发生。

**核心原则：每个独立问题域派遣一个 agent。让它们同时工作。**

---

## 何时使用

**使用场景：**

- 3+ 测试文件以不同根因失败
- 多个子系统独立损坏
- 每个问题可以在没有其他上下文的情况下理解
- 调查之间无共享状态

**不使用场景：**

- 失败相关（修复一个可能修复其他）
- 需要理解完整系统状态
- Agents 会互相干扰

---

## 流程图

```
多个失败? → [是] → 它们独立? → [是] → 能并行? → [是] → 并行派遣
                      ↓ [否 - 相关]          ↓ [否 - 共享状态]
                 单 agent 调查所有       顺序 agents
```

---

## 模式

### 1. 识别独立域

按损坏的内容分组失败：

- 文件 A 测试：工具审批流程
- 文件 B 测试：批处理完成行为
- 文件 C 测试：中止功能

每个域是独立的 — 修复工具审批不影响中止测试。

### 2. 创建专注的 Agent 任务

每个 agent 获得：

- **具体范围：** 一个测试文件或子系统
- **清晰目标：** 让这些测试通过
- **约束：** 不改其他代码
- **预期输出：** 你发现和修复的摘要

### 3. 并行派遣

```typescript
// 在 Claude Code / AI 环境中
Task("Fix agent-tool-abort.test.ts failures");
Task("Fix batch-completion-behavior.test.ts failures");
Task("Fix tool-approval-race-conditions.test.ts failures");
// 三个都同时运行
```

### 4. 审查和集成

当 agents 返回时：

- 读取每个摘要
- 验证修复不冲突
- 运行完整测试套件
- 集成所有更改

---

## Agent 提示结构

好的 agent 提示：

1. **专注** — 一个清晰问题域
2. **自包含** — 理解问题所需的所有上下文
3. **对输出具体** — Agent 应该返回什么？

```markdown
Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:

1. "should abort tool with partial output capture" - expects 'interrupted at' in message
2. "should handle mixed completed and aborted tools" - fast tool aborted instead of completed
3. "should properly track pendingToolCount" - expects 3 results but gets 0

These are timing/race condition issues. Your task:

1. Read the test file and understand what each test verifies
2. Identify root cause - timing issues or actual bugs?
3. Fix by:
   - Replacing arbitrary timeouts with event-based waiting
   - Fixing bugs in abort implementation if found
   - Adjusting test expectations if testing changed behavior

Do NOT just increase timeouts - find the real issue.

Return: Summary of what you found and what you fixed.
```

---

## 常见错误

**❌ 太宽：** "修复所有测试" — agent 迷失
**✅ 具体：** "修复 agent-tool-abort.test.ts" — 专注范围

**❌ 无上下文：** "修复竞争条件" — agent 不知道在哪
**✅ 上下文：** 粘贴错误消息和测试名称

**❌ 无约束：** Agent 可能重构一切
**✅ 约束：** "不要改生产代码"或"只修复测试"

**❌ 模糊输出：** "修复它" — 你不知道改了什么
**✅ 具体：** "返回根因和更改的摘要"

---

## 何时不使用

**相关失败：** 修复一个可能修复其他 — 先一起调查
**需要完整上下文：** 理解需要看到整个系统
**探索性调试：** 你还不知道什么坏了
**共享状态：** Agents 会干扰（编辑相同文件、使用相同资源）

---

## 验证

Agents 返回后：

1. **审查每个摘要** — 理解改了什么
2. **检查冲突** — Agents 编辑了相同代码吗？
3. **运行完整套件** — 验证所有修复一起工作
4. **抽查** — Agents 可能犯系统性错误

---

## 真实影响

来自调试会话 (2025-10-03)：

- 3 个文件中的 6 个失败
- 并行派遣 3 个 agents
- 所有调查同时完成
- 所有修复成功集成
- Agent 更改之间零冲突
