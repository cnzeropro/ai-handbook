# Spring Boot 3.x 写法规约（当前主流）

> 本文件为 **Spring Boot 3.x** 版本特有约定（jakarta 命名空间、Observability、虚拟线程等），须与版本通用的 `general-quality.md` 配合使用。Boot 2.x 遗留环境见 `springboot-legacy.md`。基线 Spring Boot 3.x（3.1+ 特性随版本标注）。

## 目录

- 一、Jakarta 命名空间
- 二、@ConfigurationProperties 构造绑定
- 三、Observability 可观测
- 四、虚拟线程（3.2+）
- 五、RestClient（3.2+）
- 六、ProblemDetail（RFC 7807，3.0+）
- 七、SSL Bundles（3.1+）
- 八、升级规约（2.x → 3.x）

## 一、Jakarta 命名空间

Boot 3.x 基于 Jakarta EE 9+：`javax.*` → `jakarta.*` 全量替换（`jakarta.validation` / `jakarta.persistence` / `jakarta.servlet`）：

```java
// ❌ 3.x 环境残留 javax（编译不过或行为异常）
import javax.validation.constraints.NotNull;

// ✅
import jakarta.validation.constraints.NotNull;
```

- 三方依赖必须选 jakarta 兼容版本（Hibernate 6 / Tomcat 10+ 等）——升级评估的核心工作量

## 二、@ConfigurationProperties 构造绑定

不可变配置属性类：构造器绑定无需 setter（单构造器类自动启用）；**record 天然契合**：

```java
@Validated
@ConfigurationProperties(prefix = "model.upload")
public record ModelUploadProperties(
    @NotNull @Positive Long maxSize,
    @NotEmpty List<String> allowedTypes
) {}
```

- YAML 键与 record 组件同名（kebab-case 映射）；嵌套配置用嵌套 record

## 三、Observability 可观测（3.0+）

- Micrometer Observation API：一次观测同时产出 metrics 与 tracing（替代手工 MDC + 自定义指标二分）

```java
private final ObservationRegistry registry;

 Observation.createNotStarted("model.query", registry)
    .lowCardinalityKeyValue("dept", dept)
    .observe(() -> repository.findByDept(dept));
```

- 低基数值用 lowCardinalityKeyValue；高基数（userId）禁入指标标签（改走 trace）

## 四、虚拟线程（3.2+）

```yaml
spring:
  threads:
    virtual:
      enabled: true
```

- 适用 IO 密集（大量并发外部调用）；**CPU 密集无收益**
- `synchronized` 块内阻塞会钉住载体线程（pinning）——热点路径换 `ReentrantLock`
- 与虚拟线程亲和的上下文传递用 ScopedValue / ThreadLocal 兼容层按项目评估

## 五、RestClient（3.2+）

同步 HTTP 客户端，替代 `RestTemplate`（维护模式）：

```java
RestClient client = RestClient.builder()
    .baseUrl("https://api.example.com")
    .build();

ModelInfo model = client.get()
    .uri("/models/{id}", id)
    .retrieve()
    .body(ModelInfo.class);
```

- 响应式栈才用 `WebClient`；RestTemplate 遗留代码保持一致，不新增使用

## 六、ProblemDetail（RFC 7807，3.0+）

标准错误响应形态，替代自造错误结构（前后端约定允许时）：

```yaml
spring:
  mvc:
    problemdetails:
      enabled: true
```

```java
@ExceptionHandler(BizException.class)
ProblemDetail handleBiz(BizException e) {
    var pd = ProblemDetail.forStatusAndDetail(e.getStatus(), e.getMessage());
    pd.setTitle(e.getCode());
    return pd;
}
```

> 项目已有统一错误包装时保持既有结构（一致性优先），本条用于新项目选型。

## 七、SSL Bundles（3.1+）

证书/信任材料集中配置，替代散落的 keystore 参数：

```yaml
spring:
  ssl:
    bundle:
      jks:
        client:
          truststore: classpath:truststore.p12
          keystore: classpath:keystore.p12
```

```java
RestClient client = RestClient.builder()
    .setSslBundle(sslBundles.getBundle("client"))
    .build();
```

## 八、升级规约（2.x → 3.x）

- 升级属于**独立任务**：Java 17+ → jakarta 全量替换 → 依赖兼容矩阵核对 → 回归
- 用官方评估工具：OpenRewrite recipe（`org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_0`）+ `spring-boot-properties-migrator`（配置项改名提示）
- Flyway 需 9+、MySQL 驱动坐标变更（`com.mysql:mysql-connector-j`）等清单以官方 Release Notes 为准
