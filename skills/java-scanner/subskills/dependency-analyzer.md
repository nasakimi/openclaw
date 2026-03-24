# Java 依赖关系分析器

> **核心理念**：依赖关系是系统复杂度的核心指标。好的架构依赖清晰、单向、无循环。

## 分析维度

| 维度         | 说明                      | 输出         |
| ------------ | ------------------------- | ------------ |
| **模块依赖** | Maven/Gradle 模块间的依赖 | 模块依赖图   |
| **包依赖**   | Java 包之间的依赖         | 包依赖图     |
| **类依赖**   | 类之间的继承、组合、调用  | 类关系图     |
| **循环检测** | 依赖环路检测              | 循环依赖报告 |

---

## 第一部分：模块依赖分析

### Maven 模块依赖提取

```bash
# 提取所有模块的依赖关系
find . -name "pom.xml" -exec sh -c '
  module=$(grep -o "<module>.*</module>" $(dirname {})/../pom.xml 2>/dev/null | grep -o ">.*<" | tr -d ">" | tr -d "<" | grep -n "^" | grep "$(basename $(dirname {})):" | cut -d: -f1)
  if [ -n "$module" ]; then
    echo "Module: $(dirname {} | sed "s|$(pwd)/||")"
    grep -A 100 "<dependencies>" {} 2>/dev/null | grep -B 1 "</dependency>" | grep "<groupId>\|<artifactId>" | paste - - | grep "<groupId>com.xxx"
  fi
' \;
```

### 输出格式：模块依赖矩阵

```markdown
## 模块依赖矩阵

### 依赖关系表

| 源模块          | 目标模块  | 依赖类型 | 说明     |
| --------------- | --------- | -------- | -------- |
| user-service    | common    | compile  | 公共组件 |
| user-service    | user-api  | compile  | API 定义 |
| order-service   | common    | compile  | 公共组件 |
| order-service   | user-api  | compile  | 用户 API |
| payment-service | order-api | compile  | 订单 API |

### 依赖层级（分层架构）
```

Layer 0: common (无外部依赖)
Layer 1: user-api, order-api, payment-api (依赖 common)
Layer 2: user-service, order-service, payment-service (依赖 api)

```

```

---

## 第二部分：包依赖分析

### 包依赖提取

```bash
# 提取包的 import 依赖
find . -name "*.java" -path "*/src/main/*" -exec sh -c '
  pkg=$(dirname {} | sed "s|.*/src/main/java/||" | tr "/" ".")
  imports=$(grep "^import com.xxx" {} | cut -d" " -f2 | tr -d ";" | cut -d. -f1-4 | sort -u)
  for imp in $imports; do
    if [ "$pkg" != "$imp" ]; then
      echo "$pkg -> $imp"
    fi
  done
' \; | sort | uniq -c | sort -rn | head -50
```

### 输出格式：包依赖热点

```markdown
## 包依赖热点

### 被依赖最多的包（核心包）

| 排名 | 包路径                | 被依赖次数 | 依赖它的包 |
| ---- | --------------------- | ---------- | ---------- |
| 1    | com.xxx.common.util   | 156        | 45 个包    |
| 2    | com.xxx.common.entity | 89         | 32 个包    |
| 3    | com.xxx.user.api      | 45         | 12 个包    |

### 跨模块依赖（潜在问题）

| 源包                    | 目标包                    | 问题描述                     |
| ----------------------- | ------------------------- | ---------------------------- |
| com.xxx.order.service   | com.xxx.user.service.impl | 直接依赖实现类（应依赖 API） |
| com.xxx.payment.service | com.xxx.order.entity      | 跨模块依赖实体（应使用 DTO） |
```

---

## 第三部分：类依赖分析

### 类依赖类型

| 依赖类型 | 说明               | 检测方式      |
| -------- | ------------------ | ------------- |
| **继承** | extends 关键字     | 正则匹配      |
| **实现** | implements 关键字  | 正则匹配      |
| **组合** | 类成员变量         | 字段类型      |
| **依赖** | 方法参数、局部变量 | import + 使用 |

### 类依赖提取

```bash
# 继承关系
grep -rn "extends\s\+[A-Z]" --include="*.java" | head -30

# 实现关系
grep -rn "implements\s\+[A-Z]" --include="*.java" | head -30

# 组合关系（成员变量）
grep -rn "private\s\+[A-Z]" --include="*.java" | grep -v "private\s\+String\|Integer\|Long" | head -30
```

---

## 第四部分：循环依赖检测

### 循环依赖的危害

1. **编译问题**：无法确定编译顺序
2. **理解困难**：无法独立理解一个模块
3. **测试困难**：无法独立测试
4. **维护困难**：修改一处影响多处

### 循环依赖检测方法

```bash
# 包级别循环检测（使用 jdeps）
jdeps -verbose:class -filter:none target/classes | grep "cycle"

# Maven 模块循环检测
mvn dependency:tree -Dverbose | grep "cyclic"
```

### 输出格式：循环依赖报告

```markdown
## 循环依赖报告

### ⚠️ 发现循环依赖：2 个

#### 循环 1：模块循环（严重）
```

user-service → order-api → user-api → user-service

```

**分析**：
- user-service 依赖 order-api（调用订单接口）
- order-api 依赖 user-api（订单需要用户 DTO）
- user-api 依赖 user-service（用户 DTO 引用用户实体）

**影响**：无法独立部署 user-service

**建议**：将共享 DTO 提取到独立的 user-dto 模块
```

---

## 第五部分：依赖度量指标

### 度量维度

| 指标                       | 说明       | 健康值  | 计算方式                 |
| -------------------------- | ---------- | ------- | ------------------------ |
| **Afferent Coupling (Ca)** | 被依赖数   | -       | 其他模块依赖此模块的数量 |
| **Efferent Coupling (Ce)** | 依赖数     | < 20    | 此模块依赖其他模块的数量 |
| **Instability (I)**        | 不稳定性   | 0.0-1.0 | I = Ce / (Ca + Ce)       |
| **Abstractness (A)**       | 抽象度     | 0.0-1.0 | 抽象类数量 / 总类数量    |
| **Distance from Main (D)** | 距离主序列 | < 0.3   | \|A + I - 1\|            |

### 健康指标参考

```
理想状态：
- 工具模块：高 Ca，低 Ce，I 接近 0（稳定）
- 业务模块：低 Ca，高 Ce，I 接近 1（不稳定但可接受）
- 接口模块：高 A，适中的 I

不健康状态：
- I = 0 且 Ce > 0：难以修改的模块（僵化）
- I = 1 且 Ca > 0：修改影响大的模块（脆弱）
- D > 0.3：偏离最佳实践
```

---

## 与本体论的集成

### 依赖关系作为本体论的「关系规则」

```json
{
  "relation_type": "dependency",
  "source": "order-service",
  "target": "user-api",
  "dependency_type": "compile",
  "metrics": {
    "afferent_coupling": 8,
    "efferent_coupling": 12,
    "instability": 0.6
  }
}
```

---

## 经验沉淀

### 常见问题模式

1. **API 模块依赖实现模块**
   - 问题：接口依赖了实现
   - 解决：提取 DTO 到独立模块

2. **服务间双向依赖**
   - 问题：A 服务调 B，B 也调 A
   - 解决：引入消息队列或回调接口

3. **公共模块膨胀**
   - 问题：所有东西都往 common 放
   - 解决：按职责拆分 common

### 依赖治理原则

1. **单向依赖**：依赖只能从上层指向下层
2. **接口隔离**：模块间只通过 API 模块通信
3. **稳定依赖**：依赖应该指向稳定的模块
4. **无循环**：任何形式的循环依赖都必须消除
