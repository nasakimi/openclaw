---
name: superpowers-workflow
description: '完整的开发工作流编排。当用户说"用superpowers做xxx"、"帮我实现xxx"、"开发xxx功能"时自动触发。会自动串联：brainstorming → writing-plans → test-driven-development → verification → finishing。'
metadata:
  openclaw:
    emoji: "⚡"
---

# Superpowers 完整工作流

## 概述

这是一个**编排 skill**，用于自动串联完整的开发流程。

**核心原则：一个请求，完整交付。**

当用户说"用 superpowers 做某事"时，自动触发整个链条，不需要用户手动指定每一步。

---

## 触发条件

**自动触发：**

- "用 superpowers 做xxx"
- "帮我实现xxx功能"
- "开发xxx"
- "构建xxx"
- "帮我完成xxx任务"

**不触发（应该用单个 skill）：**

- "帮我debug这个问题" → 用 systematic-debugging
- "帮我review代码" → 用 requesting-code-review
- "写个测试" → 用 test-driven-development

---

## 完整流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    SUPERPOWERS WORKFLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. BRAINSTORMING                                                │
│     └→ 理解需求、探索方案、设计架构                               │
│     └→ 输出：设计文档 (docs/specs/YYYY-MM-DD-<topic>-design.md)  │
│     └→ 【必须】完成后进入 Step 2                                  │
│                                                                  │
│  2. WRITING-PLANS                                                │
│     └→ 将设计转化为可执行计划                                     │
│     └→ 输出：实现计划 (docs/plans/YYYY-MM-DD-<feature>.md)       │
│     └→ 【必须】完成后询问是否执行                                 │
│                                                                  │
│  3. TEST-DRIVEN-DEVELOPMENT                                      │
│     └→ 每个任务：先写测试 → 看失败 → 写最小代码 → 通过            │
│     └→ 【必须】测试通过才能继续                                   │
│                                                                  │
│  4. VERIFICATION-BEFORE-COMPLETION                               │
│     └→ 运行验证命令确认一切正常                                   │
│     └→ 【必须】证据先于声明                                       │
│                                                                  │
│  5. FINISHING-A-DEVELOPMENT-BRANCH                               │
│     └→ 验证测试 → 展示选项 → 合并/PR/保持                        │
│     └→ 清理 worktree                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 铁律

```
WORKFLOW IS MANDATORY. DO NOT SKIP STEPS.
```

1. **必须按顺序执行** — 不能跳过任何步骤
2. **每步完成才能进入下一步** — 不完成不继续
3. **用户确认才能进入实现** — 设计批准后才开始写代码
4. **测试先行** — 任何代码都要先有失败的测试

---

## 执行协议

### Step 1: Brainstorming

**宣布：** "我正在使用 superpowers-workflow，首先进入 brainstorming 阶段。"

**调用：** `brainstorming` skill

**输出：** 设计文档

**完成条件：** 用户批准设计

**下一步：** 自动进入 writing-plans（不需要用户再次确认）

### Step 2: Writing Plans

**宣布：** "设计已批准，现在使用 writing-plans 创建实现计划。"

**调用：** `writing-plans` skill

**输出：** 实现计划文档

**完成条件：** 计划完成

**下一步：** 询问用户是否立即执行

### Step 3: Execute with TDD

**宣布：** "开始执行计划，使用 test-driven-development 方式实现每个任务。"

**调用：**

- 如果有 subagents：`subagent-driven-development`
- 否则：`executing-plans` + `test-driven-development`

**完成条件：** 所有任务完成，所有测试通过

**下一步：** 自动进入 verification

### Step 4: Verification

**宣布：** "实现完成，现在进行 verification-before-completion 验证。"

**调用：** `verification-before-completion` skill

**完成条件：** 所有验证命令通过

**下一步：** 自动进入 finishing

### Step 5: Finishing

**宣布：** "验证通过，使用 finishing-a-development-branch 完成工作。"

**调用：** `finishing-a-development-branch` skill

**输出：** 合并/PR/保持 选项

---

## 中断和恢复

### 可以中断的点

- **Step 1 后：** 用户可能想修改设计
- **Step 2 后：** 用户可能想调整计划
- **任意 Step：** 发现问题需要回到 Step 1

### 恢复方式

告诉用户当前在哪一步：

```
当前状态：Step 2 - Writing Plans
已完成：Brainstorming（设计文档在 docs/specs/...）
下一步：继续执行计划？还是修改计划？
```

---

## 与单独 Skills 的关系

| 单独使用 skill                   | 在 workflow 中的位置   |
| -------------------------------- | ---------------------- |
| `brainstorming`                  | Step 1                 |
| `writing-plans`                  | Step 2                 |
| `test-driven-development`        | Step 3（贯穿整个实现） |
| `systematic-debugging`           | Step 3 中遇到 bug 时   |
| `verification-before-completion` | Step 4                 |
| `finishing-a-development-branch` | Step 5                 |

**重要：** 如果用户只说"帮我debug"，不触发 workflow，只触发 `systematic-debugging`。

---

## 示例对话

```
用户：用 superpowers 帮我实现一个用户登录功能

你：我正在使用 superpowers-workflow，首先进入 brainstorming 阶段。

[执行 brainstorming skill]
[提问、探索、设计方案]

你：设计已完成。让我总结一下...

[展示设计]
[用户批准]

你：设计已批准，现在使用 writing-plans 创建实现计划。

[执行 writing-plans skill]
[创建详细计划]

你：计划已完成，保存在 docs/plans/2026-03-12-user-login.md
    是否立即开始执行？

用户：是的

你：开始执行计划，使用 test-driven-development 方式实现每个任务。

[执行计划，每个任务 TDD]
[所有测试通过]

你：实现完成，现在进行 verification-before-completion 验证。

[运行测试命令，确认通过]

你：验证通过，使用 finishing-a-development-branch 完成工作。

[展示选项：合并/PR/保持]
```

---

## 红旗警告

**绝不允许：**

- 跳过 brainstorming 直接写代码
- 跳过 TDD 先写实现
- 不验证就声称完成
- 不测试就提交

**如果发现问题：**

- 实现中发现设计问题 → 回到 Step 1 重新设计
- 发现 bug → 使用 systematic-debugging，修完后继续

---

## 快速参考

| Step | Skill                          | 必须输出    | 完成标志     |
| ---- | ------------------------------ | ----------- | ------------ |
| 1    | brainstorming                  | 设计文档    | 用户批准     |
| 2    | writing-plans                  | 实现计划    | 计划完成     |
| 3    | test-driven-development        | 代码 + 测试 | 所有测试通过 |
| 4    | verification-before-completion | 验证输出    | 命令通过     |
| 5    | finishing-a-development-branch | 合并/PR     | 清理完成     |
