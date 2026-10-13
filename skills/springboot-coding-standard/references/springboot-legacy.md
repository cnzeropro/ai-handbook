# Spring Boot 2.x（遗留环境）规范

> 适用：Spring Boot 2.x 遗留项目（javax 命名空间；OSS 版本 2.7 已于 2023-11 EOL，存量巨大）。仅在维护既有代码或项目明确要求时使用；新项目一律 Boot 3.x（见 `springboot3.md`）。同一项目保持大版本与写法一致，不混用。

## 一、写法基线（与 Boot 3 的差异即本册约定）

- **javax 命名空间**：`javax.validation` / `javax.persistence` / `javax.servlet`
- **Java 8+**（Boot 2.x 支持 Java 8~17 中部分版本，2.7 支持 Java 8-20 区间按补丁）；语言特性跟随项目 JDK，禁越级使用 Boot 3 时代写法
- 配置绑定：`@ConfigurationProperties` + setter（无构造绑定）；或 `@ConstructorBinding` 显式标注（2.2+ 单构造场景）
- 无虚拟线程、无 RestClient（用 RestTemplate / WebClient）、无 ProblemDetail 内建（自造错误结构保持一致）
- `@MockBean` 正常使用（3.4 才废弃）

```java
// ✅ Boot 2.x 配置属性类（setter 形态）
@ConfigurationProperties(prefix = "model.upload")
@Validated
public class ModelUploadProperties {
    @Positive
    private Long maxSize;
    @NotEmpty
    private List<String> allowedTypes;
    // getter/setter …
}
```

## 二、与 Boot 3 差异对照（升级参考）

| Boot 3.x | Boot 2.x |
| --- | --- |
| `jakarta.*` | `javax.*` |
| Java 17+ | Java 8+ |
| Spring Framework 6 | Spring Framework 5 |
| `RestClient`（3.2+）/ `WebClient` | `RestTemplate` / `WebClient` |
| `@MockitoBean`（Boot 3.4+） | `@MockBean` |
| ProblemDetail 内建 | 自造错误结构 |
| Flyway 9+ | Flyway 版本随 Boot 2.x |
| Gradle 7.5+ / Maven 3.6.3+ | 视 2.x 小版本 |

## 三、遗留代码最小侵入

- 维护期与项目既有写法一致，**不顺手升级** Boot 大版本 / Spring 5→6 / javax→jakarta；升级作为独立任务
- 遗留库中的新类遵循 `java-coding-standard` 语言规约 + 本 skill 通用规范（配置/异常/缓存等原则），API 形态随 Boot 2.x
- 升级路径：先 2.7（2.x 内最后版本）→ 官方迁移评估（OpenRewrite recipe、`spring-boot-properties-migrator`）→ 3.x；依赖矩阵（Spring 6 / Hibernate 6 / 三方 jakarta 化）提前盘点
