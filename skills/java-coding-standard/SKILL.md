---
name: "java-coding-standard"
description: "Java 语言级编码规范：命名、注释、格式、常量与枚举、异常与错误码、日志、工具类（JDK/Spring/Hutool/Guava/Commons 等）与 Java 版本适配。当编写、重构或审查 Java 代码、统一代码风格时使用。"
---

# Java 编码规范

本 skill 提供 Java 语言层面的编码规范，适用于任意 Java 代码；功能模块相关规范（分层架构、对象模型、模块命名等）见 `java-feature-standard`。

> 风格基准：以《阿里巴巴 Java 开发手册》风格为基准（4 空格缩进、单行 120 字符、导入三组分类），与 Google Java Style 的差异（2 空格/100 字符/ASCII 排序）属有意选择。

> 示例说明：本文及 `references/` 中的示例以占位业务名（Model 等）演示语法与结构，实际开发时替换为项目自身的业务名。

## 何时使用

- 编写 Java 代码时确定代码风格（命名、注释、导入、代码格式）
- 处理异常与日志（记录与抛出策略、错误码、日志格式）
- 使用常量、枚举、工具类时
- 按项目 Java 版本选择语言特性
- 代码审查时作为编码规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 任意 Java 代码（Controller、Service、工具类、脚本等），不限架构与框架
- Java 8 / 11 / 17 / 21（按项目版本使用最佳实践语法，见“六、Java 版本适配”）

### 不适用（边界）

- 分层架构、对象模型（PO(Persistent)/DTO/PO(Param)/VO）、模块命名（包/类/方法/PO(Param)/VO）等功能模块规范（参见 `java-feature-standard` skill）
- 接口方法排序（参见 `java-method-ordering` skill）
- 数据库表设计与 SQL 脚本（参见 `db-design-standard` skill）
- 代码安全审查流程（参见 `java-code-review` skill）

## 一、命名规范

- 类名使用 UpperCamelCase（大驼峰）；方法名、变量名使用 camelCase（小驼峰）
- 避免缩写（除非通用缩写）；使用有意义的名称
- 禁止拼音与英文混合命名、禁止中文命名
- 禁止以下划线或美元符号开头/结尾（如 `_name` / `name_` / `$Object`）
- 抽象类使用 `Abstract` / `Base` 开头；异常类使用 `Exception` 结尾；测试类以被测类名开头、`Test` 结尾
- 数组类型声明为 `int[] array`（类型与中括号紧挨），禁止 `int args[]`
- 常量与变量命名时，表示类型的名词放在词尾（`startTime` / `workQueue` / `nameList`）
- 接口中的方法与属性不加修饰符（`public` 也不加），保持简洁；接口中不定义常量（除非与接口方法相关的基础常量）
- 使用设计模式时在命名中体现模式（`OrderFactory` / `LoginProxy` / `ResourceObserver`）
- 避免歧视性、侮辱性词语（`blockList` / `allowList` 而非 `blackList` / `whiteList`）

```java
// ✅ private final ModelService modelService; private String modelId;
// ❌ private final ModelService ms;         private String mid;
```

> 常量与枚举的命名及使用规范见“五、常量与枚举规范”。

## 二、注释规范

- **文档注释**：类、方法、字段必须包含 Javadoc
- **类注释模板**：必须包含创建者与创建日期（`@author` + `@date`，日期格式 `yyyy/MM/dd`）
- **方法内注释**：`//` 或 `/* ... */`
- **独立成行**：注释另起一行；**禁止尾随注释**
- **同步更新**：代码修改的同时必须同步修改注释（尤其参数、返回值、异常、核心逻辑）
- **特殊标记**：TODO / FIXME 必须注明标记人与标记时间，并定期清理

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
// ✅ java.util.List / java.util.Optional → cn.hutool.* / org.springframework.* → com.example.*
```

完整示例见 `references/style-examples.md`。

## 四、代码格式与类成员布局

- 4 空格缩进；大括号不换行；方法之间空一行；类成员变量之间空一行
- `if` / `else` / `for` / `while` / `do` 必须使用大括号（即使只有一行）
- 运算符两侧加空格；`if` / `for` / `while` 等保留字与括号之间加空格
- 单行不超过 120 字符；超长换行规则：第二行相对第一行缩进 4 个空格、运算符与点号随下文一起换行、多参数在逗号后换行、括号前不换行
- **类成员布局**：字段 → 构造器 → 公有/保护方法 → 私有方法 → getter/setter（置于最后）；接口/公有方法的内部顺序遵循 `java-method-ordering`；重载方法相邻放置

```java
// ✅ 正确
public class ModelServiceImpl {

    private final ModelMapper modelMapper;

    public ModelServiceImpl(ModelMapper modelMapper) {
        this.modelMapper = modelMapper;
    }

    // 公有方法（内部顺序遵循 java-method-ordering）
    public ModelGetByIdVO getById(Long id) { ... }

    // 私有方法
    private void doInternal() { ... }

    // getter/setter 置于最后
    public Long getId() { ... }
}
```

完整示例见 `references/style-examples.md`。

## 五、常量与枚举规范

### 1. 枚举

- 使用 `@Getter` 注解；枚举值全大写、下划线分隔
- 状态、类型等字段使用英文字符串值（见名知意），取值与枚举名一一对应
- 枚举注释按 `值（值说明）` 格式写明各取值（与数据库字段注释一致），如 `模型状态：WAITING（等待中）、STAGED（已暂存）、DEPLOYED（已部署）`

**第一设计（默认）：英文字符串值**

```java
// 存储/传输使用枚举名
String status = ModelStatus.WAITING.name();          // 枚举 → 字符串（"WAITING"）
ModelStatus status = ModelStatus.valueOf(statusStr); // 字符串 → 枚举
```

**第二设计（备选）**：确有需要时（如历史兼容、绝对性能场景）才使用数字值（int 成员变量，数据库字段从 0 开始编号）。慎用 `ordinal()`（依赖枚举声明顺序，调整顺序会破坏历史数据）

```java
// ❌ 错误：状态存数字值，见名不知意，需查文档才能理解
model.setStatus(1);

// ✅ 正确：状态存英文字符串（枚举名），见名知意
model.setStatus(ModelStatus.WAITING.name());
```

### 2. 常量

- `public static final` 修饰；全大写、下划线分隔
- `long` 字面量使用大写 `L`（小写 `l` 易与数字 1 混淆）；浮点字面量后缀统一大写 `D` / `F`
- **按功能分类维护**：禁止大而全的单一常量类；按功能归类（如缓存常量 `CacheConsts`、配置常量 `ConfigConsts`）

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

| Java 版本 | 推荐使用的语言特性（按引入版本标注，该 LTS 均可用） | 说明 |
| --- | --- | --- |
| Java 8 | Lambda、Stream、Optional、方法引用、`LocalDateTime` | 示例代码默认使用的语法 |
| Java 11 | `var` 局部变量类型推断（Java 10+）、`List.of/Set.of/Map.of`（Java 9+）、`String.isBlank`、`Optional.isEmpty` | 适度使用，不滥用 `var` |
| Java 17 | record（Java 16+，14/15 为预览）、sealed class、switch 表达式（Java 14+）、文本块（Java 15+）、`Stream.toList()`（Java 16+） | record 优先用于 DTO/VO 等不可变数据载体，替代简单 `@Data` 类 |
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
public record ModelDTO(Long id) {}      // record 需 Java 16+（14/15 为预览）
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
- **异常信息使用英文**（异常信息、提示语使用英文）
- **记录与抛出二者择一**：需要记录日志的异常不再抛出；抛出（含包装后抛出）的异常不记录日志（包装时保留原始异常作为 cause），由全局异常处理器统一记录，避免重复日志
- 使用 `Optional` 处理可能为 null 的值，避免 NPE

```java
// ❌ 错误：吞掉异常返回 null，调用方无法区分"不存在"与"系统错误"
// ❌ 错误：既 log.error 记录又抛出（含包装抛出），上层处理器会再记录一次，形成重复日志
// ✅ 正确：记录与抛出择一——要记录就不抛出；要抛出（含包装，保留 cause）就不记录，交由全局异常处理器统一记录
```

完整示例（含 `ExampleException` 类定义）见 `references/style-examples.md`。

### 3. Optional 使用规范

- `Optional` 仅用于方法返回值，禁止用作字段、方法参数、集合元素
- 禁止返回 `null` 的 Optional（应使用 `Optional.empty()`）
- 链式调用使用 `map` / `flatMap` / `orElseThrow`，避免 `get()` 直接取值（无值抛异常）
- 查询无结果可 `orElse(null)` 表达"无数据"语义（与"禁止返回 null 的 Optional"不冲突，需在方法 Javadoc 说明）

### 4. 错误码

> 以下为示例方案（5 位 A/B/C 设计），实际按项目错误码规范执行。

- 错误码为字符串类型（如 5 位：来源 + 编号），来源区分用户（A）/ 本系统（B）/ 第三方（C）
- 错误码不与版本号、错误等级、业务架构、组织架构挂钩
- 错误码不直接输出给用户作为提示信息（用户提示另设 user_tip）
- 对外的 HTTP / API 开放接口必须使用错误码；应用内部推荐异常抛出；跨应用 RPC 优先使用 `Result` 封装（含错误码与简短信息）
- 避免随意定义新错误码，优先复用已有错误码表

## 八、日志规范

### 1. 日志注解与级别

- 使用 `@Slf4j` 注解，使用 `log` 变量记录日志
- **DEBUG**：调试信息；**INFO**：重要业务流程；**WARN**：警告信息；**ERROR**：错误信息

### 2. 日志格式

- **日志使用英文**
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

> 选型原则：优先使用 JDK 标准库与框架自带工具类（零额外依赖）；需补充功能时按需引入第三方工具库，功能重叠的同类工具库二选一（示例默认 Hutool），混用会增加维护成本。默认映射：字符串/集合判空与操作 → Hutool（示例默认）；不可变集合 → Guava；JSON → Jackson；文件与流 → commons-io；参数校验 → `Preconditions` 或 Spring `Assert`。

### 1. JDK 标准库

- `Objects`：判空与比较（`Objects.isNull` / `Objects.equals`）；`Collections` / `Arrays`：集合与数组操作
- `Optional`：处理可能为 null 的返回值

### 2. Spring 自带工具类（Spring 项目零依赖首选）

- `StringUtils`（Spring 版）：`hasText` / `hasLength`（注：Spring 无 `isBlank`，该方法是 commons-lang3 的）；`CollectionUtils`（Spring 版）：`isEmpty`
- `BeanUtils`（Spring 版）：属性拷贝（浅拷贝）；`Assert`：断言校验；`StopWatch`：耗时统计；`StreamUtils`：流拷贝

### 3. Hutool 工具类

- `BeanUtil`：对象属性拷贝；`CollUtil`：集合操作；`CharSequenceUtil`：字符串操作
- `ArrayUtil`：数组操作；`LocalDateTimeUtil`：日期时间操作（Java 8+ 首选；`DateUtil` 用于遗留 `Date` 字段）；`JSONUtil`：JSON 操作

### 4. Guava 工具类

- `Lists` / `Sets` / `Maps`：集合创建；`ImmutableList` / `ImmutableSet` / `ImmutableMap`：不可变集合
- `Splitter` / `Joiner`：字符串拆分与连接
- `Preconditions`：参数校验（`checkNotNull` / `checkArgument`）
- `Strings`：字符串判空处理（`nullToEmpty` / `isNullOrEmpty`）

### 5. Apache Commons

- `StringUtils`（commons-lang3）：`isBlank` / `isNotBlank` / `join`；`RandomStringUtils`：随机字符串
- `CollectionUtils`（commons-collections4）：集合操作
- `FileUtils` / `IOUtils`（commons-io）：文件与流操作

### 6. 其他常用组件

- JSON：Jackson `ObjectMapper`（Spring 默认，序列化/反序列化首选）
- 本地缓存：Caffeine（高性能本地缓存，替代 Guava Cache）
- 对象映射：MapStruct（编译期生成映射代码，类型安全，替代运行时反射拷贝）

### 7. 项目工具类

以下仅为示例，实际开发以项目内已有的工具类为准：

- `CodeUtil`：编码生成；`UserInfoUtil`：用户信息设置；`CollectionUtil`：集合工具；`JobClientUtil`：任务调度客户端

完整示例见 `references/style-examples.md`。

## 十、控制语句与逻辑风格

- if-else 嵌套不超过 3 层；优先使用卫语句、策略模式、状态模式表达复杂分支
- 避免取反逻辑运算符（`if (x < 628)` 而非 `if (!(x >= 628))`）
- 不在条件表达式中插入赋值语句（赋值独立成行）
- 复杂条件判断提取为有意义的布尔变量，提高可读性

## 十一、其他编码细节

- `final` 使用场景：不允许被继承的类、不允许修改引用的域对象、不允许被覆写的方法、不允许重新赋值的局部变量
- 可变参数规范：可变参数置于参数列表最后；参数类型避免 `Object`；尽量少用可变参数
- 集合初始化时指定初始容量（如 `new HashMap<>(16)`；已知元素量时按 `元素数 / 0.75 + 1` 估算），避免频繁扩容
- JDK 7+ 使用 diamond 语法（`new HashMap<>()`）
- 资源关闭使用 try-with-resources（JDK 7+），替代 finally 手动关闭
- `equals` 使用常量或确定非空的对象调用（`"test".equals(param)`）或 `Objects.equals(a, b)`，防 NPE
- 循环内字符串拼接使用 `StringBuilder.append`（每次 `s += x` 都会新建 StringBuilder）
- Map 遍历使用 `entrySet` 或 `Map.forEach`（JDK 8），避免 `keySet` 二次取值

## References（完整代码示例）

- `references/style-examples.md` — 注释、导入顺序、代码格式、枚举常量、异常、日志、工具类完整示例
