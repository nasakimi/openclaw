---
name: writing-plans
description: "Use when you have a spec or requirements for a multi-step task, before touching code. 当你有规格或多步骤任务需求时使用，在碰代码之前。"
metadata:
  openclaw:
    emoji: "📋"
---

# Writing Plans（编写计划）

## 概述

编写全面的实现计划，假设工程师对我们的代码库零上下文和可疑品味。记录他们需要知道的一切：每个任务要碰哪些文件、代码、测试、可能需要检查的文档、如何测试。把整个计划作为小任务给他们。DRY。YAGNI。TDD。频繁提交。

假设他们是熟练的开发者，但几乎不了解我们的工具集或问题领域。假设他们不太知道好的测试设计。

**开始时宣布：** "我正在使用 writing-plans skill 创建实现计划。"

**上下文：** 这应该在专用的 worktree 中运行（由 brainstorming skill 创建）。

**计划保存到：** `docs/plans/YYYY-MM-DD-<feature-name>.md`

- （用户对计划位置的偏好覆盖此默认值）

---

## 范围检查

如果规格覆盖多个独立子系统，它应该在头脑风暴期间被分解为子项目规格。如果没有，建议将其分解为单独的计划 — 每个子系统一个。每个计划应该自己产生可工作的、可测试的软件。

---

## 文件结构

在定义任务之前，映射出将创建或修改哪些文件以及每个文件负责什么。这是分解决策被锁定的地方。

- 设计边界清晰和定义良好接口的单元。每个文件应该有一个明确责任。
- 你对能一次保持在上下文中的代码推理最好，文件专注时编辑更可靠。更喜欢更小、专注的文件而不是做太多的大文件。
- 一起变化的文件应该在一起。按责任拆分，不是技术层。
- 在现有代码库中，遵循已建立的模式。如果代码库使用大文件，不要单方面重构 — 但如果你修改的文件变得难以控制，在计划中包含拆分是合理的。

此结构告知任务分解。每个任务应该产生有意义的、独立讲得通的自包含更改。

---

## 小任务粒度

**每一步是一个动作（2-5 分钟）：**

- "写失败测试" - 步骤
- "运行确认失败" - 步骤
- "实现最小代码使测试通过" - 步骤
- "运行测试确认通过" - 步骤
- "提交" - 步骤

---

## 计划文档头部

**每个计划必须以此头部开始：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** 如果有 subagents，使用 subagent-driven-development；否则使用 executing-plans 实现此计划。步骤使用复选框（`- [ ]`）语法跟踪。

**Goal:** [描述构建什么的一句话]

**Architecture:** [关于方法的 2-3 句话]

**Tech Stack:** [关键技术/库]

---
```

---

## 任务结构

````markdown
### Task N: [Component Name]

**Files:**

- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

---

## 记住

- 总是确切的文件路径
- 计划中的完整代码（不是"添加验证"）
- 带预期输出的确切命令
- DRY、YAGNI、TDD、频繁提交

---

## 执行交接

保存计划后：

**"计划完成并保存到 `docs/plans/<filename>.md`。准备执行？"**

**执行路径取决于 harness 能力：**

**如果 harness 有 subagents（Claude Code 等）：**

- 【必须】调用 subagent-driven-development skill
- 每任务新鲜 subagent + 两阶段审查

**如果 harness 没有 subagents：**

- 【必须】调用 executing-plans skill
- 带审查检查点的批量执行

---

## 链式调用

```
[当前 skill: writing-plans]
           ↓
    用户确认执行？
           ↓ [是]
    ┌──────┴──────┐
    ↓             ↓
有 subagents?   无 subagents
    ↓             ↓
subagent-driven   executing-plans
development       skill
    skill
    ↓             ↓
    └──────┬──────┘
           ↓
verification-before-completion skill
           ↓
finishing-a-development-branch skill
```

**完成此 skill 后必须调用下一个 skill。** 不要停留在计划阶段。
