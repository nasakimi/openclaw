# Root Cause Tracing（根因追踪）

## 概述

Bug 经常出现在调用栈深处（在错误目录 git init、在错误位置创建文件、用错误路径打开数据库）。你的本能是在错误出现的地方修复，但那是在治疗症状。

**核心原则：向后追踪调用链直到找到原始触发点，然后在源头修复。**

## 何时使用

**使用场景：**

- 错误发生在执行深处（不在入口点）
- 堆栈跟踪显示长调用链
- 不清楚无效数据从哪里来
- 需要找出哪个测试/代码触发了问题

---

## 追踪过程

### 1. 观察症状

```
Error: git init failed in /Users/jesse/project/packages/core
```

### 2. 找到直接原因

**什么代码直接导致了这个？**

```typescript
await execFileAsync("git", ["init"], { cwd: projectDir });
```

### 3. 问：谁调用了这个？

```typescript
WorktreeManager.createSessionWorktree(projectDir, sessionId)
  → called by Session.initializeWorkspace()
  → called by Session.create()
  → called by test at Project.create()
```

### 4. 继续向上追踪

**传入了什么值？**

- `projectDir = ''`（空字符串！）
- 空字符串作为 `cwd` 解析为 `process.cwd()`
- 那就是源代码目录！

### 5. 找到原始触发点

**空字符串从哪里来？**

```typescript
const context = setupCoreTest(); // Returns { tempDir: '' }
Project.create("name", context.tempDir); // 在 beforeEach 之前访问！
```

---

## 添加堆栈跟踪

当你无法手动追踪时，添加工具代码：

```typescript
// 在有问题的操作之前
async function gitInit(directory: string) {
  const stack = new Error().stack;
  console.error("DEBUG git init:", {
    directory,
    cwd: process.cwd(),
    nodeEnv: process.env.NODE_ENV,
    stack,
  });

  await execFileAsync("git", ["init"], { cwd: directory });
}
```

**关键：** 在测试中使用 `console.error()`（不是 logger - 可能不显示）

**运行并捕获：**

```bash
npm test 2>&1 | grep 'DEBUG git init'
```

**分析堆栈跟踪：**

- 寻找测试文件名
- 找到触发调用的行号
- 识别模式（同一个测试？同一个参数？）

---

## 找出哪个测试导致污染

如果测试期间出现了什么但不知道是哪个测试：

使用二分脚本 `find-polluter.sh`：

```bash
./find-polluter.sh '.git' 'src/**/*.test.ts'
```

逐个运行测试，在第一个污染者处停止。

---

## 真实示例：空 projectDir

**症状：** `.git` 在 `packages/core/` 创建（源代码目录）

**追踪链：**

1. `git init` 在 `process.cwd()` 运行 ← 空 cwd 参数
2. WorktreeManager 用空 projectDir 调用
3. Session.create() 传递空字符串
4. 测试在 beforeEach 之前访问 `context.tempDir`
5. setupCoreTest() 初始返回 `{ tempDir: '' }`

**根因：** 顶层变量初始化访问空值

**修复：** 让 tempDir 成为 getter，如果在 beforeEach 之前访问则抛出

**同时添加纵深防御：**

- Layer 1: Project.create() 验证目录
- Layer 2: WorkspaceManager 验证非空
- Layer 3: NODE_ENV 守卫拒绝在 tmpdir 外 git init
- Layer 4: git init 前堆栈跟踪日志

---

## 关键原则

**永远不要只在错误出现的地方修复。** 追溯回去找到原始触发点。

## 堆栈跟踪技巧

**在测试中：** 使用 `console.error()` 而不是 logger - logger 可能被抑制
**操作前：** 在危险操作前记录，不是失败后
**包含上下文：** 目录、cwd、环境变量、时间戳
**捕获堆栈：** `new Error().stack` 显示完整调用链

---

## 真实影响

来自调试会话 (2025-10-03)：

- 通过 5 层追踪找到根因
- 在源头修复（getter 验证）
- 添加 4 层防御
- 1847 测试通过，零污染
