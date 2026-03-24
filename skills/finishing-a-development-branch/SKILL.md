---
name: finishing-a-development-branch
description: "Use when implementation is complete, all tests pass, and you need to decide how to integrate the work. 实现完成、所有测试通过时使用，决定如何集成工作。"
metadata:
  openclaw:
    emoji: "🏁"
---

# Finishing a Development Branch（完成开发分支）

## 概述

通过展示清晰选项和处理选择的工作流来指导开发工作的完成。

**核心原则：验证测试 → 展示选项 → 执行选择 → 清理。**

**开始时宣布：** "我正在使用 finishing-a-development-branch skill 完成此工作。"

---

## 流程

### Step 1: 验证测试

**在展示选项前，验证测试通过：**

```bash
# 运行项目的测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**

```
测试失败（<N> 失败）。完成前必须修复：

[展示失败]

测试通过前无法继续合并/PR。
```

停止。不要继续到 Step 2。

**如果测试通过：** 继续 Step 2。

### Step 2: 确定基础分支

```bash
# 尝试常见基础分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问："此分支从 main 分出 — 正确吗？"

### Step 3: 展示选项

精确展示这 4 个选项：

```
实现完成。你想做什么？

1. 本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保持分支原样（我稍后处理）
4. 丢弃此工作

哪个选项？
```

**不要添加解释** — 保持选项简洁。

### Step 4: 执行选择

#### 选项 1: 本地合并

```bash
# 切换到基础分支
git checkout <base-branch>

# 拉取最新
git pull

# 合并功能分支
git merge <feature-branch>

# 验证合并结果上的测试
<test command>

# 如果测试通过
git branch -d <feature-branch>
```

然后：清理 worktree（Step 5）

#### 选项 2: 推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>

# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

然后：清理 worktree（Step 5）

#### 选项 3: 保持原样

报告："保持分支 <name>。Worktree 保留在 <path>。"

**不要清理 worktree。**

#### 选项 4: 丢弃

**先确认：**

```
这将永久删除：
- 分支 <name>
- 所有提交：<commit-list>
- <path> 的 Worktree

输入 'discard' 确认。
```

等待确切确认。

如果确认：

```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后：清理 worktree（Step 5）

### Step 5: 清理 Worktree

**对于选项 1、2、4：**

检查是否在 worktree 中：

```bash
git worktree list | grep $(git branch --show-current)
```

如果是：

```bash
git worktree remove <worktree-path>
```

**对于选项 3：** 保留 worktree。

---

## 快速参考

| 选项        | 合并 | 推送 | 保留 Worktree | 清理分支 |
| ----------- | ---- | ---- | ------------- | -------- |
| 1. 本地合并 | ✓    | -    | -             | ✓        |
| 2. 创建 PR  | -    | ✓    | ✓             | -        |
| 3. 保持原样 | -    | -    | ✓             | -        |
| 4. 丢弃     | -    | -    | -             | ✓ (强制) |

---

## 常见错误

**跳过测试验证**

- **问题：** 合并坏代码，创建失败 PR
- **修复：** 提供选项前总是验证测试

**开放式问题**

- **问题：** "我接下来该做什么？" → 模糊
- **修复：** 精确展示 4 个结构化选项

**自动 worktree 清理**

- **问题：** 在可能需要时移除 worktree（选项 2、3）
- **修复：** 只为选项 1 和 4 清理

**丢弃无确认**

- **问题：** 意外删除工作
- **修复：** 需要输入"discard"确认

---

## 红旗警告

**绝不：**

- 带失败测试继续
- 不验证合并结果上测试就合并
- 无确认删除工作
- 无明确请求强制推送

**总是：**

- 提供选项前验证测试
- 精确展示 4 个选项
- 选项 4 获取输入确认
- 只为选项 1 & 4 清理 worktree
