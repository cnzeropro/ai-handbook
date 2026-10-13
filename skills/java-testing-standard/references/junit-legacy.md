# JUnit 4 及以下（遗留环境）规范

> 适用：JUnit 4.x 遗留项目。仅在维护既有代码或项目明确要求时使用；新测试一律 JUnit 5（见 `junit5-mockito.md`）。同一项目保持一代 JUnit，不混用两代注解。

## 一、写法基线（JUnit 4）

- 注解在 `org.junit` 包：`@Test` / `@Before` / `@After` / `@BeforeClass`（static）/ `@AfterClass`（static）/ `@Ignore`
- `@Test` 方法必须 public；预期异常用 `@Test(expected = X.class)`（只能验类型，不能验消息）或 `@Rule ExpectedException`
- 参数化用 `@RunWith(Parameterized.class)` + `@Parameterized` 标注的构造器注入（写法笨重，能用 `@CsvSource` 思路在 JUnit 5 中对应的场景，迁移收益最大）
- 断言用 Hamcrest `assertThat` + `org.hamcrest.Matchers`，或 JUnit `assertEquals`
- 测试规则 `@Rule` / `@ClassRule`（如 `TemporaryFolder`、`ExpectedException`）
- 集成 Spring 用 `@RunWith(SpringRunner.class)`

```java
@RunWith(Parameterized.class)
public class AmountValidationTest {
    @Parameterized.Parameter
    public String input;

    @Parameterized.Parameters(name = "{0}")
    public static Collection<String> inputs() {
        return Arrays.asList("", "  ", "abc");
    }

    @Test(expected = IllegalArgumentException.class)
    public void 非法输入应抛异常() {
        validator.check(input);
    }
}
```

## 二、JUnit 4 → JUnit 5 对照（升级参考）

| JUnit 4 | JUnit 5 |
| --- | --- |
| `org.junit.Test` | `org.junit.jupiter.api.Test` |
| `@Before` / `@After` | `@BeforeEach` / `@AfterEach` |
| `@BeforeClass` / `@AfterClass` | `@BeforeAll` / `@AfterAll` |
| `@Ignore("原因")` | `@Disabled("原因")` |
| `@RunWith(Parameterized.class)` + 构造器 | `@ParameterizedTest` + 各 Source |
| `@Test(expected = X.class)` | `assertThrows` / `assertThatThrownBy` |
| `@Rule ExpectedException` | `assertThatThrownBy` |
| Hamcrest `assertThat` | AssertJ `assertThat`（推荐） |
| `@Category` | `@Tag` |
| `@RunWith(SpringRunner.class)` | `@ExtendWith(SpringExtension.class)`（Spring Boot 自动） |
| `@Rule TemporaryFolder` | `@TempDir` |

## 三、遗留代码最小侵入

- **不混用两代注解**：JUnit 5 的 vintage engine 可继续执行旧 JUnit 4 测试，但单个测试类内禁混写
- 不顺手迁移：升级作为独立任务（先引入 `junit-jupiter` 与 `junit-vintage` 引擎并存执行，逐类迁移，全部迁移完成后移除 JUnit 4 依赖）
- 断言库二选一统一（Hamcrest 或 AssertJ），禁同类断言混用
- JUnit 4 项目引入 Mockito 5 注意：Mockito 5 要求 JDK 11+；JUnit 4 老项目如锁 JDK 8，用 Mockito 4.x（`mockito-core` 4.x，final class mock 需 `mockito-inline`）
