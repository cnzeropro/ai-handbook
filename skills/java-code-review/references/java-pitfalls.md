# Java 语言陷阱审查清单

> 本文件为 `../SKILL.md` 的补充检查清单：Java 语言层面的正确性陷阱（bug 高发点），审查时逐类排查。

## 1. 数值与包装类

- 包装类之间值比较使用 `equals` 而非 `==`（`Integer` 在 -128~127 走缓存复用对象，区间外 `==` 结果为 false）
- 浮点数等值判断：基本类型禁用 `==`，包装类型禁用 `equals`（用误差范围或 `BigDecimal`）
- 禁止 `new BigDecimal(double)`（精度损失；用 String 构造或 `BigDecimal.valueOf`）
- `BigDecimal` 等值比较用 `compareTo` 而非 `equals`（equals 比较精度：1.0 与 1.00 不等）
- POJO 属性使用包装类型（数据库查询结果可能为 null，基本类型接收自动拆箱 NPE）
- 三目运算符两端类型不一致时可能自动拆箱 NPE（如 `Integer result = (flag ? a * b : c)`，c 为 null 时拆箱抛 NPE）

## 2. 集合

- 覆写 `equals` 必须覆写 `hashCode`；作为 Set 元素 / Map key 的自定义对象必须覆写两者
- `Collectors.toMap` 无 mergeFunction 时重复 key 抛 `IllegalStateException`；value 为 null 时抛 NPE
- foreach 循环内禁止 remove/add（`ConcurrentModificationException`；用 `Iterator.remove` 或 collect 过滤）
- `subList` 结果不可强转 ArrayList；对父集合增删会导致子列表遍历异常
- `toArray` 使用带参方法 `toArray(new T[0])`（无参返回 `Object[]`，强转抛 `ClassCastException`）
- `Arrays.asList` 返回固定大小视图，add/remove/clear 抛 `UnsupportedOperationException`
- 集合判空用 `isEmpty()` 而非 `size() == 0`
- `split` 结果按索引访问前检查长度（尾部分隔符被丢弃，如 `"a,b,c,,".split(",").length == 3`）

## 3. 日期时间

- 禁止使用 `java.sql.Date` / `java.sql.Time` / `java.sql.Timestamp`（getHours/getYear 抛异常、与 `java.util.Date` 比较存在 JDK BUG）
- 日期格式化年份用小写 `yyyy`（大写 `YYYY` 是"周所在年"，跨年周会多算一年）
- 禁止硬编码一年 365 天（闰年问题；用 `LocalDate.lengthOfYear` 等）

## 4. 并发与锁

- `SimpleDateFormat` 线程不安全，禁止 static 共享（用 `DateTimeFormatter` 或 `ThreadLocal`）
- `ThreadLocal` 使用后必须 `remove()`（线程池复用线程场景内存泄漏）
- 多资源加锁保持一致的加锁顺序（防死锁）
- `lock()` 必须在 try 块之外、加锁与 try 之间无可抛异常调用（防 finally 中解锁未加锁对象）
- `tryLock` 进入业务前必须判断是否持有锁
- 单例获取与其中方法必须线程安全
- 禁止应用内显式创建线程（用线程池）；线程池禁止 `Executors` 创建（用 `ThreadPoolExecutor`，防无界队列 OOM）；线程必须命名
- 并发更新同一记录需加锁或乐观锁（version），避免更新丢失；资金类场景用悲观锁
- 高并发场景避免"等于"判断作为中断/退出条件（等值被击穿，用区间判断）
- 双重检查锁（DCL）的目标字段必须 `volatile`
- `HashMap` 并发写入导致数据不一致（JDK 7 的 resize 死链问题为历史版本），并发场景用 `ConcurrentHashMap`

## 5. 类设计与序列化

- 构造方法内禁止业务逻辑（初始化放 `init` 方法）
- POJO 必须写 `toString`（异常排查时打印属性）
- POJO 禁止同时存在 `isXxx()` 与 `getXxx()`（框架无法确定调用哪个）
- 序列化类新增属性不修改 `serialVersionUID`；完全不兼容升级才修改
- 类成员与方法访问控制从严（private 优先；工具类禁 public 构造）
- 覆写方法必须加 `@Override`（防 `get0bject` / `getObject` 笔误）

## 6. 控制语句

- switch 每个 case 必须终止（continue/break/return）或注释说明贯穿；必须有 default（置于最后）
- `switch(String)` 变量为外部参数时先判 null
- 循环体外可移出的操作不放入循环（对象定义、连接获取、try-catch）

## 7. 其他

- 正则表达式预编译（`Pattern` 定义在常量而非方法体内）
- `Math.random()` 取整数用 `Random.nextInt` / `nextLong`（放大取整存在随机序列偏差与取整边界问题）
- 枚举的属性字段必须私有且不可变
