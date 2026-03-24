# Java 代码坏味道检测器

> **核心理念**：代码坏味道是更深层次设计问题的症状。检测坏味道，找到根本原因。

## 什么是代码坏味道？

代码坏味道（Code Smell）是指代码中表面上的问题，它们通常暗示着更深层次的设计问题。

### 坏味道分类

| 类别                                     | 说明         | 坏味道数量 |
| ---------------------------------------- | ------------ | ---------- |
| **Bloater（膨胀类）**                    | 代码膨胀过大 | 5 种       |
| **Object-Orientation Abusers（OO滥用）** | 违反 OO 原则 | 4 种       |
| **Change Preventers（变化阻碍者）**      | 难以修改     | 3 种       |
| **Dispensables（多余物）**               | 不必要的代码 | 4 种       |
| **Couplers（耦合类）**                   | 过度耦合     | 4 种       |

---

## 可检测的坏味道清单

### 1. 膨胀类（Bloaters）

#### 1.1 过长方法（Long Method）

**检测规则**：方法行数 > 50 行

```java
// 坏味道示例
public void processOrder(Order order) {
    // 验证逻辑 10 行
    // 计算逻辑 15 行
    // 保存逻辑 10 行
    // 通知逻辑 10 行
    // 日志记录 10 行
    // 总计 55 行 → 坏味道！
}

// 修复建议：提取方法
public void processOrder(Order order) {
    validateOrder(order);
    calculateAmount(order);
    saveOrder(order);
    notifyCustomer(order);
    logOrder(order);
}
```

#### 1.2 过大类（Large Class）

**检测规则**：类行数 > 300 行 或 方法数 > 20

```java
// 坏味道：UserController 500 行，30 个方法
@Controller
public class UserController {
    // 用户 CRUD 5 个方法
    // 用户查询 5 个方法
    // 用户权限 5 个方法
    // 用户统计 5 个方法
    // 用户通知 5 个方法
    // 总计 30 个方法 → 职责过多！
}

// 修复建议：职责拆分
// UserCrudController (CRUD)
// UserQueryController (查询)
// UserPermissionController (权限)
```

#### 1.3 过长参数列表（Long Parameter List）

**检测规则**：方法参数 > 5 个

```java
// 坏味道
public void createUser(String name, String mobile, String email,
                       String idCard, int age, String address,
                       String city, String province) {
    // 8 个参数 → 坏味道！
}

// 修复建议：参数对象
public void createUser(UserCreateInfo info) {
    // 封装为对象
}
```

#### 1.4 数据泥团（Data Clumps）

**检测规则**：多个参数总是结伴出现

```java
// 坏味道：这三个参数总是同时出现
public void ship(String province, String city, String address);
public void deliver(String province, String city, String address);

// 修复建议：提取值对象
public void ship(Address address);
public void deliver(Address address);
```

---

### 2. OO 滥用（Object-Orientation Abusers）

#### 2.1 Switch 语句（Switch Statements）

**检测规则**：大量 switch/if-else 基于类型判断

```java
// 坏味道
public double calculate(Order order) {
    switch (order.getType()) {
        case "NORMAL": return order.getAmount();
        case "DISCOUNT": return order.getAmount() * 0.9;
        case "VIP": return order.getAmount() * 0.8;
    }
}

// 修复建议：多态
interface OrderType {
    double calculate(double amount);
}
```

#### 2.2 临时字段（Temporary Field）

**检测规则**：对象中有字段只在某些情况下使用

```java
// 坏味道
public class Order {
    private String discountCode;    // 仅折扣订单使用
    private String giftWrap;        // 仅礼品订单使用
    private String deliveryTime;    // 仅外卖订单使用
}
```

---

### 3. 变化阻碍者（Change Preventers）

#### 3.1 发散式变化（Divergent Change）

**检测规则**：一个类因多种原因需要修改

```java
// 坏味道：UserController 因多种原因变化
public class UserController {
    // 用户验证逻辑变化
    // 数据库结构变化
    // 前端接口变化
    // 都需要修改这个类
}
```

#### 3.2 霰弹式修改（Shotgun Surgery）

**检测规则**：一个变化需要修改多个类

```java
// 坏味道：新增一个订单类型需要修改 10+ 个类
// - OrderType 枚举
// - OrderService
// - OrderValidator
// - OrderCalculator
// ...
```

---

### 4. 多余物（Dispensables）

#### 4.1 重复代码（Duplicated Code）

**检测规则**：相似代码块在多处出现

```java
// 坏味道：重复的验证逻辑
public class UserService {
    public void createUser(UserDTO dto) {
        if (StringUtils.isEmpty(dto.getName())) {
            throw new BizException("姓名不能为空");
        }
        if (StringUtils.isEmpty(dto.getMobile())) {
            throw new BizException("手机号不能为空");
        }
    }
}

public class OrderService {
    public void createOrder(OrderDTO dto) {
        if (StringUtils.isEmpty(dto.getCustomerName())) {
            throw new BizException("客户姓名不能为空");
        }
        if (StringUtils.isEmpty(dto.getMobile())) {
            throw new BizException("手机号不能为空");
        }
    }
}

// 修复建议：提取公共验证方法
public class ValidationUtil {
    public static void validateNotEmpty(String value, String fieldName) {
        if (StringUtils.isEmpty(value)) {
            throw new BizException(fieldName + "不能为空");
        }
    }
}
```

#### 4.2 懒惰类（Lazy Class）

**检测规则**：类行数 < 20 行 或 只有 1-2 个方法

```java
// 坏味道
public class UserHelper {
    public String getFullName(User user) {
        return user.getFirstName() + " " + user.getLastName();
    }
}

// 修复建议：合并到 User 类
public class User {
    public String getFullName() {
        return firstName + " " + lastName;
    }
}
```

---

### 5. 耦合类（Couplers）

#### 5.1 依恋情节（Feature Envy）

**检测规则**：一个方法大量调用另一个类的数据

```java
// 坏味道
public class OrderService {
    public double calculateDiscount(Order order) {
        if (order.getUser().isVip()) {
            if (order.getUser().getLevel() > 3) {
                return order.getAmount() * 0.8;
            }
        }
        return order.getAmount() * 0.9;
    }
}

// 修复建议：将方法移到 Order 类
public class Order {
    public double calculateDiscount() {
        if (user.isVip() && user.getLevel() > 3) {
            return amount * 0.8;
        }
        return amount * 0.9;
    }
}
```

#### 5.2 中间人（Middle Man）

**检测规则**：类中 > 50% 的方法只是委托给其他类

```java
// 坏味道
public class OrderService {
    public void create(Order order) {
        repository.save(order);
    }
    public Order get(Long id) {
        return repository.findById(id);
    }
    // 90% 的方法只是委托
}
```

#### 5.3 消息链（Message Chains）

**检测规则**：方法调用链过长

```java
// 坏味道
String city = order.getUser().getAddress().getCity().getName();

// 修复建议：隐藏委托
String city = order.getUserCity();
```

---

## 检测命令

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

## 输出格式

```markdown
## 代码坏味道报告

### 摘要

| 类别       | 发现数量 | 严重程度 |
| ---------- | -------- | -------- |
| 膨胀类     | 15       | 中       |
| OO 滥用    | 8        | 高       |
| 变化阻碍者 | 5        | 高       |
| 多余物     | 12       | 低       |
| 耦合类     | 10       | 中       |
| **总计**   | **50**   | -        |

### 严重问题（需立即处理）

#### 1. 过大类: UserController (高优先级)

- **文件**: UserController.java
- **问题**: 520 行，32 个方法
- **影响**: 难以理解、难以测试、修改风险高
- **建议**: 拆分为 UserCrudController、UserQueryController
```

---

## 坏味道优先级

| 优先级 | 坏味道       | 原因           |
| ------ | ------------ | -------------- |
| P0     | 重复代码     | 维护成本高     |
| P0     | 过大类       | 难以理解和测试 |
| P1     | 过长方法     | 难以理解和复用 |
| P1     | 发散式变化   | 修改风险高     |
| P2     | 基本类型偏执 | 代码可读性差   |
| P2     | 过长参数列表 | 调用复杂       |
