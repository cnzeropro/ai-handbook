# Java 8（LTS）规范

> Java 8 为本 skill 通用规范的示例基线：`general-quality.md` 中的示例默认以 Java 8 语法呈现。项目为 Java 8 时，遵循通用规范与本文件；更高版本另见 `java-11.md` / `java-17.md` / `java-21.md` / `java-25.md`。

## 一、Lambda 与方法引用

- 简短逻辑用 Lambda；仅调用既有方法时优先方法引用（`Model::getName`），可读性更好
- 优先使用标准函数式接口（`Function` / `Predicate` / `Consumer` / `Supplier` / `BiFunction`），不随意自定义
- Lambda 体超过 3~5 行或需要复用时，提取为具名方法再以方法引用传入

```java
// ❌ 冗长匿名类
list.sort(new Comparator<Model>() {
    public int compare(Model a, Model b) {
        return a.getName().compareTo(b.getName());
    }
});

// ✅
list.sort(Comparator.comparing(Model::getName));
```

## 二、Stream

- Stream 用于声明式转换：分清中间操作（`map` / `filter` / `sorted`）与终结操作（`collect` / `forEach` / `reduce`）；一个 Stream 只消费一次
- 不在 Stream 中产生副作用（修改外部变量或集合），结果一律 `collect` 收集
- 多级嵌套 Stream 影响可读性时，拆分为具名方法或退回普通循环
- `parallelStream` 慎用：仅适用于无共享可变状态、数据量大的 CPU 密集任务，引入前压测验证收益

```java
// ❌ 副作用收集
List<String> names = new ArrayList<>();
models.stream().forEach(m -> names.add(m.getName()));

// ✅
List<String> names = models.stream()
    .map(Model::getName)
    .collect(Collectors.toList());
```

## 三、Optional 与 null 处理

- 详见通用规范 `general-quality.md`"六、异常处理规范 → Optional 使用规范"：仅用于方法返回值、`map` / `flatMap` / `orElseThrow` 链式调用、禁返回 null 的 Optional

## 四、日期时间 API（java.time）

- 用 `LocalDate` / `LocalDateTime` / `Instant` + `DateTimeFormatter` 替代 `Date` / `Calendar` / `SimpleDateFormat`（新 API 不可变且线程安全）
- 日期运算用 `plusDays` / `minusMonths` 等语义化方法，不手工换算毫秒

```java
// ❌
SimpleDateFormat fmt = new SimpleDateFormat("yyyy-MM-dd");
Date start = fmt.parse("2025-01-01");

// ✅
LocalDate start = LocalDate.parse("2025-01-01");
```

## 五、接口与集合增强

- 接口 `default` / `static` 方法用于接口演进；复杂逻辑仍放实现类或工具类
- `Map.computeIfAbsent` / `computeIfPresent` / `merge` 替代手工 containsKey-put 模式
- `CompletableFuture` 组合异步任务（`thenApply` / `thenCombine` / `exceptionally`），替代手工 Future 轮询
