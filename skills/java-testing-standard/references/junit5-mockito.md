# JUnit 5 + Mockito 写法规约（当前基线）

> 本文件为 **JUnit 5 + Mockito + AssertJ**（当前基线组合）的写法规约，须与版本通用的 `general-quality.md` 配合使用。基线：JUnit 5.10+ / Mockito 5.x / AssertJ 3.24+。JUnit 4 遗留见 `junit-legacy.md`。

## 目录

- 一、注解与生命周期 —— @Test、@BeforeEach/@AfterEach、@BeforeAll 静态约束、@Nested、@DisplayName、@Disabled
- 二、参数化规约 —— @ParameterizedTest 与各 Source
- 三、AssertJ 断言规约 —— 统一链式入口、异常断言、软断言、禁混用
- 四、Mockito 桩与验证 —— @Mock、构造注入被测对象、匹配器禁忌、verify 适度、ArgumentCaptor
- 五、常见陷阱速查

## 一、注解与生命周期

- `@Test` 方法**无需 public**（包级可见即可）；JEP 语法细节随版本
- `@BeforeEach` / `@AfterEach` 每用例执行（准备与清理）；`@BeforeAll` / `@AfterAll` 仅执行一次且必须 `static`（或 `@TestInstance(Lifecycle.PER_CLASS)`）——只放昂贵一次性资源（容器启动、文件句柄）
- **`@Nested` 内层类表达场景分组**（Given-When-Then），内层可访问外层字段：

```java
@DisplayName("提款服务")
class WithdrawServiceTest {
    @Nested
    @DisplayName("当账户余额不足")
    class 余额不足 {
        @Test
        @DisplayName("扣款失败并抛出余额不足异常")
        void 扣款失败() { … }
    }
}
```

- `@DisplayName` 表达"条件 + 预期"（中文直读）；`@Disabled` 必须注明原因与 issue 链接，禁裸禁用
- 生命周期方法保持简单：只编排具名方法调用

## 二、参数化规约

多组输入输出用 `@ParameterizedTest` 表驱动，禁复制粘贴近似用例：

```java
// ✅ 单参数
@ParameterizedTest
@ValueSource(strings = {"", "   ", "\t"})
@DisplayName("空白输入抛出参数异常")
void 空白输入(String input) {
    assertThatThrownBy(() -> validator.check(input))
        .isInstanceOf(IllegalArgumentException.class);
}

// ✅ 多参数表：@CsvSource
@ParameterizedTest(name = "{0} + {1} = {2}")
@CsvSource({ "1, 1, 2", "0, 5, 5", "-1, 1, 0" })
void 求和(int a, int b, int expected) {
    assertThat(calc.add(a, b)).isEqualTo(expected);
}

// ✅ 复杂对象：@MethodSource 静态工厂
static Stream<Arguments> 非法模型() {
    return Stream.of(
        Arguments.of(model(m -> m.name(""))),
        Arguments.of(model(m -> m.status(null)))
    );
}
```

- 枚举全遍历用 `@EnumSource`；每个参数化用例自动获得独立生命周期

## 三、AssertJ 断言规约

- **统一 AssertJ 链式断言**（`assertThat` 入口）；禁 JUnit `assertEquals` / `assertTrue` 与 AssertJ 混用（错误输出与可读性两套）

```java
// ❌ JUnit 断言，失败信息差
assertEquals(2, result.size());
assertTrue(result.contains(model));

// ✅ AssertJ：一条链表达完整意图
assertThat(result)
    .hasSize(2)
    .contains(model);
```

常用断言链：

```java
assertThat(models)
    .hasSize(2)
    .extracting(Model::getName)
    .containsExactly("Alpha", "Beta");                     // 顺序敏感
    // containsExactlyInAnyOrder("Beta", "Alpha");          // 顺序不敏感

// 值对象递归比较（替代逐字段 equals）
assertThat(actual).usingRecursiveComparison().isEqualTo(expected);

// 从对象提取后断言
assertThat(result).returns(1L, Model::getId);
```

异常断言：

```java
// ❌ try-catch 手工断言异常
// ✅
assertThatThrownBy(() -> service.withdraw(account, amount))
    .isInstanceOf(InsufficientBalanceException.class)
    .hasMessageContaining("余额不足")
    .hasMessageEndingWith("请先充值");

// ✅ 断言"不抛异常"
assertThatCode(() -> service.close(account)).doesNotThrowAnyException();
```

- 浮点用 `isCloseTo`（指定精度），禁 `isEqualTo` 直接比较
- 多字段 DTO 全量校验用 `SoftAssertions`（一次汇报全部差异，而非逐个失败即停）

## 四、Mockito 桩与验证

### 1. 声明与被测对象构造

```ts
@ExtendWith(MockitoExtension.class)
class WithdrawServiceTest {
    @Mock private AccountRepository accountRepository;
    @Mock private NotificationClient notificationClient;

    // ✅ 被测对象用构造注入真实组装（优于 @InjectMocks 反射注入：重构安全、依赖显式）
    private final WithdrawService service =
        new WithdrawService(accountRepository, notificationClient);
}
```

### 2. 桩（when-thenReturn）

```java
// ✅
when(accountRepository.findById(1L)).thenReturn(Optional.of(account));
when(accountRepository.findById(anyLong())).thenReturn(Optional.empty());

// ✅ 连续调用返回不同值（第二次起取最后一个）
when(client.send(any())).thenReturn(true).thenReturn(false);

// ✅ 抛异常桩
when(repository.save(any())).thenThrow(new DataAccessException("…") {});
```

- **匹配器混用禁忌**：同一调用的参数要么全用匹配器（`any()` / `eq()`），要么全用实际值——混用直接抛 `InvalidUseOfMatchersException`：

```java
// ❌
when(repo.findByNameAndStatus("Alpha", any(Status.class)));

// ✅
when(repo.findByNameAndStatus(eq("Alpha"), any(Status.class)));
```

- 未使用桩在 `MockitoExtension` 下默认严格模式报错（`UnnecessaryStubbingException`）——这是保护；确需宽松用 `lenient()` 并注释

### 3. verify 适度

- 只对**命令型关键交互**做验证（发通知、写审计、调外部系统）；查询型调用（返回值被使用）不断言"调了几次"

```java
// ✅ 命令型交互验证（次数 + 参数）
verify(notificationClient, times(1)).send(eq(accountId), captor.capture());
```

### 4. ArgumentCaptor 捕获验证

```java
// ✅ 捕获入参断言关键字段
verify(notificationClient).send(eq(accountId), payloadCaptor.capture());
assertThat(payloadCaptor.getValue().getMessage()).contains("余额不足");
```

### 5. 其他

- Mockito 5 默认支持 mock final 类与方法（无需 mockito-inline 配置）
- `mockStatic` 慎用（需 try-with-resources 限定作用域且测试串行），仅限无法注入的静态边界
- `lenient()` 慎用：未使用桩报错是保护机制，放宽必须注释原因

## 五、常见陷阱速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 匹配器混用 | `when(m.find("a", any()))` | `when(m.find(eq("a"), any()))` |
| 查询型调用过度 verify | `verify(repo, times(2)).findById(1L)` | 断言返回结果即可 |
| 裸 @Disabled | `@Disabled` 无说明 | `@Disabled("issue-123 待修复")` |
| JUnit/AssertJ 混用 | `assertEquals` + `assertThat` | 统一 AssertJ |
| 浮点 isEqualTo | `assertThat(0.3).isEqualTo(0.1 + 0.2)` | `isCloseTo(0.3, within(1e-9))` |
| @BeforeAll 非 static 报错 | 实例方法 | `static` 或 `@TestInstance(PER_CLASS)` |
| mockStatic 泄漏 | 静态 mock 未关闭 | try-with-resources 包裹 |
| 长逻辑在 @BeforeEach | 钩子内展开 | 编排具名方法 |
