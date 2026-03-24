---
name: java-scanner
description: |
  Java 项目扫描统一入口 - 协调所有 Scanner 并行/串行执行，支持增量更新，生成项目知识库。
  当用户说"扫描这个项目"、"全面分析代码"、"分析这个屎山"、"理解这个项目"时触发。
  适用于 OpenClaw Agent 运行时环境。
license: MIT
---

# OpenClaw Java 项目扫描器

> **统一入口**：协调所有 java-\* Scanner，支持并行执行、增量更新、进度追踪。
>
> **核心价值**：从代码中提取【元元素】+【关系规则】+【业务场景】的组合，这才是真正值钱的东西。

## 快速开始

### 一键全量扫描

```
用户: 扫描这个项目
用户: 全面分析这个代码库
用户: 帮我理解这个屎山代码

AI: [执行全面扫描]
    Phase 1: 项目概览（统计规模）
    Phase 2: 架构扫描（模块、依赖、技术栈）
    Phase 3: 并行扫描（实体、API、枚举、配置）
    Phase 4: 本体论提取（元元素、关系、场景）

    生成文件: project-*.md (10+ 个)
```

### 本体论视角

```
用户: 分析这个项目的本体论
用户: 提取这个项目的核心元元素

AI: [执行本体论分析]
    1. 扫描元元素（实体、枚举、API、服务）
    2. 分析关系规则（调用链、数据流）
    3. 识别业务场景（核心业务流程）
    4. 生成：ontology-manifest.md
```

---

## Scanner 体系

### 总览

```
java-scanner (统一入口)
    │
    ├── Phase 1: 快速概览
    │   └── 架构扫描              → project-architecture.md
    │
    ├── Phase 2: 详细扫描 (并行)
    │   ├── API 接口扫描          → project-api-*.md
    │   ├── 实体扫描              → project-entity-*.md
    │   ├── 枚举扫描              → project-enums-*.md
    │   ├── 配置扫描              → project-config-guide.md
    │   ├── 公共类扫描            → project-common-*.md
    │   └── 外部调用扫描          → project-external-*.md
    │
    ├── Phase 3: 深度分析 (可选)
    │   ├── 流程分析              → project-flow-*.md
    │   ├── 依赖分析              → dependency-*.md
    │   └── 代码坏味道检测        → code-smells-*.md
    │
    └── Phase 4: 本体论
        └── 本体论提取            → ontology-*.md
```

---

## 本体论核心概念

> **核心理念**：代码不值钱，值钱的是系统中的【元元素】+【关系规则】+【业务场景】组合。
>
> 一个复杂系统，核心元元素通常只有 **100-200 个**，它们的组合构成了整个系统。

### 本体论三要素

| 要素         | 说明               | 示例                       |
| ------------ | ------------------ | -------------------------- |
| **元元素**   | 系统的原子概念     | 用户、订单、支付、状态枚举 |
| **关系规则** | 元元素之间的交互   | 用户 → 下单 → 订单 → 支付  |
| **业务场景** | 关系规则的具体组合 | "用户申请贷款并放款" 场景  |

### 元元素类型

| 类型                     | 说明             | 识别方式                      |
| ------------------------ | ---------------- | ----------------------------- |
| **实体 (Entity)**        | 核心业务对象     | @Entity、@Table               |
| **值对象 (ValueObject)** | 不可变的业务概念 | DTO、VO、Request/Response     |
| **枚举 (Enum)**          | 业务状态/类型    | enum 关键字                   |
| **服务 (Service)**       | 业务行为载体     | @Service、\*Service.java      |
| **接口 (API)**           | 对外暴露点       | @RestController、@FeignClient |
| **事件 (Event)**         | 业务事件         | \*Event.java、MQ 消息         |
| **规则 (Rule)**          | 业务规则         | *Rule.java、*Validator.java   |
| **外部系统 (External)**  | 依赖的外部服务   | Feign Client、HTTP 调用       |

---

## 执行策略

### 大项目分批执行

> **重要**：存量 Java 项目类成千上万，必须分批执行！

#### 阶段 1：快速扫描（估计元元素数量）

```bash
# 统计各类元元素数量
echo "实体: $(find . -name "*.java" | xargs grep -l "@Entity" | wc -l)"
echo "枚举: $(find . -name "*.java" | xargs grep -l "^public enum" | wc -l)"
echo "服务: $(find . -name "*Service.java" -path "*/src/main/*" | grep -v Test | wc -l)"
echo "API: $(find . -name "*.java" | xargs grep -l "@RestController" | wc -l)"
```

#### 阶段 2：按模块提取元元素

```
模块1: user-service → 提取元元素 → 写入 ontology-elements-user.md
模块2: order-service → 提取元元素 → 写入 ontology-elements-order.md
...
```

#### 阶段 3：合并分析关系

```
分析跨模块调用关系 → 写入 ontology-relations.md
识别业务场景 → 写入 ontology-scenarios.md
```

---

## 分批扫描规则

### 架构扫描

```bash
# 识别项目类型和构建工具
find . -name "pom.xml" -o -name "build.gradle" | head -20

# 多模块项目
grep -o "<module>.*</module>" pom.xml 2>/dev/null | head -20

# 主要依赖
grep -E "spring-boot|mybatis|dubbo|redis|kafka" pom.xml 2>/dev/null | head -30
```

**输出**: project-architecture.md

- 模块结构
- 技术栈清单
- 包结构分层

---

### 实体扫描

```bash
# JPA/Hibernate 实体
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "@Entity\|@Table" | head -50

# MyBatis 实体（通常在 *Entity.java, *DO.java）
find . -name "*Entity.java" -o -name "*DO.java" -path "*/src/main/*" | head -50
```

**输出**: project-entity-\*.md

- 实体清单（类名、表名、字段数）
- ER 关系图（Mermaid）
- 实体基类

---

### API 扫描

```bash
# Spring MVC 控制器
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "@RestController\|@Controller" | head -50

# RPC 接口
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "@FeignClient\|@DubboService\|@GrpcService" | head -30

# 消息消费者
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "@RabbitListener\|@KafkaListener\|@RocketMQMessageListener" | head -30

# 定时任务
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "@Scheduled\|@XxlJob" | head -20
```

**输出**: project-api-\*.md

- API 接口清单（路径、方法、参数、说明）
- RPC 服务清单
- 消息消费者清单
- 定时任务清单

---

### 枚举扫描

```bash
# 所有枚举类
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "^public enum" | head -100
```

**输出**: project-enums-\*.md

- 状态类枚举（\*Status）
- 类型类枚举（\*Type）
- 错误码枚举（*Code, *Error）

---

### 外部调用扫描

```bash
# Feign 客户端
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "@FeignClient" | head -30

# RestTemplate 使用
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "RestTemplate" | head -30

# 第三方 SDK（支付宝、微信等）
grep -rn "AlipayClient\|WeChatPay\|TencentSms" --include="*.java" | head -20
```

**输出**: project-external-\*.md

- 内部服务调用
- 第三方服务调用
- 配置位置

---

## 代码坏味道检测

### 可检测的坏味道

| 类别       | 坏味道       | 检测规则                    |
| ---------- | ------------ | --------------------------- |
| **膨胀类** | 过长方法     | 方法行数 > 50               |
| **膨胀类** | 过大类       | 类行数 > 300 或 方法数 > 20 |
| **膨胀类** | 过长参数列表 | 参数 > 5 个                 |
| **多余物** | 重复代码     | 相似代码块                  |
| **耦合类** | 中间人       | > 50% 方法只是委托          |
| **耦合类** | 消息链       | 调用链 > 3 层               |

### 检测命令

```bash
# 过长方法
find . -name "*.java" -path "*/src/main/*" | xargs -I {} sh -c '
  lines=$(wc -l < {})
  methods=$(grep -c "public\|private\|protected" {} 2>/dev/null || echo 0)
  if [ "$lines" -gt 300 ] || [ "$methods" -gt 20 ]; then
    echo "过大类: {} ($lines 行, $methods 方法)"
  fi
'

# 过长参数列表
grep -rn "public.*(.*)\s*{" --include="*.java" | \
  awk -F"[(),]" "{if(NF>6) print}"

# 重复代码（简化检测）
find . -name "*.java" -path "*/src/main/*" -exec md5sum {} \; | \
  sort | uniq -w32 -D | head -20
```

---

## 依赖分析

### 模块依赖提取

```bash
# 提取所有模块的依赖关系
find . -name "pom.xml" -exec sh -c '
  module=$(grep -o "<module>.*</module>" $(dirname {})/../pom.xml 2>/dev/null | grep -o ">.*<" | tr -d ">" | tr -d "<" | grep -n "^" | grep "$(basename $(dirname {})):")
  if [ -n "$module" ]; then
    echo "Module: $(dirname {} | sed "s|$(pwd)/||")"
    grep -A 100 "<dependencies>" {} 2>/dev/null | grep -B 1 "</dependency>" | grep "<groupId>\|<artifactId>" | paste - - | grep "<groupId>com.xxx"
  fi
' \;
```

### 循环依赖检测

```bash
# Maven 模块循环检测
mvn dependency:tree -Dverbose | grep "cyclic"

# 使用 jdeps (JDK 自带)
jdeps -verbose:class -filter:none target/classes | grep "cycle"
```

### 依赖度量指标

| 指标                       | 说明       | 健康值  | 计算方式                 |
| -------------------------- | ---------- | ------- | ------------------------ |
| **Afferent Coupling (Ca)** | 被依赖数   | -       | 其他模块依赖此模块的数量 |
| **Efferent Coupling (Ce)** | 依赖数     | < 20    | 此模块依赖其他模块的数量 |
| **Instability (I)**        | 不稳定性   | 0.0-1.0 | I = Ce / (Ca + Ce)       |
| **Abstractness (A)**       | 抽象度     | 0.0-1.0 | 抽象类数量 / 总类数量    |
| **Distance from Main (D)** | 距离主序列 | < 0.3   | \|A + I - 1\|            |

---

## 增量更新

### 变化检测

```bash
# 检测元元素变化
git diff --name-only | grep -E "(Entity|Enum|Service|Controller)\.java$"

# 检测配置变化
git diff --name-only | grep -E "\.(yml|properties|xml)$"
```

### 更新策略

| 变化类型 | 更新方式                    |
| -------- | --------------------------- |
| 新增实体 | 追加到 project-entity-\*.md |
| 新增 API | 追加到 project-api-\*.md    |
| 新增枚举 | 追加到 project-enums-\*.md  |
| 大量变化 | 建议全量重扫                |

---

## 输出格式

### 元素清单

```markdown
## 元元素清单

### 核心实体

| 元素ID | 名称  | 表名    | 字段数 | 业务含义 | 文件位置         |
| ------ | ----- | ------- | ------ | -------- | ---------------- |
| E001   | User  | t_user  | 15     | 用户     | UserEntity.java  |
| E002   | Order | t_order | 22     | 订单     | OrderEntity.java |

### 业务枚举

| 元素ID | 名称        | 值数量 | 业务含义 | 关联实体     |
| ------ | ----------- | ------ | -------- | ------------ |
| EN001  | OrderStatus | 5      | 订单状态 | Order.status |

### 服务组件

| 元素ID | 名称        | 方法数 | 业务职责 | 依赖的元元素 |
| ------ | ----------- | ------ | -------- | ------------ |
| S001   | UserService | 12     | 用户管理 | E001, EN003  |

### API 接口

| 元素ID | 路径                  | 方法       | 业务操作 | 调用的服务 |
| ------ | --------------------- | ---------- | -------- | ---------- |
| A001   | POST /api/user/create | createUser | 创建用户 | S001       |

---

## 元素统计

- 实体: 25 个
- 枚举: 40 个
- 服务: 35 个
- API: 120 个
- **总计核心元元素: 220 个**
```

---

### 关系规则

```markdown
## 关系规则清单

### 数据关系

| 关系ID | 源元素 | 关系类型 | 目标元素 | 说明             |
| ------ | ------ | -------- | -------- | ---------------- |
| DR001  | User   | 1:N      | Order    | 用户拥有多个订单 |

### 调用关系

| 关系ID | 调用方      | 被调用方     | 调用方法           | 触发场景   |
| ------ | ----------- | ------------ | ------------------ | ---------- |
| CR001  | UserService | OrderService | createDefaultOrder | 用户注册时 |

### 状态转换

| 转换ID | 实体  | 初始状态 | 触发条件 | 目标状态 |
| ------ | ----- | -------- | -------- | -------- |
| ST001  | Order | PENDING  | 支付成功 | PAID     |
```

---

### 业务场景

```markdown
## 业务场景清单

### 场景 S001: 用户注册

| 阶段 | 操作         | 涉及元元素          | 触发关系 |
| ---- | ------------ | ------------------- | -------- |
| 1    | 创建用户     | User, UserService   | A001     |
| 2    | 发送欢迎短信 | SmsService          | CR005    |
| 3    | 创建默认订单 | Order, OrderService | CR001    |

### 场景 S002: 下单支付

| 阶段 | 操作         | 涉及元元素              | 状态变化       |
| ---- | ------------ | ----------------------- | -------------- |
| 1    | 创建订单     | Order, OrderService     | - → PENDING    |
| 2    | 发起支付     | Payment, PaymentService | -              |
| 3    | 支付回调     | AlipayClient            | -              |
| 4    | 更新订单状态 | Order                   | PENDING → PAID |
```

---

## 输出文件结构

```
.claude/skills/project-knowledge/
├── project-architecture.md      # 项目架构
├── project-api-*.md             # API 接口（按模块）
├── project-entity-*.md          # 实体模型（按模块）
├── project-enums-*.md           # 枚举定义
├── project-config-guide.md      # 配置指南
├── project-common-*.md          # 公共组件
├── project-external-*.md        # 外部依赖
├── ontology-manifest.md         # 本体论总览
├── ontology-elements.md         # 元元素清单
├── ontology-relations.md        # 关系规则
├── ontology-scenarios.md        # 业务场景
├── dependency-modules.md        # 模块依赖
├── dependency-packages.md       # 包依赖热点
├── dependency-cycles.md         # 循环依赖报告
├── dependency-metrics.md        # 依赖度量
└── code-smells.md               # 代码坏味道（可选）
```

---

## 与动态分析的结合

| 静态分析（本 Scanner） | 动态分析（APM） |
| ---------------------- | --------------- |
| API 接口清单           | API 调用链追踪  |
| 实体关系图             | SQL 执行分析    |
| 代码坏味道             | 性能瓶颈检测    |
| 依赖关系图             | 运行时依赖验证  |

**组合使用**：静态分析提供结构，动态分析验证实际行为。

---

## 最佳实践

### 首次扫描

1. 先执行快速扫描（架构 + 实体 + 枚举）
2. 确认输出符合预期
3. 再执行全面扫描（包含本体论）

### 定期更新

1. 每周检查一次 Skill 是否过期
2. 重大发布前全量更新
3. 日常开发用增量更新

### 团队协作

1. project-\*.md 纳入版本控制
2. 重大更新时通知团队
3. 新成员通过 Skill 快速上手
