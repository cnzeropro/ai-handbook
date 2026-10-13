# Java 测试通用质量规范

> 本文件为 java-testing-standard 的**版本通用**质量规范（JUnit 4 与 JUnit 5 通用原则部分；版本特有写法见 `junit5-mockito.md` / `junit-legacy.md`）。来源：JUnit 5 官方用户指南、Spring Boot Testing 官方文档与 Java 社区测试实践。完整正反例见 `style-examples.md`（同目录）。测试代码本身的命名、格式、注释遵循 `java-coding-standard`。

## 目录

- 一、测试策略与分层
- 二、测行为，不测实现
- 三、用例设计（FIRST 与 AAA）
- 四、测试数据
- 五、mock 边界
- 六、稳定性与确定性
- 七、覆盖率的态度

## 一、测试策略与分层

| 层级 | 对象 | 占比（参考） | 特点 |
| --- | --- | --- | --- |
| 单元测试 | 类 / 方法（依赖 mock） | 最多 | 快、稳、定位准 |
| 切片测试 | 单层装配（@WebMvcTest / @DataJpaTest） | 居中 | 只装载该层 Bean |
| 集成测试 | 全上下文 + 真实依赖（Testcontainers） | 最少 | 最接近生产、最慢 |

- E2E / 长流程验证只做**金路径冒烟**，不覆盖全部分支——越往下越慢越脆
- 静态检查（编译器、TS 式约束在 Java 即编译与 lint）能拦住的错误不写测试
- **bug 先写复现测试**：先写暴露缺陷的失败测试，再修复转绿——回归防护随修复沉淀

## 二、测行为，不测实现

- 断言**业务结果**（返回值、状态变化、对外协作），不断言内部私有状态与方法调用细节
- 判断标准："重构实现（不改行为）后，测试必须仍然全绿"——重构要改测试，说明测的是实现

```java
// ❌ 测实现：依赖内部字段与私有方法
assertEquals(1, counter.getClass().getDeclaredFields().length);
assertTrue(ReflectionTestUtils.invokeMethod(counter, "isPositive"));

// ✅ 测行为
assertThat(counter.increment()).isEqualTo(1);
```

- Mock 验证同理：查询型调用（返回值被使用）不断言"调了几次"；只对**命令型关键交互**（发通知、写审计）做 verify

## 三、用例设计（FIRST 与 AAA）

- **FIRST**：Fast（快）/ Independent（独立）/ Repeatable（可重复）/ Self-validating（自校验）/ Timely（及时）
- **AAA**：Arrange（准备）→ Act（执行）→ Assert（断言），空行分段
- 一个 `@Test` 只测一个主题；新场景新用例，禁一个用例串多断言主题
- **用例命名用 `@DisplayName` 表达"条件 + 预期"**（中文直读），方法名保持合法标识符

```java
@Nested
class 当账户余额不足 {
    @Test
    @DisplayName("扣款失败并抛出余额不足异常")
    void 扣款失败() { … }
}
```

- 多组输入输出用参数化表驱动（写法见 `junit5-mockito.md` 二），禁复制粘贴近似用例

## 四、测试数据

- 测试数据用**工厂方法或 Builder** 构造，支持覆盖项；禁在多个用例复制大字面量

```java
// ✅ 工厂方法：默认值 + 用例覆盖
private Model model(Consumer<Model.ModelBuilder> overrides) {
    var builder = Model.builder().id(1L).name("Alpha").status(DRAFT);
    overrides.accept(builder);
    return builder.build();
}
```

- 禁共享可变静态测试数据（用例间污染、执行顺序依赖）
- 测试常量集中定义，与业务常量分开（测试模块内）

## 五、mock 边界

- **只 mock 外部依赖与不可控边界**：网络客户端、时钟、随机源、第三方 SDK；被测类内部逻辑与值对象（DTO / VO / 实体）**禁 mock**
- mock 越多，测试越接近"测试 mock 本身"；一个用例 mock 超过 2~3 个协作者，通常说明被测类职责过重

```java
// ❌ mock 值对象与被测内部
Model model = mock(Model.class);
when(counter.isPositive()).thenReturn(true);

// ✅ 值对象直接构造；只 mock 外部服务
Model model = model(m -> m.name("Alpha"));
counter = new CounterService(modelClient);
```

## 六、稳定性与确定性

- 单元测试**禁真实时间、随机数、网络、文件系统**——不确定性与慢的根源；时钟与随机源通过依赖注入，测试注入固定值
- 时区敏感逻辑显式固定时区；禁依赖默认时区
- 禁依赖执行顺序（JUnit 默认顺序确定性但语义上不应依赖）与 `Thread.sleep` 等待异步结果（用 CountDownLatch / Awaitility 等明确等待条件）

## 七、覆盖率的态度

- 覆盖率是**参考指标**不是目标：无断言跑覆盖率没有意义
- 看"关键业务路径与历史 bug 是否有测试防护"；不为凑数写浅断言测试
- 新代码与 bug 修复必须有对应测试（bug 先写复现测试）
