---
name: "java-coding-standard"
description: "Java 编码规范：命名、Javadoc 注释、导入顺序、代码格式、常量与枚举、Java 版本适配、异常处理（自定义异常、记录与抛出择一）、日志规范（SLF4J、占位符）、工具类使用（Hutool）。当编写、重构或审查 Java 代码、统一代码风格时使用。"
---

# Java 编码规范

本 skill 提供 Java 语言层面的编码规范，适用于任意 Java 代码；功能模块相关规范（分层架构、对象模型、模块命名等）见 `java-feature-coding-standard`。

> 示例说明：本文及 `references/` 中的示例以占位业务名（Model 等）演示语法与结构，实际开发时替换为项目自身的业务名。

## 何时使用

- 编写 Java 代码时确定代码风格（命名、注释、导入、代码格式）
- 处理异常与日志（记录与抛出策略、日志格式）
- 使用常量、枚举、工具类时
- 按项目 Java 版本选择语言特性
- 代码审查时作为编码规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 任意 Java 代码（Controller、Service、工具类、脚本等），不限架构与框架
- Java 8 / 11 / 17 / 21（按项目版本使用最佳实践语法，见“六、Java 版本适配”）

### 不适用（边界）

- 分层架构、对象模型（PO/DTO/Param/VO）、模块命名（包/类/方法/Param/VO）等功能模块规范（参见 `java-feature-coding-standard` skill）
- 接口方法排序（参见 `java-method-ordering` skill）
- 数据库表设计与 SQL 脚本（参见 `db-design-standard` skill）
- 代码安全审查流程（参见 `java-code-review` skill）

## 一、命名规范

- 类名使用 UpperCamelCase（大驼峰）；方法名、变量名使用 camelCase（小驼峰）
- 避免缩写（除非通用缩写）；使用有意义的名称

```java
// ✅ private final ModelService modelService; private String modelId;
// ❌ private final ModelService ms;         private String mid;
```

> 常量与枚举的命名及使用规范见“五、常量与枚举规范”。

## 二、注释规范

- **文档注释**：类、方法、字段必须包含 Javadoc
- **方法内注释**：`//` 或 `/* ... */`
- **独立成行**：注释另起一行；**禁止尾随注释**

```java
// ❌ return BeanUtil.copyProperties(model, ModelGetByIdVO.class); // 转换为视图对象（尾随注释）
// ❌ 类、方法缺少 Javadoc
// ✅ 注释独立成行，类与方法均有 Javadoc
```

完整示例见 `references/style-examples.md`。

## 三、导入顺序

三组依次排列，组间空一行：

1. JDK 标准库
2. 第三方库
3. 项目内部

```java
// ❌ 三组混排
// ✅ java.util.* → cn.hutool.* / org.springframework.* → com.example.*
```

完整示例见 `references/style-examples.md`。

## 四、代码格式

- 4 空格缩进；大括号不换行；方法之间空一行；类成员变量之间空一行

完整示例见 `references/style-examples.md`。

## 五、常量与枚举规范

### 1. 枚举

- 使用 `@Getter` 注解；枚举值全大写、下划线分隔
- 状态、类型等字段使用英文字符串值（见名知意），取值与枚举名一一对应

**第一设计（默认）：英文字符串值**

```java
// 存储/传输使用枚举名
String status = ModelStatus.WAITING.name();          // 枚举 → 字符串（"WAITING"）
ModelStatus status = ModelStatus.valueOf(statusStr); // 字符串 → 枚举
```

**第二设计（备选）**：确有需要时（如历史兼容）才使用数字值（`ordinal()` 或 int 成员变量），数据库字段从 0 开始编号

```java
// ❌ 错误：状态存数字值，见名不知意，需查文档才能理解
model.setStatus(1);

// ✅ 正确：状态存英文字符串（枚举名），见名知意
model.setStatus(ModelStatus.WAITING.name());
```

### 2. 常量

- `public static final` 修饰；全大写、下划线分隔；集中管理在常量类中

```java
// ❌ 错误：魔法值散落在业务代码中
if ("prediction".equals(model.getType())) { ... }
// ✅ 正确：使用常量
if (ModelConstants.MODEL_TYPE_PREDICTION.equals(model.getType())) { ... }
```

完整示例见 `references/style-examples.md`。

## 六、Java 版本适配（按项目 Java 版本使用最佳实践语法）

- **以项目声明的 Java 版本为准**（`pom.xml` 的 `java.version` / `maven.compiler.source` / `build.gradle` 的 `sourceCompatibility`）：只使用该版本及以下支持的语言特性，禁止越级使用更高版本语法（否则编译失败）
- 示例代码默认以 Java 8 语法呈现；当项目版本更高时，应主动采用对应版本的最佳实践语法

| Java 版本 | 推荐使用的语言特性 | 说明 |
| --- | --- | --- |
| Java 8 | Lambda、Stream、Optional、方法引用、`LocalDateTime` | 示例代码默认使用的语法 |
| Java 11 | `var` 局部变量类型推断、`List.of/Set.of/Map.of`、`String.isBlank`、`Optional.isEmpty` | 适度使用，不滥用 `var` |
| Java 17 | record、sealed class、switch 表达式、文本块、`Stream.toList()` | record 优先用于 DTO/VO 等不可变数据载体，替代简单 `@Data` 类 |
| Java 21 | record pattern、switch 模式匹配、虚拟线程（`Thread.ofVirtual`） | 高并发 IO 场景考虑虚拟线程 |

**正反例（假设项目为 Java 17）：**

```java
// ❌ 错误：项目已支持 Java 17，仍使用低版本的冗长写法
@Data
@AllArgsConstructor
public class ModelDTO {
    private Long id;
    private String name;
}

// ✅ 正确：使用 Java 17 最佳实践语法
public record ModelDTO(Long id, String name) {
}
```

**正反例（假设项目为 Java 8）：**

```java
// ❌ 错误：使用了高于项目版本的语法，编译失败
public record ModelDTO(Long id) {}      // Java 14+ 语法
List<String> list = List.of("a", "b");    // Java 9+ 语法

// ✅ 正确：Java 8 语法
public class ModelDTO {
    private final Long id;
    // ...
}
List<String> list = Arrays.asList("a", "b");
```

## 七、异常处理规范

### 1. 自定义异常

- 继承 `RuntimeException` 或项目基础异常类
- 提供有意义的错误信息

### 2. 异常处理

- 不吞异常、不返回 null 掩盖错误
- **记录与抛出二者择一**：需要记录日志的异常不再抛出；抛出（含包装后抛出）的异常不记录日志（包装时保留原始异常作为 cause），由全局异常处理器统一记录，避免重复日志
- 使用 `Optional` 处理可能为 null 的值，避免 NPE

```java
// ❌ 错误：吞掉异常返回 null，调用方无法区分"不存在"与"系统错误"
// ❌ 错误：既 log.error 记录又抛出（含包装抛出），上层处理器会再记录一次，形成重复日志
// ✅ 正确：记录与抛出择一——要记录就不抛出；要抛出（含包装，保留 cause）就不记录，交由全局异常处理器统一记录
```

完整示例（含 `ExampleException` 类定义）见 `references/style-examples.md`。

## 八、日志规范

### 1. 日志注解与级别

- 使用 `@Slf4j` 注解，使用 `log` 变量记录日志
- **DEBUG**：调试信息；**INFO**：重要业务流程；**WARN**：警告信息；**ERROR**：错误信息

### 2. 日志格式

- 使用占位符而非字符串拼接
- 包含必要的上下文信息
- 异常日志包含堆栈信息
- 关键业务流程记录日志

```java
// ❌ 错误：字符串拼接（即使不输出也计算成本）、异常无堆栈、无上下文
// ❌ 错误：敏感信息直接入日志（如 password）
// ✅ 正确：占位符 + 上下文 + 堆栈
log.info("Model job ran successfully: jobId={}", id);
log.error("Model job failed: jobId={}", id, e);
```

完整示例见 `references/style-examples.md`。

## 九、工具类使用

### 1. Hutool 工具类

- `BeanUtil`：对象属性拷贝；`CollUtil`：集合操作；`CharSequenceUtil`：字符串操作
- `ArrayUtil`：数组操作；`LocalDateTimeUtil` / `DateUtil`：日期时间操作；`JSONUtil`：JSON 操作
- 注：`Optional` 为 JDK 标准库（`java.util.Optional`），不属于 Hutool

### 2. 项目工具类

以下仅为示例，实际开发以项目内已有的工具类为准：

- `CodeUtil`：编码生成；`UserInfoUtil`：用户信息设置；`CollectionUtil`：集合工具；`JobClientUtil`：任务调度客户端

完整示例见 `references/style-examples.md`。

## References（完整代码示例）

- `references/style-examples.md` — 注释、导入顺序、代码格式、枚举常量、异常、日志、工具类完整示例
