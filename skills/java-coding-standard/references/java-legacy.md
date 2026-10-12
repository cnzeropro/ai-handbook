# Java 7 及以下（遗留环境）规范

> 适用：构建目标为 Java 7 及以下的遗留项目。仅在维护遗留代码或项目明确要求时使用；新代码应使用更高版本（见 `java-8.md` 及后续 LTS 分册）。Java 8 引入的 Lambda、Stream、Optional、`java.time` 等在本环境不可用。

## 一、Java 8+ → Java 7 写法对照表

| Java 8+ 写法 | Java 7 等价写法 |
| --- | --- |
| Lambda 表达式 | 匿名内部类 |
| 方法引用 `Model::getName` | 匿名内部类 |
| Stream 管道（map/filter/collect） | 显式循环 + 集合操作 |
| `Optional` 返回值 | null 判空 + 方法命名区分语义（`findXxx` 可返回 null，`getXxx` 不应返回 null） |
| `java.time`（LocalDateTime 等） | `Date` / `Calendar`（注意线程安全与可变性，见下） |
| `CompletableFuture` | `ExecutorService` + `Future.get` 或回调接口 |
| 接口 `default` / `static` 方法 | 配套工具类或抽象基类 |
| `Collectors.groupingBy` 分组 | 循环 + `Map<K, List<V>>` 手工分组 |
| `List.sort(comparator)` | `Collections.sort(list, comparator)` |

## 二、Java 7 已有能力（正常使用，勿倒退）

以下特性 Java 7 起可用，遗留项目中直接使用：

- 泛型 diamond 语法：`new HashMap<>()`
- try-with-resources 自动关闭资源
- `switch` 支持 `String`
- 多异常捕获：`catch (IOException | SQLException e)`
- 字面量改进：二进制 `0b` 前缀、数值下划线 `1_000_000`
- Java 5+ 能力：泛型、枚举、注解、增强 for、可变参数

> 若目标为 Java 6 及以下：上述 Java 7 特性亦不可用（无 diamond、无 try-with-resources、无 String switch、无 multi-catch），需逐项回退。

## 三、遗留代码典型坑

### 1. SimpleDateFormat 线程不安全

共享实例并发解析会互相错乱，必须 `ThreadLocal` 封装或每次新建：

```java
// ❌ 共享常量实例
private static final SimpleDateFormat FORMAT = new SimpleDateFormat("yyyy-MM-dd");

// ✅ ThreadLocal 封装（Java 7 写法）
private static final ThreadLocal<SimpleDateFormat> FORMAT =
    new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd");
        }
    };
```

### 2. Date / Calendar 可变性

`Date` 是可变对象，作为入参或返回值被外部修改会引发隐蔽 bug；跨边界传递用防御性拷贝：`new Date(date.getTime())`。

### 3. 手写分组 / 去重易错

循环分组注意 null key、大小写归一等边界；可评估 Google Guava（`Multimap` / `ImmutableMap`）降低成本——引入依赖须按仓库 `dev-rules.md` 依赖管理执行。

## 四、遗留代码最小侵入

- 修改遗留代码时与周边风格保持一致，不顺手做语法升级，避免无关 diff
- 确需整体升级（如升到 Java 8/11/17），作为独立任务执行，不与功能变更混杂
- 遗留库中的新文件与项目整体语言版本保持一致，不单独越级使用高版本语法
