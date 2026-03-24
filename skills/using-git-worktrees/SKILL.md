---
name: using-git-worktrees
description: "Use when starting feature work that needs isolation from current workspace or before executing implementation plans. 启动需要隔离的功能工作或执行实现计划前使用。"
metadata:
  openclaw:
    emoji: "🌳"
---

# Using Git Worktrees（使用 Git Worktrees）

## 概述

Git worktrees 创建共享相同仓库的隔离工作区，允许同时处理多个分支而不切换。

**核心原则：系统的目录选择 + 安全验证 = 可靠隔离。**

**开始时宣布：** "我正在使用 using-git-worktrees skill 设置隔离工作区。"

---

## 目录选择流程

按此优先顺序：

### 1. 检查现有目录

```bash
# 按优先顺序检查
ls -d .worktrees 2>/dev/null     # 首选（隐藏）
ls -d worktrees 2>/dev/null      # 备选
```

**如果找到：** 使用该目录。如果都存在，`.worktrees` 优先。

### 2. 检查项目配置

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**如果指定偏好：** 不用问就使用它。

### 3. 询问用户

如果没有目录存在且没有配置偏好：

```
未找到 worktree 目录。我应该在哪里创建 worktrees？

1. .worktrees/（项目本地、隐藏）
2. ~/.config/worktrees/<project-name>/（全局位置）

你更喜欢哪个？
```

---

## 安全验证

### 对于项目本地目录（.worktrees 或 worktrees）

**必须在创建 worktree 前验证目录被忽略：**

```bash
# 检查目录是否被忽略（遵守 local、global 和 system gitignore）
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：**

按规则"立即修复坏东西"：

1. 添加适当行到 .gitignore
2. 提交更改
3. 继续创建 worktree

**为什么关键：** 防止意外提交 worktree 内容到仓库。

### 对于全局目录（~/.config/worktrees）

不需要 .gitignore 验证 — 完全在项目外。

---

## 创建步骤

### 1. 检测项目名称

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 创建 Worktree

```bash
# 确定完整路径
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/worktrees/*)
    path="~/.config/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# 用新分支创建 worktree
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 运行项目设置

自动检测并运行适当设置：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 验证干净基线

运行测试确保 worktree 干净开始：

```bash
# 示例 - 使用项目适当命令
npm test
cargo test
pytest
go test ./...
```

**如果测试失败：** 报告失败，询问是继续还是调查。

**如果测试通过：** 报告就绪。

### 5. 报告位置

```
Worktree 准备好于 <full-path>
测试通过（<N> 测试，0 失败）
准备实现 <feature-name>
```

---

## 快速参考

| 情况                       | 动作                     |
| -------------------------- | ------------------------ |
| `.worktrees/` 存在         | 使用它（验证被忽略）     |
| `worktrees/` 存在          | 使用它（验证被忽略）     |
| 都存在                     | 使用 `.worktrees/`       |
| 都不存在                   | 检查配置 → 询问用户      |
| 目录未被忽略               | 添加到 .gitignore + 提交 |
| 基线测试失败               | 报告失败 + 询问          |
| 无 package.json/Cargo.toml | 跳过依赖安装             |

---

## 常见错误

### 跳过忽略验证

- **问题：** Worktree 内容被跟踪、污染 git 状态
- **修复：** 创建项目本地 worktree 前总是使用 `git check-ignore`

### 假设目录位置

- **问题：** 创建不一致、违反项目约定
- **修复：** 遵循优先级：现有 > 配置 > 问

### 带失败测试继续

- **问题：** 无法区分新 bug 和预先存在的问题
- **修复：** 报告失败、获取明确许可继续

### 硬编码设置命令

- **问题：** 在使用不同工具的项目上失败
- **修复：** 从项目文件自动检测（package.json 等）

---

## 示例工作流

```
你：我正在使用 using-git-worktrees skill 设置隔离工作区。

[检查 .worktrees/ - 存在]
[验证被忽略 - git check-ignore 确认 .worktrees/ 被忽略]
[创建 worktree: git worktree add .worktrees/auth -b feature/auth]
[运行 npm install]
[运行 npm test - 47 通过]

Worktree 准备好于 /Users/jesse/myproject/.worktrees/auth
测试通过（47 测试，0 失败）
准备实现 auth 功能
```

---

## 红旗警告

**绝不：**

- 创建 worktree 不验证它被忽略（项目本地）
- 跳过基线测试验证
- 不询问就带失败测试继续
- 模糊时假设目录位置
- 跳过配置检查

**总是：**

- 遵循目录优先级：现有 > 配置 > 问
- 对项目本地验证目录被忽略
- 自动检测并运行项目设置
- 验证干净测试基线

---

## 集成

**被以下调用：**

- **brainstorming**（Phase 4）— 设计批准并跟进实现时 REQUIRED
- **subagent-driven-development** — 执行任何任务前 REQUIRED
- **executing-plans** — 执行任何任务前 REQUIRED
- 任何需要隔离工作区的 skill

**配合：**

- **finishing-a-development-branch** — 工作完成后清理 REQUIRED
