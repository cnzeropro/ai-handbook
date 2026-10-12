# Java 21（LTS）规范

> 适用：项目构建目标达到 Java 21 时使用。覆盖 Java 18 ~ 21 增量；须同时适用此前全部分册与通用规范。

## 一、record pattern 与 switch 模式匹配

- record pattern 解构嵌套数据；switch 模式匹配按类型分派（对 sealed 层次具有穷举检查），替代 if-else 类型判断链
- `when` 守卫表达附加条件，减少嵌套

```java
// ✅
String describe(Shape shape) {
    return switch (shape) {
        case Circle c when c.radius() > 100 -> "large circle";
        case Circle c -> "circle";
        case Rectangle(Point tl, Point br) -> "rectangle";
    };
}
```

## 二、虚拟线程（Java 21）

- 适用：高并发 IO 密集任务（HTTP 调用、DB 访问）；"每任务一线程"模型，创建成本极低
- 用 `Executors.newVirtualThreadPerTaskExecutor()` 或 `Thread.ofVirtual().start(...)`；**不要池化虚拟线程**
- 注意：`synchronized` 块内的阻塞 IO 会导致载体线程 pin，热点路径改用 `ReentrantLock` 或缩小同步块
- `ThreadLocal` 在海量虚拟线程下内存放大，慎用；跨任务传递上下文优先 Scoped Values（Java 25 正式，见 `java-25.md`）
- CPU 密集任务仍用固定大小的平台线程池

## 三、SequencedCollection（Java 21）

- `List` / `Deque` 等序列集合统一提供 `getFirst` / `getLast` / `reversed`，替代手工下标访问

```java
// ❌
Model last = models.get(models.size() - 1);

// ✅
Model last = models.getLast();
```
