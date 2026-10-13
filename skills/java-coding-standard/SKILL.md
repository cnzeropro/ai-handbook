---
name: "java-coding-standard"
description: "Java 语言级编码规范，按 LTS 版本（Java 7 及以下遗留 / 8 / 11 / 17 / 21 / 25）组织最佳实践：命名、注释、格式布局、常量枚举、异常错误码、日志、工具类与版本特性。当编写、重构或审查 Java 代码、按项目 Java 版本选择写法时使用。凡涉及 Java 代码编写与修改的任务均应主动使用。"
---

# Java 编码规范

本 skill 按 Java LTS 版本组织 Java 语言级编码最佳实践，适用于任意 Java 代码；功能模块相关规范（分层架构、对象模型、模块命名等）见 `java-feature-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按项目 Java 版本与主题按需加载。
>
> 示例说明：references/ 中的示例以占位业务名（Model 等）演示语法与结构，实际开发时替换为项目自身的业务名。

## 何时使用

- 确定项目 Java 版本后，选择该版本可用的最佳实践写法
- 编写、重构或审查任意 Java 代码（命名、注释、格式、常量枚举、异常、日志、工具类）
- 维护 Java 8 以下遗留代码，或将旧代码升级到更高版本写法
- 代码审查时作为编码规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 任意 Java 代码（Controller、Service、工具类、脚本等），不限架构与框架
- Java 7 及以下（遗留）至 Java 25 全部版本，按项目实际版本选用（见“三、各 LTS 版本最佳实践速查”）

### 不适用（边界）

- 分层架构、对象模型（PO(Persistent)/DTO/PO(Param)/VO）、模块命名（包/类/方法/PO(Param)/VO）等功能模块规范（参见 `java-feature-standard` skill）
- 接口方法排序（参见 `java-method-ordering` skill）
- 数据库表设计与 SQL 脚本（参见 `db-design-standard` skill）
- 代码安全审查流程（参见 `java-code-review` skill）

## 一、版本判定与总则

1. **以项目声明的 Java 版本为准**（`pom.xml` 的 `java.version` / `maven.compiler.source` / `build.gradle` 的 `sourceCompatibility`）
2. **禁止越级**：只使用该版本及以下支持的语言特性，禁止使用更高版本语法（否则编译失败）
3. **用足当前版本**：在该版本约束内，主动采用该版本可用的最佳实践写法，不写低版本冗长等价物
4. **不用预览特性**：preview 特性默认不使用，除非项目明确开启 `--enable-preview` 并接受变更风险
5. 新项目无特殊约束时，建议选用最新 LTS（撰写本文时为 Java 25）

**正反例（假设项目为 Java 17）：**

```java
// ❌ 项目已支持 Java 17，仍使用低版本的冗长写法
@Data
@AllArgsConstructor
public class ModelDTO {
    private Long id;
    private String name;
}

// ✅ 使用 Java 17 record
public record ModelDTO(Long id, String name) {
}
```

**正反例（假设项目为 Java 8）：**

```java
// ❌ 使用了高于项目版本的语法，编译失败
public record ModelDTO(Long id) {}        // record 需 Java 16+
List<String> list = List.of("a", "b");    // List.of 需 Java 9+

// ✅ Java 8 语法
public class ModelDTO {
    private final Long id;
    // ...
}
List<String> list = Arrays.asList("a", "b");
```

## 二、通用规则精要（各版本通用）

完整规则见 `references/general-quality.md`，完整代码示例见 `references/style-examples.md`：

- 命名：类 UpperCamelCase、方法/变量 camelCase、常量全大写下划线分隔；禁拼音与中文混用；`Abstract`/`Exception`/`Test` 命名约定；用 `blockList`/`allowList`
- 注释：中文为主；类与方法必须 Javadoc（`@author` + `@date`，格式 yyyy/MM/dd）；独立成行禁止尾随；TODO/FIXME 注明标记人与时间
- 导入：JDK → 第三方 → 项目内部三组依次，组间空行
- 格式与布局：4 空格缩进、大括号不换行、单行 120 字符；类成员布局：字段 → 构造器 → 公有/保护方法 → 私有方法 → getter/setter；重载相邻
- 常量与枚举：按功能分类常量类、禁魔法值；枚举值全大写、英文字符串值（第一设计）、注释含取值说明
- 异常：不吞异常、不返回 null 掩盖；记录与抛出择一（全局处理器统一记录）；异常信息英文；`Optional` 仅作返回值；对外接口错误码（5 位 A/B/C 示例方案）
- 日志：`@Slf4j` + 英文 + 占位符 + 上下文与堆栈；级别语义正确；敏感信息不入日志
- 工具类：JDK 与 Spring 自带优先（零额外依赖），Hutool/Guava/Commons 按需补充、功能重叠二选一；JSON 用 Jackson
- 控制语句：if-else 嵌套 ≤ 3 层、卫语句优先、避免取反逻辑、赋值不进条件、复杂条件提布尔变量
- 其他细节：`final` 四类场景、可变参数置最后且少用、集合指定初始容量、diamond 语法、try-with-resources、`equals` 常量在前或 `Objects.equals`、循环内 `StringBuilder`、`entrySet` 遍历

## 三、各 LTS 版本最佳实践速查

高版本特性覆盖低版本：项目为 Java 21 时，同时适用此前所有版本的规范。

| 版本 | 解锁的关键特性与最佳实践 | 详细规范 |
| --- | --- | --- |
| Java 7 及以下（遗留） | 匿名内部类、显式循环、`Date`/`Calendar`（仅维护遗留代码时使用） | `references/java-legacy.md` |
| Java 8（LTS） | Lambda、方法引用、Stream、Optional、`java.time`、接口 default 方法、CompletableFuture（通用规范示例基线） | `references/java-8.md` |
| Java 11（LTS） | `var` 局部推断、`List.of` 等不可变集合工厂、String 增强（isBlank/strip/lines/repeat）、`Optional.isEmpty`、标准 HttpClient | `references/java-11.md` |
| Java 17（LTS） | record、sealed、switch 表达式、文本块、`instanceof` 模式匹配、`Stream.toList()` | `references/java-17.md` |
| Java 21（LTS） | record pattern、switch 模式匹配（`when` 守卫）、虚拟线程、SequencedCollection | `references/java-21.md` |
| Java 25（LTS） | 未命名变量 `_`、Markdown 文档注释、Stream Gatherers、紧凑源文件与实例 main、模块导入声明、灵活构造器体、Scoped Values | `references/java-25.md` |

## References（完整规范与代码示例）

- `references/general-quality.md` — 通用质量规范：命名、注释、导入顺序、格式与类成员布局、常量与枚举、异常与错误码、日志、工具类、控制语句、其他编码细节
- `references/style-examples.md` — 通用规范完整代码示例（注释、导入、格式、枚举常量、异常、日志、工具类）与常见正反例速查（循环拼接、equals/==、遍历删除、吞异常、资源关闭等高频问题）
- `references/java-legacy.md` — Java 7 及以下遗留环境：写法对照表、SimpleDateFormat 等典型坑、最小侵入原则
- `references/java-8.md` — Java 8：Lambda 与方法引用、Stream、Optional、日期时间 API、接口与集合增强
- `references/java-11.md` — Java 11：var、不可变集合工厂、Stream/Optional 增强、String 增强、HttpClient
- `references/java-17.md` — Java 17：record、sealed、switch 表达式、文本块、instanceof 模式匹配、Stream.toList()
- `references/java-21.md` — Java 21：record pattern、switch 模式匹配、虚拟线程、SequencedCollection
- `references/java-25.md` — Java 25：未命名变量、Markdown 文档注释、Stream Gatherers、紧凑源文件、模块导入、灵活构造器体、Scoped Values
