# Java 25（LTS）规范

> 适用：项目构建目标达到 Java 25 时使用。覆盖 Java 22 ~ 25 增量；须同时适用此前全部分册与通用规范。

## 一、未命名变量与模式（Java 22）

- 不使用的变量、异常参数、Lambda 参数、模式变量统一用 `_` 占位，替代随意命名（`e` / `unused` / `arg0`）：

```java
// ❌ 绑定了从未使用的变量
int sides = switch (shape) {
    case Circle c -> 1;    // c 未使用
    case Rect r -> 4;      // r 未使用
};

// ✅ 未命名变量占位
int sides = switch (shape) {
    case Circle _ -> 1;
    case Rect _ -> 4;
};

// ❌ 解构时声明了不使用的组件
if (obj instanceof Point(int x, int y)) {
    use(x);                // y 未使用
}

// ✅ 用 _ 忽略不需要的组件
if (obj instanceof Point(int x, _)) {
    use(x);
}
```

> 注意：捕获到的异常对象**不得**用 `_` 丢弃——要么按日志规范携带堆栈记录（`log.error("...", e)`），要么按“记录与抛出择一”包装重抛（见 `general-quality.md`“六、异常处理规范”）。`_` 只适用于真正无需使用的变量。

## 二、Markdown 文档注释（Java 23）

- Javadoc 可用 `///` Markdown 风格书写；**同一项目二选一**（传统 `/** */` 或 Markdown），不混用
- 无既有约定时沿用传统 `/** */`——与通用注释规范及主流工具链兼容性最好

## 三、Stream Gatherers（Java 24）

- 定制中间操作用 `Stream.gather(Gatherer)` 表达（滑动窗口、去重、自定义聚合等），替代“collect 后循环再包回 Stream”的断裂写法
- 自定义 Gatherer 复杂度过高时，仍优先拆分具名方法保证可读性

## 四、紧凑源文件与实例 main 方法（Java 25）

- `void main()` 无类声明的紧凑源文件，适用于脚本、一次性工具、原型验证
- 生产应用与类库仍用标准类结构（包声明、类、Javadoc），不因语法允许而放开

```java
// 一次性工具 / 原型
void main() {
    IO.println("done");
}
```

## 五、模块导入声明（Java 25）

- `import module java.base;` 按模块批量导入，适合紧凑源文件与原型
- 生产代码优先显式单类型导入（与通用规范导入顺序一致），避免隐式依赖面过大

## 六、灵活构造器体（Java 25）

- 允许在 `super(...)` / `this(...)` 之前执行语句：用于参数校验与派生计算，把非法状态挡在父类构造之前
- 不得在构造完成前泄漏 `this`（注册回调、启动线程、把 this 传给外部方法）

```java
public ModelView(Model model) {
    // super 前校验，非法参数直接失败
    Objects.requireNonNull(model, "model must not be null");
    super(model.id(), model.name());
}
```

## 七、Scoped Values（Java 25）

- 替代 `ThreadLocal` 传递请求级上下文（traceId、租户、当前用户）：不可变、有界生命周期、与虚拟线程亲和
- 用法：`ScopedValue.where(CONTEXT, ctx).run(task)`；虚拟线程场景尤其适用（见 `java-21.md`）
- Structured Concurrency 仍为预览特性，生产暂不使用（见下）

## 八、预览特性

- primitive type patterns（JEP 507）、Structured Concurrency（JEP 505）等仍处预览：默认不使用，除非项目明确开启 `--enable-preview` 并接受变更风险（与 `SKILL.md`“一、版本判定与总则”第 4 条一致）
