# Java Scanner for OpenClaw

> **一站式 Java 项目分析器** - 从屎山代码中提取核心价值

## 核心价值

**代码不值钱，值钱的是：**

- **元元素**：系统的原子概念（~200个）
- **关系规则**：元元素之间的交互
- **业务场景**：关系规则的具体组合

## 快速开始

```
用户: 扫描这个项目
用户: 全面分析这个代码库
用户: 帮我理解这个屎山代码
```

## 文件结构

```
java-scanner/
├── SKILL.md                    # 主入口 - 协调所有 Scanner
└── subskills/
    ├── ontology-analyzer.md    # 本体论分析 - 提取元元素、关系、场景
    ├── code-smell-detector.md  # 坏味道检测 - 20 种代码坏味道
    ├── dependency-analyzer.md  # 依赖分析 - 模块/包/类依赖 + 循环检测
    └── scanner-guide.md        # 执行指南 - 分批扫描策略
```

## Scanner 体系

| Scanner      | 功能                 | 输出                    |
| ------------ | -------------------- | ----------------------- |
| 架构扫描     | 模块划分、技术栈     | project-architecture.md |
| API 扫描     | HTTP/RPC/MQ/定时任务 | project-api-\*.md       |
| 实体扫描     | 数据库实体、ER 关系  | project-entity-\*.md    |
| 枚举扫描     | 业务状态和常量       | project-enums-\*.md     |
| 配置扫描     | 配置文件和环境差异   | project-config-guide.md |
| 外部调用扫描 | 第三方服务依赖       | project-external-\*.md  |
| 本体论分析   | 元元素 + 关系 + 场景 | ontology-\*.md          |
| 代码坏味道   | 20 种坏味道检测      | code-smells.md          |
| 依赖分析     | 依赖图 + 循环检测    | dependency-\*.md        |

## 本体论三要素

### 1. 元元素类型

- **实体 (Entity)**: @Entity, @Table
- **值对象 (ValueObject)**: DTO, VO, Request/Response
- **枚举 (Enum)**: 业务状态/类型
- **服务 (Service)**: @Service, \*Service.java
- **接口 (API)**: @RestController, @FeignClient
- **事件 (Event)**: \*Event.java, MQ 消息
- **规则 (Rule)**: *Rule.java, *Validator.java
- **外部系统 (External)**: Feign Client, HTTP 调用

### 2. 关系规则

- 数据关系: User 1:N Order
- 调用关系: UserService → OrderService
- 状态转换: Order PENDING → PAID

### 3. 业务场景

- 用户注册流程
- 下单支付流程
- 贷款审批流程

## 代码坏味道检测

| 类别   | 坏味道       | 检测规则                |
| ------ | ------------ | ----------------------- |
| 膨胀类 | 过长方法     | 行数 > 50               |
| 膨胀类 | 过大类       | 行数 > 300 或 方法 > 20 |
| 膨胀类 | 过长参数列表 | 参数 > 5                |
| 耦合类 | 中间人       | > 50% 方法只是委托      |
| 耦合类 | 消息链       | 调用链 > 3 层           |
| 多余物 | 重复代码     | 相似代码块              |

## 依赖分析

- **模块依赖**: Maven/Gradle 模块间依赖
- **包依赖**: Java 包之间依赖热点
- **循环检测**: 依赖环路检测
- **度量指标**: Ca, Ce, I, A, D

## 分批执行策略

```
阶段 1: 快速扫描（估计元元素数量）
阶段 2: 按模块提取元元素
阶段 3: 合并分析关系
阶段 4: 识别业务场景
```

## 输出文件

```
project-architecture.md      # 项目架构
project-api-*.md             # API 接口
project-entity-*.md          # 实体模型
project-enums-*.md           # 枚举定义
ontology-manifest.md         # 本体论总览
ontology-elements.md         # 元元素清单
ontology-relations.md        # 关系规则
ontology-scenarios.md        # 业务场景
dependency-*.md              # 依赖分析
code-smells.md               # 代码坏味道
```

## 适用场景

- 遗留代码分析
- 系统重构评估
- 新人上手指导
- 代码质量审查
- 架构依赖治理
