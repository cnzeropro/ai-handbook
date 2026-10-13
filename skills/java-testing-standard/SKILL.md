---
name: "java-testing-standard"
description: "Java 测试编码规范（JUnit 5 + Mockito + AssertJ 基线，JUnit 4 遗留分册）：测试策略与分层、用例设计与命名、参数化、AssertJ 断言、Mockito 桩与验证、Spring Boot 切片测试与 Testcontainers。当编写、重构或审查 Java 测试代码时使用。凡涉及 Java 测试代码编写与修改的任务均应主动使用。"
---

# Java 测试编码规范

本 skill 提供 Java 测试编码规范（领域专项），基线 **JUnit 5 + Mockito + AssertJ**（Spring Boot 生态主流组合），覆盖 Spring Boot 测试（切片测试 / MockMvc / Testcontainers）。规则来源为 JUnit 5 官方用户指南、Spring Boot Testing 官方文档与各工具官方文档。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。
>
> 与其他 skill 的关系：被测代码本身的写法规约见 `java-coding-standard`；模块结构与分层见 `java-feature-standard`；测试代码本身的命名 / 格式 / 注释同样遵循 `java-coding-standard`，本 skill 只约束测试特有规则。

## 何时使用

- 编写、重构或审查 Java 测试：单元测试、切片测试（MockMvc / DataJpaTest）、集成测试（Testcontainers）
- 设计测试分层策略、判断"该不该 mock、mock 到哪一层"时
- 处理测试数据构造、参数化用例、测试稳定性时
- 代码审查时作为测试规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- Java 测试代码：单元测试（JUnit 5 + Mockito + AssertJ）、Spring Boot 切片与集成测试（MockMvc / Testcontainers）
- Spring Boot 技术栈（与 `java-feature-standard` 同栈）；非 Spring 的纯 Java 单元测试同样适用

### 不适用（边界）

- JUnit 4 及以下遗留项目（参见 `junit-legacy.md`；TestNG 项目遵循官方文档）
- 前端测试（参见 `frontend-testing-standard` skill）
- 被测代码本身的写法规约（参见 `java-coding-standard` skill；测试代码本身的命名、格式、注释也遵循它）
- 性能压测、安全渗透测试（不属于写法规约范畴）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **测试分层**：单元测试为主，切片测试（`@WebMvcTest` / `@DataJpaTest`）居中，全量 `@SpringBootTest` 与 Testcontainers 集成测试最少——越往下越慢越脆
- **测行为，不测实现**：断言业务结果，不断言内部状态、私有方法与 Mock 调用细节；"重构实现（不改行为）后测试必须全绿"
- **bug 先写复现测试**：先写暴露缺陷的失败测试，再修复使其转绿——回归防护随修复沉淀
- **FIRST**：Fast / Independent / Repeatable / Self-validating / Timely；测试间禁相互依赖与共享可变静态状态
- **AAA 结构**：Arrange-Act-Assert 空行分段；一个 `@Test` 只测一个主题
- **用例命名用 `@DisplayName`** 表达"条件 + 预期"（中文）；方法名保持合法标识符
- **断言统一 AssertJ 链式**：`assertThat(...)`，异常用 `assertThatThrownBy`；禁 JUnit assertions 与 AssertJ 混用
- **参数化表驱动**：多组输入输出用 `@ParameterizedTest`，禁复制粘贴近似用例
- **mock 边界**：只 mock 外部依赖与不可控边界（网络、时钟、随机）；禁 mock 被测类内部；值对象 / DTO 禁 mock
- **单元测试禁真实时间 / 随机 / 网络 / 文件系统**——不确定性与慢的来源；集成层用 Testcontainers
- **覆盖率是参考指标**：不为凑数写无断言测试；bug 修复先补复现测试

## 二、版本与基线

| 版本 | 说明 | 详细规范 |
| --- | --- | --- |
| JUnit 5 + Mockito 5 + AssertJ（当前基线） | 注解 / 参数化 / 嵌套 / AssertJ 断言 / Mockito 桩与验证 | `references/junit5-mockito.md` |
| Spring Boot 测试 | 切片测试注解、MockMvc、Testcontainers、`@ServiceConnection` | `references/spring-boot-test.md` |
| JUnit 4 及以下（遗留） | 注解包 / Runner / ExpectedException 体系 | `references/junit-legacy.md` |

## References（完整规范与代码示例）

- `references/general-quality.md` — 版本通用规范：测试策略与分层、测行为不测实现、用例设计、测试数据、mock 边界、稳定性、覆盖率态度
- `references/junit5-mockito.md` — JUnit 5 + Mockito 写法规约：注解与生命周期、参数化、AssertJ 断言、Mockito 桩与验证、常见陷阱
- `references/spring-boot-test.md` — Spring Boot 测试：切片注解选择、MockMvc、Testcontainers、事务回滚、配置
- `references/junit-legacy.md` — JUnit 4 遗留环境：写法对照、最小侵入原则
- `references/style-examples.md` — 编码风格完整示例与常见正反例速查
