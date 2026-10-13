# Java 测试编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：JUnit 5 用户指南、Spring Boot Testing 文档、AssertJ / Mockito 官方文档与社区审查常见问题）。规则正文见 `SKILL.md` 与各分册；本文件供代码审查对照与深入学习。

## 目录

- 1. 标准单元测试完整示例（@Nested + DisplayName + AssertJ + Mockito）
- 2. 参数化完整示例（ValueSource / CsvSource / MethodSource / EnumSource）
- 3. AssertJ 常用断言链速查
- 4. Mockito 正反例（匹配器混用、verify 适度、ArgumentCaptor、lenient）
- 5. MockMvc 切片测试完整示例
- 6. Testcontainers 集成测试完整示例
- 7. JUnit 4 → 5 对照速查
- 8. 常见正反例速查

## 1. 标准单元测试完整示例

```java
@ExtendWith(MockitoExtension.class)
class WithdrawServiceTest {

    @Mock
    private AccountRepository accountRepository;

    @Mock
    private NotificationClient notificationClient;

    // ✅ 被测对象构造注入真实组装
    private final WithdrawService service =
        new WithdrawService(accountRepository, notificationClient);

    @Nested
    @DisplayName("当账户余额充足")
    class 余额充足 {

        @Test
        @DisplayName("扣款成功并返回新余额")
        void 扣款成功() {
            // Arrange
            var account = model(a -> a.balance(100L));
            when(accountRepository.findById(1L)).thenReturn(Optional.of(account));

            // Act
            var result = service.withdraw(1L, 30L);

            // Assert
            assertThat(result.balance()).isEqualTo(70L);
        }
    }

    @Nested
    @DisplayName("当账户不存在")
    class 账户不存在 {

        @Test
        @DisplayName("抛出账户不存在异常且不发送通知")
        void 账户不存在() {
            when(accountRepository.findById(1L)).thenReturn(Optional.empty());

            assertThatThrownBy(() -> service.withdraw(1L, 30L))
                .isInstanceOf(AccountNotFoundException.class);

            verifyNoInteractions(notificationClient);
        }
    }
}
```

## 2. 参数化完整示例

```java
// ✅ 单参数：@ValueSource
@ParameterizedTest
@ValueSource(strings = {"", "   ", "\t"})
@DisplayName("空白输入抛出参数异常")
void 空白输入(String input) { … }

// ✅ 多参数表：@CsvSource（name 属性让失败用例可读）
@ParameterizedTest(name = "{0} + {1} = {2}")
@CsvSource({ "1, 1, 2", "0, 5, 5", "-1, 1, 0" })
void 求和(int a, int b, int expected) {
    assertThat(calc.add(a, b)).isEqualTo(expected);
}

// ✅ 枚举全遍历：@EnumSource
@ParameterizedTest
@EnumSource(ModelStatus.class)
void 各状态均可序列化(ModelStatus status) { … }

// ✅ 复杂对象：@MethodSource 静态工厂
static Stream<Arguments> 非法模型() {
    return Stream.of(
        Arguments.of(model(m -> m.name(""))),
        Arguments.of(model(m -> m.status(null)))
    );
}

@ParameterizedTest
@MethodSource("非法模型")
void 非法模型校验失败(Model model) {
    assertThatThrownBy(() -> validator.check(model))
        .isInstanceOf(ValidationException.class);
}
```

## 3. AssertJ 常用断言链速查

```java
// 集合
assertThat(models)
    .hasSize(2)
    .extracting(Model::getName)
    .containsExactly("Alpha", "Beta")                       // 顺序敏感
    .containsExactlyInAnyOrder("Beta", "Alpha");            // 顺序不敏感

// 值对象递归比较（替代逐字段断言）
assertThat(actual).usingRecursiveComparison().isEqualTo(expected);

// 从对象提取后断言
assertThat(result).returns(1L, Model::getId);

// 浮点精度
assertThat(0.1 + 0.2).isCloseTo(0.3, within(1e-9));

// 软断言：一次汇报全部差异（DTO 全量校验）
SoftAssertions.assertSoftly(softly -> {
    softly.assertThat(dto.getName()).isEqualTo("Alpha");
    softly.assertThat(dto.getStatus()).isEqualTo(ModelStatus.DRAFT);
    softly.assertThat(dto.getTags()).isEmpty();
});
```

## 4. Mockito 正反例（匹配器混用、verify 适度、ArgumentCaptor、lenient）

匹配器混用禁忌：

```java
// ❌ 实际值与匹配器混用：InvalidUseOfMatchersException
when(repo.findByNameAndStatus("Alpha", any(Status.class)));

// ✅
when(repo.findByNameAndStatus(eq("Alpha"), any(Status.class)));
```

verify 适度：

```java
// ❌ 查询型调用过度 verify（返回值已被断言，再数次数是测实现）
verify(repo, times(2)).findById(1L);

// ✅ 命令型关键交互才 verify（次数 + 参数）
verify(notificationClient, times(1)).send(eq(accountId), payloadCaptor.capture());
```

lenient 慎用：

```java
// ❌ 全局放宽：未使用桩的报错是保护机制
@MockitoSettings(strictness = Strictness.LENIENT)

// ✅ 仅个别桩放宽并注释原因
lenient().when(config.isEnabled()).thenReturn(true);   // 部分用例不触发该配置
```

## 5. MockMvc 切片测试完整示例

```java
@WebMvcTest(ModelController.class)
class ModelControllerTest {

    @Autowired MockMvc mockMvc;

    @MockitoBean ModelService modelService;   // Boot 3.4+；3.3 及以前用 @MockBean

    @Test
    void 创建模型返回201与模型体() throws Exception {
        when(modelService.create(any())).thenReturn(
            new ModelInfo(1L, "Alpha", ModelStatus.DRAFT));

        mockMvc.perform(post("/api/models")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name": "Alpha"}
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").isNumber())
            .andExpect(jsonPath("$.name").value("Alpha"))
            .andExpect(jsonPath("$.status").value("DRAFT"));
    }

    @Test
    void 参数缺失返回400并含字段提示() throws Exception {
        mockMvc.perform(post("/api/models")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{}"))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors[0].field").value("name"));
    }
}
```

- `jsonPath` 断言具体值与类型，禁只断言状态码；业务逻辑在 Controller 出现时下沉 Service

## 6. Testcontainers 集成测试完整示例

```java
@SpringBootTest
@Testcontainers
class ModelIntegrationTest {

    // ✅ 静态单例容器：全测试类共享，避免每用例重启
    @Container
    @ServiceConnection                        // Boot 3.1+ 自动注入连接属性
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0");

    @Autowired ModelRepository modelRepository;

    @Test
    @DisplayName("保存并按名称查询")
    void 保存并按名称查询() {
        modelRepository.save(Model.builder().name("Alpha").build());
        assertThat(modelRepository.findByName("Alpha")).isPresent();
    }
}
```

无 `@ServiceConnection` 的中间件用 `@DynamicPropertySource`：

```java
@Container
static GenericContainer<?> redis = new GenericContainer<>(DockerImageName.parse("redis:7"))
    .withExposedPorts(6379);

@DynamicPropertySource
static void redisProps(DynamicPropertyRegistry registry) {
    registry.add("spring.data.redis.host", redis::getHost);
    registry.add("spring.data.redis.port", () -> redis.getMappedPort(6379));
}
```

- 容器镜像版本与生产对齐；本机无 Docker 时的跳转策略按项目 CI 约定

## 7. JUnit 4 → 5 对照速查

| JUnit 4 | JUnit 5 |
| --- | --- |
| `org.junit.Test` | `org.junit.jupiter.api.Test` |
| `@Before` / `@After` | `@BeforeEach` / `@AfterEach` |
| `@BeforeClass` / `@AfterClass`（static） | `@BeforeAll` / `@AfterAll` |
| `@Ignore("原因")` | `@Disabled("原因")` |
| `@RunWith(Parameterized.class)` + 构造器 | `@ParameterizedTest` + 各 Source |
| `@Test(expected = X.class)` | `assertThrows` / `assertThatThrownBy` |
| `@Rule ExpectedException` | `assertThatThrownBy` |
| Hamcrest `assertThat` | AssertJ `assertThat` |
| `@Category` | `@Tag` |
| `@RunWith(SpringRunner.class)` | `@ExtendWith(SpringExtension.class)`（Boot 自动） |
| `@Rule TemporaryFolder` | `@TempDir` |

## 8. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 匹配器混用 | `find("a", any())` | `find(eq("a"), any())` |
| 查询型过度 verify | `verify(repo, times(2)).findById` | 断言返回结果即可 |
| 裸 @Disabled | `@Disabled` | `@Disabled("issue-123 原因")` |
| 断言库混用 | `assertEquals` + `assertThat` | 统一 AssertJ |
| 浮点 isEqualTo | `0.1 + 0.2 == 0.3` | `isCloseTo(0.3, within(1e-9))` |
| mock 值对象 | `mock(Model.class)` | 直接构造（工厂/Builder） |
| @InjectMocks 反射注入 | 复杂依赖注入失败静默 | 构造注入组装被测对象 |
| mockStatic 泄漏 | 静态 mock 未关闭 | try-with-resources 限定 |
| lenient 放纵 | 全局 LENIENT | 个别桩 `lenient().when` + 注释 |
| 钩子内长逻辑 | ngOnInit 式展开 | 编排具名方法 |
| 断言库为 Hamcrest | `assertThat(x, is(1))` | 迁移 AssertJ |
| 用例相互依赖 | 共享可变静态数据 | 工厂方法独立造数 |
