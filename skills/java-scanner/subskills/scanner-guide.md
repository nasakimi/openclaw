# Scanner 执行指南

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

### 1. 架构扫描

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

### 2. 实体扫描

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

### 3. API 扫描

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

### 4. 枚举扫描

```bash
# 所有枚举类
find . -name "*.java" -path "*/src/main/*" | xargs grep -l "^public enum" | head -100
```

**输出**: project-enums-\*.md

- 状态类枚举（\*Status）
- 类型类枚举（\*Type）
- 错误码枚举（*Code, *Error）

---

### 5. 外部调用扫描

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

### 6. 公共类扫描

```bash
# 工具类
find . -name "*Util.java" -o -name "*Utils.java" -o -name "*Helper.java" -path "*/src/main/*" | head -50

# 基础组件
find . -name "*Component.java" -o -name "*Config.java" -path "*/src/main/*" | head -30
```

**输出**: project-common-\*.md

- 工具类清单
- 基础组件清单
- 可复用代码

---

### 7. 配置扫描

```bash
# Spring 配置
find . -name "application*.yml" -o -name "application*.properties" | head -20

# Spring Boot 配置类
find . -name "*Config.java" -o -name "*Configuration.java" -path "*/src/main/*" | head -30
```

**输出**: project-config-guide.md

- 配置文件清单
- 关键配置项
- 环境差异

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
