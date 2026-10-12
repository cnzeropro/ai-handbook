# Java 17（LTS）规范

> 适用：项目构建目标达到 Java 17 时使用。覆盖 Java 12 ~ 17 增量；须同时适用此前全部分册与通用规范。正反例对照见 `SKILL.md`“一、版本判定与总则”。

## 一、record（Java 16）

- 用于不可变数据载体：DTO/VO、多返回值、聚合键；替代只有字段、构造器、getter、equals/hashCode/toString 的简单 `@Data` 类
- 约束：隐式 final 不可继承、组件不可变（集合组件注意防御性拷贝）、不承载复杂业务行为的领域对象

```java
// ❌ 项目已支持 Java 17，仍使用低版本的冗长写法
@Data
@AllArgsConstructor
public class ModelDTO {
    private Long id;
    private String name;
}

// ✅
public record ModelDTO(Long id, String name) {
}
```

## 二、sealed 密封类（Java 17）

- 需要封闭继承层次（领域状态、解析结果、事件类型）时用 `sealed` + `permits` 显式列出子类，配合 switch 模式匹配获得穷举检查
- 面向外部扩展的框架钩子、SPI 不适用 sealed

```java
public sealed interface ParseResult permits Success, Failure {
}

public record Success(String value) implements ParseResult {
}

public record Failure(String reason) implements ParseResult {
}
```

## 三、switch 表达式（Java 14）

- 箭头语法无 fall-through、多标签合并；有返回值时用 `yield`
- 对 enum / sealed 的分派具有穷举检查，替代冗长 if-else 链

```java
// ❌
int len;
switch (name) {
    case "a":
        len = 1;
        break;
    default:
        len = 0;
        break;
}

// ✅
int len = switch (name) {
    case "a" -> 1;
    default -> 0;
};
```

## 四、文本块（Java 15）

- SQL、JSON、HTML 等多行文本用文本块 `"""`；结尾定界符的缩进决定公共前缀剔除
- 含占位符时配合 `formatted`（或 `String.format`）

```java
String sql = """
    SELECT id, name
    FROM model
    WHERE status = '%s'
    """.formatted(status);
```

## 五、其他

- `instanceof` 模式匹配（Java 16）：`if (obj instanceof String s)` 替代“判断 + 强转”两步写法
- `Stream.toList()`（Java 16）替代 `collect(Collectors.toList())`；注意返回**不可变**列表，调用方需要可变列表时仍用 `collect(Collectors.toCollection(ArrayList::new))`
- Helpful NullPointerException（Java 15 起默认开启）：可依赖精确 NPE 提示定位链式调用断点，但仍应按通用规范用 `Optional` 减少链式 NPE
