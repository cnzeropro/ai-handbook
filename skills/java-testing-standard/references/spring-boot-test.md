# Spring Boot 测试规范

> 本文件为 **Spring Boot 3.x** 测试写法规约，须与版本通用的 `general-quality.md` 与 `junit5-mockito.md` 配合使用。基线 Spring Boot 3.x（3.4 起 `@MockBean` 废弃 → `@MockitoBean`）。

## 目录

- 一、切片测试注解选择 —— 每层只装载该层
- 二、Web 层 MockMvc —— @WebMvcTest + jsonPath
- 三、集成测试 Testcontainers —— 禁 H2 兼容假设
- 四、测试数据与事务 —— 回滚语义与陷阱
- 五、配置与 profile
- 六、@MockitoBean 使用边界

## 一、切片测试注解选择

**每个测试只装载被测层**——全量上下文 `@SpringBootTest` 启动慢、耦合全应用，仅用于金路径集成验证：

| 注解 | 装载范围 | 用途 |
| --- | --- | --- |
| `@WebMvcTest(ModelController.class)` | Controller + MVC 基础设施 | Web 层契约（配 MockMvc） |
| `@DataJpaTest` | Repository + 内存库（或接真实库） | 持久层查询与映射 |
| `@DataRedisTest` / `@DataJdbcTest` / `@JsonTest` | 对应组件 | 相应切片 |
| `@SpringBootTest` | 全上下文 | 集成与金路径（最少） |

```java
// ❌ 单元级验证拉起全上下文
@SpringBootTest
class ModelServiceTest { … }

// ✅ 纯业务逻辑无 Spring 依赖，直接 new + mock（最快）
class ModelServiceTest {
    private final ModelService service =
        new ModelService(mock(ModelRepository.class), mock(EventPublisher.class));
}
```

## 二、Web 层 MockMvc

`@WebMvcTest` 只装载 Web 层，业务服务用 `@MockitoBean` 桩（Boot 3.4+；3.3 及以前是 `@MockBean`）：

```java
@WebMvcTest(ModelController.class)
class ModelControllerTest {
    @Autowired MockMvc mockMvc;

    @MockitoBean ModelService modelService;

    @Test
    void 创建模型返回201与模型体() throws Exception {
        when(modelService.create(any())).thenReturn(
            new ModelInfo(1L, "Alpha", Status.DRAFT));

        mockMvc.perform(post("/api/models")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name": "Alpha"}
                    """))
            .andExpect(status().isCreated())
            .andExpect(header().string("Location", containsString("/api/models/")))
            .andExpect(jsonPath("$.name").value("Alpha"))
            .andExpect(jsonPath("$.id").isNumber());
    }
}
```

- `jsonPath` 断言**具体值与类型**，禁只断言状态码
- Controller 内禁业务逻辑：切片测试暴露"控制器做业务"时，把逻辑下沉 Service

## 三、集成测试 Testcontainers

- **禁 H2 兼容假设**：H2 与生产库的方言差异（MySQL 特有 SQL、锁行为、JSON 函数、大小写敏感）是"测试全绿、线上爆炸"的事故源——持久层与集成测试用 Testcontainers 拉起**与生产一致的数据库**
- 容器镜像版本与生产对齐（`mysql:8.0` 而非 `latest`）

```java
@Testcontainers
@SpringBootTest
class ModelIntegrationTest {

    @Container
    @ServiceConnection                       // Boot 3.1+：自动注入连接属性
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0");

    @Autowired ModelRepository modelRepository;

    @Test
    void 保存并按名称查询() {
        modelRepository.save(Model.builder().name("Alpha").build());
        assertThat(modelRepository.findByName("Alpha")).isPresent();
    }
}
```

- 无 `@ServiceConnection` 的中间件用 `@DynamicPropertySource` 注入连接属性：

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

- **单例容器模式**提性能：容器声明为 `static` + 手动启动（或复用容器注解），全测试类共享，禁每用例重启容器
- 本机无 Docker 时该类测试的跳转策略按项目 CI 约定（CI 必有 Docker）

## 四、测试数据与事务

- 切片/集成测试加 `@Transactional` 默认**测试结束回滚**——数据不落库、用例隔离
- 回滚语义的三个陷阱：
  1. 测试内代码自己提交（`REQUIRES_NEW`、独立事务管理器）→ 数据残留
  2. 异步线程看不到未提交数据 → 断言失败
  3. `@Sql` 脚本与回滚混用顺序混乱 → 统一数据准备方式

```java
// ✅ 集成数据用 @Sql 脚本准备
@Test
@Sql("/testdata/models.sql")
void 按状态查询() { … }
```

- 测试数据用数据构造器 / `@Sql` / Testcontainers 初始化脚本，禁依赖执行顺序

## 五、配置与 profile

- 测试专属配置放 `src/test/resources/application-test.yml`，测试类标 `@ActiveProfiles("test")`
- 测试专属 Bean 用 `@TestConfiguration`（内部类或独立文件），禁改生产配置类迁就测试
- 外部依赖地址一律走配置注入（`@DynamicPropertySource` / `@ServiceConnection`），禁硬编码

## 六、@MockitoBean 使用边界

- Boot 3.4+ 用 `@MockitoBean` 替代已废弃的 `@MockBean`（语义一致）
- mock Bean **少量且明确**；能构造真实例的（内存实现、配置对象）不 mock——mock 掉基础设施配置类会让切片测试失去意义
