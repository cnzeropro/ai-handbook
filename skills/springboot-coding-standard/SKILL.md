---
name: "springboot-coding-standard"
description: "Spring Boot 编码规范，按版本组织（Boot 3.x 当前主流 / 2.x 遗留）：配置管理与 @ConfigurationProperties、条件装配与 Bean 管理、全局异常处理、Web 层约定、缓存、异步与调度、Actuator 可观测、安全基线。当编写、重构或审查 Spring Boot 工程代码时使用。凡涉及 Spring Boot 代码编写与修改的任务均应主动使用。"
---

# Spring Boot 编码规范

本 skill 提供 Spring Boot 框架层编码规范（**Boot 3.x 当前主流，兼容 2.x 遗留注记**），规则来源为 Spring 官方参考文档（docs.spring.io/spring-boot）。分层架构、对象模型（PO/DTO/PO(Param)/VO）、注解与事务、接口设计、依赖管理见 `java-feature-standard`；语言级写法见 `java-coding-standard`；测试见 `java-testing-standard`（含 `spring-boot-test.md` 切片测试分册）。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查 Spring Boot 工程代码：配置类、装配、全局异常、Web 层、缓存、异步调度、Actuator
- 设计配置结构（application.yml 分层与 profile）或判断循环依赖、事务边界问题时
- 代码审查时作为 Spring Boot 规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- Spring Boot 3.x 项目（当前主流：jakarta 命名空间、Java 17+）
- Spring Boot 2.x 遗留项目（javax 命名空间，存量巨大）：保持既有写法一致，差异对照见 `springboot-legacy.md`

### 不适用（边界）

- 分层架构、对象模型（PO/DTO/PO(Param)/VO）、注解与事务、接口设计、二方库与依赖管理（参见 `java-feature-standard` skill）
- Java 语言级写法（参见 `java-coding-standard` skill）
- 测试写法（参见 `java-testing-standard` skill；其 `spring-boot-test.md` 分册覆盖切片测试）
- 非 Spring Boot 的 Java 应用（纯脚本、CLI）
- Spring Security 细粒度配置、微服务治理（注册发现/网关/熔断）等：跟随项目与官方文档

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **配置管理**：配置属性类 `@ConfigurationProperties` 类型安全绑定（优于散落 `@Value`）；`@Validated` 启动期校验；配置按 profile 分层，禁魔法值进代码
- **依赖注入**：构造器注入为默认（官方推荐，利于测试与不可变）；字段注入禁用；单构造器省略 `@Autowired`
- **循环依赖**：出现即设计坏味道——靠 `@Lazy` / `@PostConstruct` 掩盖是反模式，回到职责拆分
- **条件装配**：`@Conditional*` 用于 starter / 基础设施模块；业务代码少用
- **全局异常处理**：`@RestControllerAdvice` 统一错误响应结构（错误码 / 消息 / traceId）；禁吞异常、禁堆栈外泄、校验错误聚合返回
- **Web 层**：参数校验 `@Valid` + `@Validated` 分组；状态码语义化；统一响应包装需全局一致（含错误路径）
- **缓存**：`@Cacheable` 抽象 + 显式 key 规约 + TTL 必设；一致性约定"先更库后删缓存"
- **异步与调度**：`@Async` 必须自定义线程池；`@Scheduled` 池化 + 幂等（多实例部署分布式锁）；优雅停机开启
- **Actuator**：暴露端点最小化（health / info；prometheus 按需 + 鉴权）；禁暴露敏感端点到公网
- **安全基线**：端点默认拒绝显式放行；密码 BCrypt 编码；CSRF 按 API/表单形态取舍（细节跟随项目安全方案）

## 二、版本与基线

| 版本 | 说明 | 详细规范 |
| --- | --- | --- |
| Boot 2.x（遗留） | javax 命名空间、Java 8、Spring 5；OSS 已 EOL，存量巨大 | `references/springboot-legacy.md` |
| Boot 3.x（当前主流） | jakarta 命名空间、Java 17+、虚拟线程、Observability、ProblemDetail | `references/general-quality.md`（通用）+ `references/springboot3.md`（版本特性）+ `references/style-examples.md`（正反例） |

> Boot 4.0 已发布（2025-11，基于 Framework 7 / Jakarta EE 11）：核心工程约定延续 3.x；升级用官方迁移工具（OpenRewrite recipe、`spring-boot-properties-migrator`）评估，属独立任务——不与功能变更混杂。

## References（完整规范与代码示例）

- `references/general-quality.md` — 版本通用规范：配置管理、Bean 管理与条件装配、全局异常处理、Web 层约定、缓存、异步与调度、Actuator 可观测、安全基线
- `references/springboot3.md` — Boot 3.x 写法规约：jakarta、构造绑定、Observability、虚拟线程、RestClient、ProblemDetail、SSL Bundles、升级规约
- `references/springboot-legacy.md` — Boot 2.x 遗留环境：写法基线、与 Boot 3 差异对照、最小侵入原则
- `references/style-examples.md` — 编码风格完整示例与常见正反例速查
