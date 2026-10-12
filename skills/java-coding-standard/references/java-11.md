# Java 11（LTS）规范

> 适用：项目构建目标达到 Java 11 时使用。覆盖 Java 9 ~ 11 增量；须同时适用 `java-8.md` 与通用规范。高版本特性覆盖低版本。

## 一、var 局部变量类型推断（Java 10）

- 适用：右侧类型一目了然的局部变量
- 禁用：字段、方法参数、方法返回值；右侧为 null / Lambda / 方法引用（无法推断或损失信息）
- 适度使用不滥用：右侧为工厂方法或链式调用中间结果时优先显式类型

```java
// ✅ 右侧类型明确
var models = new ArrayList<Model>();
var name = model.getName();

// ❌ 右侧不明确，可读性受损
var result = service.process();
```

## 二、不可变集合工厂（Java 9）

- `List.of` / `Set.of` / `Map.of` 创建不可变集合，替代 `Arrays.asList`（定长视图）与 `Collections.unmodifiableXxx` 样板
- 注意：不可存 null 元素、不可增删改；需要可变集合时仍用构造器

```java
// ❌
List<String> statuses = Collections.unmodifiableList(Arrays.asList("A", "B"));

// ✅
List<String> statuses = List.of("A", "B");
```

## 三、Stream 与 Optional 增强

- `takeWhile` / `dropWhile`（Java 9）按条件截断流
- `Optional.isEmpty`（Java 11）替代 `!opt.isPresent()`
- `Collectors.toUnmodifiableList` / `toUnmodifiableMap`（Java 10）收集为不可变集合

## 四、String 增强（Java 11）

- `isBlank`（含空白字符判断，区别于 `isEmpty`）
- `strip` / `stripLeading` / `stripTrailing`（Unicode 感知，区别于 `trim`）
- `lines`（按行拆分为流）、`repeat(n)`

## 五、标准 HTTP 客户端（Java 11）

- 用 `java.net.http.HttpClient` 替代手工 `HttpURLConnection`，或替代仅为 HTTP 而引入的第三方依赖
- 支持 HTTP/2，同步 `send` 与异步 `sendAsync` 按场景选用
