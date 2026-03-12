# Java 项目本体论分析器

> **核心理念**：代码不值钱，值钱的是系统中的【元元素】+【关系规则】+【业务场景】组合。
>
> 一个复杂系统，核心元元素通常只有 **100-200 个**，它们的组合构成了整个系统。

## 本体论三要素

| 要素         | 说明               | 示例                       |
| ------------ | ------------------ | -------------------------- |
| **元元素**   | 系统的原子概念     | 用户、订单、支付、状态枚举 |
| **关系规则** | 元元素之间的交互   | 用户 → 下单 → 订单 → 支付  |
| **业务场景** | 关系规则的具体组合 | "用户申请贷款并放款" 场景  |

---

## 元元素类型

| 类型                     | 说明             | 识别方式                      | 示例                      |
| ------------------------ | ---------------- | ----------------------------- | ------------------------- |
| **实体 (Entity)**        | 核心业务对象     | @Entity、@Table               | User, Order, Payment      |
| **值对象 (ValueObject)** | 不可变的业务概念 | DTO、VO、Request/Response     | Address, Money, UserInfo  |
| **枚举 (Enum)**          | 业务状态/类型    | enum 关键字                   | OrderStatus, UserType     |
| **服务 (Service)**       | 业务行为载体     | @Service、\*Service.java      | UserService, OrderService |
| **接口 (API)**           | 对外暴露点       | @RestController、@FeignClient | /api/user/create          |
| **事件 (Event)**         | 业务事件         | \*Event.java、MQ 消息         | OrderCreatedEvent         |
| **规则 (Rule)**          | 业务规则         | *Rule.java、*Validator.java   | CreditCheckRule           |
| **外部系统 (External)**  | 依赖的外部服务   | Feign Client、HTTP 调用       | AlipayClient              |

---

## 分析流程

### Phase 1: 扫描元元素

```bash
# 实体
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "@Entity\|@Table"

# 枚举
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "^public enum"

# 服务
find . -name "*Service.java" -path "*/src/main/*" | grep -v Test

# API
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "@RestController\|@Controller"
```

### Phase 2: 分析关系规则

```bash
# 调用关系
grep -rn "userService\|orderService" --include="*.java" | grep -v "import"

# 继承关系
grep -rn "extends\|implements" --include="*.java" | head -50

# 状态字段
grep -rn "private.*Status\|private.*Type" --include="*.java"
```

### Phase 3: 识别业务场景

通过分析：

1. API 入口 → 调用链 → 最终状态变化
2. 事件触发 → 处理逻辑 → 后续动作
3. 定时任务 → 业务操作 → 数据变更

---

## 输出格式

### ontology-manifest.md（本体论总览）

```markdown
# 项目本体论总览

## 统计

- 元元素总数: 186
- 关系规则: 45
- 业务场景: 12

## 元元素分布

| 类型     | 数量 |
| -------- | ---- |
| 实体     | 35   |
| 枚举     | 48   |
| 服务     | 42   |
| API      | 156  |
| 事件     | 8    |
| 规则     | 12   |
| 外部系统 | 15   |

## 核心元元素（按被依赖次数排序）

1. User（被依赖 89 次）
2. Order（被依赖 67 次）
3. Payment（被依赖 34 次）
```

### ontology-elements.md（元元素清单）

```markdown
## 核心实体

| 元素ID | 名称    | 表名      | 字段数 | 业务含义 | 文件位置           |
| ------ | ------- | --------- | ------ | -------- | ------------------ |
| E001   | User    | t_user    | 15     | 用户信息 | UserEntity.java    |
| E002   | Order   | t_order   | 22     | 订单信息 | OrderEntity.java   |
| E003   | Payment | t_payment | 18     | 支付记录 | PaymentEntity.java |

## 业务枚举

| 元素ID | 名称        | 值数量 | 业务含义 | 关联实体     |
| ------ | ----------- | ------ | -------- | ------------ |
| EN001  | OrderStatus | 5      | 订单状态 | Order.status |
| EN002  | UserType    | 3      | 用户类型 | User.type    |

## 服务组件

| 元素ID | 名称         | 方法数 | 业务职责 | 依赖的元元素      |
| ------ | ------------ | ------ | -------- | ----------------- |
| S001   | UserService  | 12     | 用户管理 | E001, EN002       |
| S002   | OrderService | 18     | 订单管理 | E002, EN001, E001 |
```

### ontology-relations.md（关系规则）

```markdown
## 数据关系

| 关系ID | 源元素 | 关系类型 | 目标元素 | 说明             |
| ------ | ------ | -------- | -------- | ---------------- |
| DR001  | User   | 1:N      | Order    | 用户拥有多个订单 |
| DR002  | Order  | 1:N      | Payment  | 订单对应多笔支付 |

## 调用关系

| 关系ID | 调用方       | 被调用方       | 调用方法           | 触发场景   |
| ------ | ------------ | -------------- | ------------------ | ---------- |
| CR001  | UserService  | OrderService   | createDefaultOrder | 用户注册时 |
| CR002  | OrderService | PaymentService | createPayment      | 下单时     |

## 状态转换

| 转换ID | 实体  | 初始状态 | 触发条件 | 目标状态 |
| ------ | ----- | -------- | -------- | -------- |
| ST001  | Order | PENDING  | 支付成功 | PAID     |
| ST002  | Order | PAID     | 发货     | SHIPPED  |
```

### ontology-scenarios.md（业务场景）

```markdown
## 场景 S001: 用户注册

| 阶段 | 操作         | 涉及元元素          | 触发关系 |
| ---- | ------------ | ------------------- | -------- |
| 1    | 创建用户     | User, UserService   | A001     |
| 2    | 发送欢迎短信 | SmsService          | CR005    |
| 3    | 创建默认订单 | Order, OrderService | CR001    |

## 场景 S002: 下单支付

| 阶段 | 操作         | 涉及元元素              | 状态变化       |
| ---- | ------------ | ----------------------- | -------------- |
| 1    | 创建订单     | Order, OrderService     | - → PENDING    |
| 2    | 发起支付     | Payment, PaymentService | -              |
| 3    | 支付回调     | AlipayClient            | -              |
| 4    | 更新订单状态 | Order                   | PENDING → PAID |
```

---

## 分批执行策略

### 大项目（元元素 > 200）

```
第1步: 按模块分批扫描
  模块1: user-service → ontology-elements-user.md
  模块2: order-service → ontology-elements-order.md
  ...

第2步: 合并分析跨模块关系
  → ontology-relations.md

第3步: 识别业务场景
  → ontology-scenarios.md
```

### 中小项目（元元素 < 200）

```
一次性全量扫描 → ontology-*.md
```
