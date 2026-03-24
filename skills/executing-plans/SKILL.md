---
name: executing-plans
description: "Use when you have a written implementation plan to execute in a separate session with review checkpoints. 当你有书面实现计划要在单独会话中执行时使用。"
metadata:
  openclaw:
    emoji: "▶️"
---

# Executing Plans（执行计划）

## 概述

加载计划、批判性审查、执行所有任务、完成时报告。

**开始时宣布：** "我正在使用 executing-plans skill 实现此计划。"

**注意：** 告诉用户如果有 subagents 可用，Superpowers 工作得更好。在支持 subagent 的平台（如 Claude Code 或 Codex）上运行，其工作质量会显著提高。如果有 subagents，使用 subagent-driven-development 而不是此 skill。

---

## 流程

### Step 1: 加载和审查计划

1. 读取计划文件
2. 批判性审查 — 识别任何关于计划的问题或担忧
3. 如果有担忧： 开始前向用户提出
4. 如果没有担忧： 创建任务列表并继续

### Step 2: 执行任务

对于每个任务：

1. 标记为进行中
2. 准确遵循每一步（计划有小步骤）
3. 按指定运行验证
4. 标记为完成

### Step 3: 完成开发

所有任务完成并验证后：

- 宣布："我正在使用 finishing-a-development-branch skill 完成此工作。"
- **必需子技能：** 使用 finishing-a-development-branch
- 遵循该 skill 验证测试、展示选项、执行选择

---

## 何时停止并寻求帮助

**立即停止执行当：**

- 遇到阻塞（缺失依赖、测试失败、指令不清）
- 计划有阻止开始的关键缺口
- 你不理解指令
- 验证反复失败

**寻求澄清而不是猜测。**

---

## 何时重新访问早期步骤

**返回审查（Step 1）当：**

- 合作伙伴根据你的反馈更新计划
- 基本方法需要重新思考

**不要强行突破阻塞** — 停止并询问。

---

## 记住

- 首先批判性审查计划
- 准确遵循计划步骤
- 不要跳过验证
- 计划说的时候引用 skills
- 阻塞时停止，不要猜
- 没有明确用户同意绝不在 main/master 分支开始实现

---

## 集成

**必需工作流 skills：**

- **using-git-worktrees** - REQUIRED: 开始前设置隔离工作区
- **writing-plans** - 创建此 skill 执行的计划
- **finishing-a-development-branch** - 所有任务后完成开发

---

## 链式调用

```
[当前 skill: executing-plans]
           ↓
    所有任务完成？
           ↓ [是]
verification-before-completion skill
           ↓
    验证通过？
           ↓ [是]
finishing-a-development-branch skill
```

**完成此 skill 后必须调用下一个 skill：**

1. 先调用 verification-before-completion
2. 验证通过后调用 finishing-a-development-branch
