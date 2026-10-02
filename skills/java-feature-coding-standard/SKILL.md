---
name: "java-feature-coding-standard"
description: "提供 Java 后端功能模块开发规范：分层架构、命名约定（包/类/方法/Param/VO）、对象模型（PO/DTO/Param/VO）、注解、事务管理、数据更新策略与开发流程。适用于 Spring Boot/Spring MVC/MyBatis(-Plus)/JPA/Dubbo/Feign/MQ 等技术栈；当需要新建功能或模块、重构旧代码、统一模块结构与命名时使用。语言级编码规范见 `java-coding-standard`。"
---

# 功能模块开发规范

此 skill 提供 Java 后端功能模块开发规范。核心规则见本文件；各层完整代码示例见 `references/`（按需读取，不默认加载）。语言级编码规范（注释、格式、常量枚举、异常、日志等）见 `java-coding-standard`。

> 示例说明：本文及 `references/` 中的所有示例以"模型管理平台"业务域（Model + Job 任务调度 + Deployment 部署）演示语法与结构，`Model`/`Job` 均为占位业务名，实际开发时替换为项目自身的业务域。

## 何时使用

- 新建功能模块或从零开发新功能
- 重构旧代码、统一模块结构、命名与分层风格
- 代码审查时作为模块规范检查依据
- 咨询命名（包/类/方法/Param/VO）或分层结构问题
- 不确定 PO/Param/VO/DTO 该如何选择时

## 适用范围

### 适用对象（本规则适用于）

**技术栈**

- 语言与基础库：Java 8 / 11 / 17 / 21（按项目版本使用最佳实践语法，见 `java-coding-standard`）、Lombok、Hutool / Guava
- Web / 接口层：Spring Boot / Spring MVC / Spring WebFlux、Jakarta Validation
- 持久层：MyBatis / MyBatis-Plus、Spring Data JPA / Hibernate、jOOQ、Spring JDBC
- 远程调用：Spring Cloud OpenFeign、Spring 6 HTTP Interface（`@HttpExchange`）、Dubbo、gRPC、RestTemplate / WebClient
- 消息与任务：RocketMQ / Kafka / RabbitMQ、XXL-Job、Spring `@Scheduled`
- 缓存与基础设施：Redis、Nacos / Consul、SLF4J + Logback
- 测试：JUnit 5 + Mockito、Testcontainers

**架构形态**

- 微服务多模块分层架构（`common` 公共模块 + 各业务模块）、单体应用分层架构
- 经典 Controller → Service → Mapper/Repository 三层结构

**代码对象**

- Controller / Service 接口 / ServiceImpl / Mapper 接口 / Mapper XML（或 Repository）
- PO（持久化对象）/ DTO（数据传输对象）/ Param（请求参数对象）/ VO（响应视图对象）
- 枚举、常量类、自定义业务异常

**技术栈适配说明**

核心规范（分层职责、对象模型、注解、事务、数据更新策略）与技术栈无关；示例默认以 Spring Boot + MyBatis-Plus 呈现，使用其他技术栈时按以下映射替换：

| 概念 | MyBatis-Plus（示例默认） | 纯 MyBatis | Spring Data JPA |
| --- | --- | --- | --- |
| 持久化对象 | `{Entity}PO` + `@TableName` | 同左 | `{Entity}PO` + `@Entity` / `@Table` |
| 数据访问接口 | `{Entity}Mapper extends BaseMapper<PO>` | `{Entity}Mapper`（注解或 XML） | `{Entity}Repository extends JpaRepository<PO, ID>` |
| Service 基类 | `IService<PO>` / `ServiceImpl<Mapper, PO>` | 普通接口 + `@Service` 实现类 | 普通接口 + `@Service` 实现类（注入 Repository） |
| 分页返回 | `IPage<T>` | `PageInfo<T>`（PageHelper） | `Page<T>`（Spring Data） |

### 不适用（边界）

- 非 Java 项目（前端、Python、Go 等）
- 非分层架构的 Java 应用（纯脚本、CLI 工具，编码规范参见 `java-coding-standard`）
- 纯静态工具库、算法库
- Java 语言层面的编码规范（命名、注释、格式、常量枚举、异常、日志、工具类等，参见 `java-coding-standard` skill）
- 数据库表设计与 SQL 脚本规范（参见 `db-design-standard` skill）
- 接口方法排序规则（参见 `java-method-ordering` skill）
- 代码安全审查流程（参见 `java-code-review` skill）

## 核心工作流

1. **模块确认**：确定功能所属模块，创建包结构（见“二、命名规范”）
2. **常量枚举**：创建常量类与枚举（规范见 `java-coding-standard`）
3. **对象模型**：创建 PO → Param/VO → DTO（见“三”）
4. **数据访问**：创建 Mapper/Repository 接口与 XML（见“一”）
5. **业务逻辑**：创建 Service 接口与实现（见“一”）
6. **接口层**：创建 Controller（见“一”）
7. **规范自检**：逐条核对“八、验收清单”

## 一、分层架构与职责

| 层 | 职责 | 禁止事项 |
| --- | --- | --- |
| Controller | 接收 HTTP 请求、参数校验、调用 Service、组装返回 | 编写业务逻辑、直接调用 Mapper |
| Service | 业务逻辑处理、事务管理、调用 Mapper | 处理 HTTP 协议细节 |
| Mapper / Repository（数据访问层） | 数据库访问、SQL 执行 | 编写业务逻辑 |

### 1. Controller 层规范

1. 使用 `@RequiredArgsConstructor` 构造器注入依赖
2. 使用 `@RestController` 和 `@RequestMapping` 注解
3. 方法返回统一使用 `Result` 包装类；下载（导出）相关方法除外，需使用 `ResponseEntity`
4. 接口风格：遵循 RESTful 风格——查询用 `@GetMapping`，新增用 `@PostMapping`，更新用 `@PutMapping`（部分更新可用 `@PatchMapping`），删除用 `@DeleteMapping`
5. 参数使用 `@RequestParam` 或 `@RequestBody`
6. 方法顺序遵循 `java-method-ordering` skill（查询 → 新增 → 更新 → 删除 → 其他）
7. 异常交由全局异常处理器统一处理，不逐接口堆叠 try-catch（异常书写规范见 `java-coding-standard`）

### 2. Service 层规范

1. Service 接口：MyBatis-Plus 项目继承 `IService<PO>`；其他技术栈使用普通接口
2. Service 实现：MyBatis-Plus 项目继承 `ServiceImpl<Mapper, PO>`；其他技术栈使用普通 `@Service` 实现类，注入 Mapper/Repository
3. 使用 `@Transactional` 管理事务（见“五、事务管理规范”）
4. 使用 `Optional` 处理可能为 null 的值
5. 使用 `Wrappers` 构建查询条件（MyBatis-Plus）
6. 方法顺序遵循 `java-method-ordering` skill，与 Controller 上下对齐
7. 业务异常直接抛出，不吞异常（见 `java-coding-standard`）

### 3. Mapper / Repository 层规范

1. MyBatis-Plus 项目：Mapper 接口继承 `BaseMapper<PO>`；JPA 项目：`Repository` 继承 `JpaRepository<PO, ID>`
2. 使用 `@Mapper` 注解（可选，MyBatis-Plus 会自动扫描）；JPA 项目使用 `@Repository`
3. XML 文件位于 `mapper/xml/` 目录下
4. 使用 `resultMap` 映射结果集
5. 除基本 CRUD 外，大部分联表查询语句在 Mapper XML 中编写

### 4. 分层职责正反例

```java
// ❌ 错误：Controller 编写业务逻辑、直接调用 Mapper
// ✅ 正确：Controller 只做参数校验与转发，业务逻辑在 Service
```

完整代码见 `references/layer-examples.md`。

## 二、命名规范

### 1. 包命名

| 包 | 路径 |
| --- | --- |
| 模块包 | `com.example.module.{module-name}` |
| 常量包 | `com.example.module.{module-name}.constant` |
| 控制器包 | `com.example.module.{module-name}.controller` |
| 服务包 | `com.example.module.{module-name}.service` |
| 服务实现包 | `com.example.module.{module-name}.service.impl` |
| Mapper 包 | `com.example.module.{module-name}.mapper` |
| Mapper XML 包 | `com.example.module.{module-name}.mapper.xml` |
| 工具类包 | `com.example.module.{module-name}.util` |
| 持久化对象包 | `com.example.module.{module-name}.model.persistent` |
| 数据传输对象包 | `com.example.module.{module-name}.model.transfer` |
| 请求参数对象包 | `com.example.module.{module-name}.model.param` |
| 响应视图对象包 | `com.example.module.{module-name}.model.view` |

**正反例：**

```java
// ❌ 错误：缺少 module 层级 / PO、Param、VO 混在同一包
package com.example.model;
package com.example.module.project.vo;

// ✅ 正确：按模块 + 对象类型分层分包
package com.example.module.project;
package com.example.module.project.model.param;
package com.example.module.project.model.view;
```

### 2. 类命名

| 类型 | 命名规则 | 示例 |
|------|---------|------|
| Controller | `{Entity}Controller` | `ModelController`, `ModelJobController` |
| Service 接口 | `{Entity}Service` | `ModelService`, `ModelJobService` |
| Service 实现 | `{Entity}ServiceImpl` | `ModelServiceImpl`, `ModelJobServiceImpl` |
| Mapper 接口 | `{Entity}Mapper` | `ModelMapper`, `ModelJobMapper` |
| Mapper XML | `{Entity}Mapper.xml` | `ModelMapper.xml`, `ModelJobMapper.xml` |
| PO（持久化对象） | `{Entity}PO` | `ModelPO`, `ModelJobPO` |
| DTO（数据传输对象） | `{Entity}DTO` | `ModelDTO`, `ModelJobDTO` |
| Param（请求参数对象） | `{Entity}{Operation}PO` | `ModelListPO`, `ModelListWithUserPO`, `ModelSavePO` |
| VO（响应视图对象） | `{Entity}{Operation}VO` | `ModelListVO`, `ModelListWithUserVO` |
| 枚举 | `{Entity}Status`, `{Entity}Type` | `ModelStatus`, `ModelJobStatus` |

**重要规则：**

- 任何接口都必须有自己的 Param（请求参数对象）和 View（响应视图对象，如果存在返回值）
- 禁止在接口中直接使用持久化对象
- 因此持久化对象可以放在各自的微服务模块中，无需放在 `common` 模块

**正反例：**

```java
// ❌ 错误：接口直接暴露持久化对象 / 实现类命名不符约定
public Result<ModelPO> getById(@RequestParam Long id) { ... }
public class ModelControllerImpl { ... }   // Controller 不需要 Impl
public class ModelDao { ... }              // 应使用 Mapper

// ✅ 正确：接口使用 Param/VO，命名符合约定
public Result<ModelGetByIdVO> getById(@RequestParam Long id) { ... }
public class ModelController { ... }
public interface ModelMapper extends BaseMapper<ModelPO> { ... }
```

### 3. 方法命名

| 功能 | 命名规则 | 示例 |
|------|---------|------|
| 查询单个 | `getBy{Field}` | `getById` |
| 查询单个并带其他信息 | `getWith{Info}By{Field}` | `getWithUserById`, `getWithJobAndUserById` |
| 查询列表 | `list`, `listBy{Field}` | `list`, `listByGroupId` |
| 查询列表带其他信息 | `listWith{Info}`, `listWith{Info}By{Field}` | `listWithUser`, `listWithJobAndUserByGroupId` |
| 分页查询 | `page` | `page` |
| 分页查询带其他信息 | `pageWith{Info}` | `pageWithUser`, `pageWithJobAndUser` |
| 统计 | `count` | `count` |
| 保存 | `save` | `save` |
| 带其他信息并一起保存 | `saveWith{Info}` | `saveWithUser`, `saveWithJobAndUser` |
| 更新 | `updateBy{Field}` | `updateById` |
| 带其他信息并一起更新 | `updateWith{Info}By{Field}` | `updateWithUserById`, `updateWithJobAndUserById` |
| 删除单个 | `removeBy{Field}` | `removeById` |
| 删除单个并带其他信息一起删除 | `removeWith{Info}By{Field}` | `removeWithUserById`, `removeWithJobAndUserById` |
| 批量删除 | `removeBy{Field}s` | `removeByIds` |
| 批量删除并带其他信息一起删除 | `removeWith{Info}By{Field}s` | `removeWithUserByIds`, `removeWithJobAndUserByIds` |
| 启用/停用 | `enable`, `disable` | `enable`, `disable` |
| 执行（启动）/停止（终止） | `run`, `execute`, `start`, `stop`, `terminate`, `pause`, `interrupt`, `restart`, `resume` | `run` |
| 上传/下载 | `upload`, `download` | `upload`, `download` |
| 导入/导出 | `import{Data}`, `export{Data}` | `importJob`, `exportJob` |
| 暂存 | `stage` | `stage` |
| 部署 | `deploy` | `deploy` |

**正反例：**

```java
// ❌ 错误：命名不体现语义
ModelGetWithUserVO getModelInfo(Long id);        // 无法看出是 ById 还是 With 信息
List<ModelListVO> queryAll();                      // 应使用 list
boolean deleteModelById(Long id);                // 应使用 removeById

// ✅ 正确
ModelGetWithUserVO getWithUserById(Long id);
Collection<ModelListVO> list(ModelListPO param);
boolean removeById(Long id);
```

### 4. Param（请求参数对象）命名规则

- 列表查询：`{Entity}ListPO`
- 带其他信息的列表查询：`{Entity}ListWith{Info}PO`
- 分页查询：`{Entity}PagePO`
- 带其他信息的分页查询：`{Entity}PageWith{Info}PO`
- 根据 ID 查询：`{Entity}GetByIdPO`
- 带其他信息的根据 ID 查询：`{Entity}GetByIdWith{Info}PO`
- 统计：`{Entity}CountPO`
- 保存：`{Entity}SavePO`
- 带其他信息的保存：`{Entity}SaveWith{Info}PO`
- 通过 ID 更新：`{Entity}UpdateByIdPO`
- 带其他信息通过 ID 更新：`{Entity}UpdateByIdWith{Info}PO`
- 通过 ID 启用：`{Entity}EnableByIdPO`
- 通过 ID 停用：`{Entity}DisableByIdPO`
- 通过 ID 删除：`{Entity}RemoveByIdPO`
- 批量 ID 删除：`{Entity}RemoveByIdsPO`
- 带其他信息通过 ID 删除：`{Entity}RemoveByIdWith{Info}PO`
- 带其他信息批量 ID 删除：`{Entity}RemoveByIdsWith{Info}PO`

**正反例：**

```java
// ❌ public Result<Void> save(@RequestBody ModelPO model) { ... }            // 直接用 PO
// ❌ public Result<Void> update(@RequestBody UpdateModelParam param) { ... } // 未按 Entity+Operation 命名
// ✅ public Result<Void> save(@RequestBody ModelSavePO modelSaveParam) { ... }
// ✅ public Result<Void> updateById(@RequestBody ModelUpdateByIdPO modelUpdateByIdParam) { ... }
```

### 5. VO（响应视图对象）命名规则

- 列表视图：`{Entity}ListVO`
- 带其他信息的列表视图：`{Entity}ListWith{Info}VO`
- 分页视图：`{Entity}PageVO`
- 带其他信息的分页视图：`{Entity}PageWith{Info}VO`
- 详情视图：`{Entity}GetByIdVO`
- 带其他信息的详情视图：`{Entity}GetByIdWith{Info}VO`

**正反例：**

```java
// ❌ public Result<List<UserVO>> list() { ... }                 // 应使用 ModelListVO
// ❌ public Result<ModelVO> getWithUserById(Long id) { ... }  // 应使用 ModelGetByIdWithUserVO
// ✅ public Result<Collection<ModelListVO>> list(ModelListPO param) { ... }
// ✅ public Result<ModelGetByIdWithUserVO> getWithUserById(Long id) { ... }
```

## 三、对象模型规范（PO/DTO/Param/VO）

| 对象 | 目录 | 用途 | 命名 |
| --- | --- | --- | --- |
| PO（持久化对象） | `model/persistent/` | 对应数据库表 | `{Entity}PO` |
| DTO（数据传输对象） | `model/transfer/` | 模块间数据传输 | `{Entity}DTO` |
| Param（请求参数对象） | `model/param/` | 接收请求参数 | `{Entity}{Operation}PO` |
| VO（响应视图对象） | `model/view/` | 复杂视图展示 | `{Entity}{Operation}VO` |

### 1. PO（持久化对象）

- `@TableName` 指定表名、`@TableId` 指定主键、`@TableField` 指定字段映射（JPA 项目对应 `@Entity` / `@Id` / `@Column`）
- 主键 `id` 用 `Long` + `@TableId(type = IdType.ASSIGN_ID)`（应用层雪花 ID，与 `db-design-standard` 主键规则对应）
- 逻辑删除字段用 `@TableLogic`（`isDeleted`）：数据库 `is_deleted` 与属性 `isDeleted` 为下划线转驼峰自动映射，布尔/标志字段使用 `Integer` / `Boolean` 包装类型
- 使用 lombok 注解

### 2. DTO（数据传输对象）

- 微服务多模块架构中位于 `common` 公共模块（供模块间共享）；单体应用放在对应业务模块。用于模块间数据传输
- 使用 lombok 注解

### 3. Param（请求参数对象）

- 微服务多模块架构中位于 `common` 公共模块；单体应用放在对应业务模块。用于接收请求参数
- 使用 lombok 注解

### 4. VO（响应视图对象）

- 微服务多模块架构中位于 `common` 公共模块；单体应用放在对应业务模块。用于复杂视图展示
- 包含多个表的关联数据、额外的计算字段或展示字段
- 使用 lombok 注解

### 5. 使用边界正反例

```java
// ❌ 错误：Controller 入参出参直接用 PO/DTO
public Result<ModelPO> getById(@RequestParam Long id) { ... }
public Result<ModelDTO> sync(ModelDTO dto) { ... }     // 接口层不应暴露 DTO

// ✅ 正确：接口用 Param/VO，DTO 仅用于模块间（Feign/RPC）传输
public Result<ModelGetByIdVO> getById(@RequestParam Long id) { ... }
public Result<Void> save(@RequestBody ModelSavePO modelSaveParam) { ... }
```

完整类定义见 `references/object-model-examples.md`。

## 四、注解规范

### 1. Lombok 注解

- Controller：`@RequiredArgsConstructor`（构造器注入）
- Service 实现：`@Slf4j`
- 对象类（PO/DTO/Param/VO）：`@Data` + `@SuperBuilder(toBuilder = true)` + `@NoArgsConstructor` + `@AllArgsConstructor`
- 枚举：`@Getter`

> 注：`@SuperBuilder` 需 Lombok 1.18.2+；使用旧版 Lombok 的项目可退回 `@Builder`

```java
// ❌ @Autowired 字段注入（难以测试、隐藏依赖）
// ✅ @RequiredArgsConstructor + private final 构造器注入
```

### 2. Spring 注解

- Controller：`@RestController` + `@RequestMapping("{entity}")`
- Service 实现：`@Service`
- Mapper：`@Mapper`（可选）
- 事务：`@Transactional(rollbackFor = Exception.class)`；只读：`@Transactional(readOnly = true)`

完整示例见 `references/annotation-examples.md`。

## 五、事务管理规范

- 使用 `@Transactional` 注解
- 指定 `rollbackFor = Exception.class`
- 只读事务使用 `readOnly = true`

```java
// ❌ 错误：不指定 rollbackFor，受检异常与 RuntimeException 之外的异常不触发回滚
// ❌ 错误：写操作标注 readOnly，事务无效或行为异常
// ✅ 正确：写操作 rollbackFor = Exception.class；查询标注 readOnly = true
```

完整示例见 `references/annotation-examples.md`。

## 六、数据与数据库变更规范

### 1. 关联数据全量更新策略

更新关联数据时，入参语义约定：

- `null`：表示不更新，保持原状
- `[]`（空列表）：表示清空关联数据
- 非空列表：先删除旧数据，再插入新数据

```java
// ❌ 错误：null 被当作"清空"，导致误删关联数据
// ✅ 正确：严格区分 null 与空列表（null 跳过；其余先删后插）
```

### 2. 级联删除

删除父实体时，必须同步删除所有关联的子实体数据。

```java
// ❌ 错误：只删除父实体，留下孤儿数据
// ✅ 正确：同步级联删除子实体（@Transactional 内先删子后删父）
```

### 3. 数据库变更规范

- **多数据库支持**：项目同时使用多种数据库时，每种目标数据库的脚本均需提供（单库项目仅需该库脚本）
- **默认值安全**：新增字段时，默认值应设为 `NULL` 或安全值，避免影响历史数据
- **逻辑删除**：所有实体表应包含 `is_deleted` 字段，并使用逻辑删除而非物理删除

## 七、最佳实践与常见错误规避

### 1. 性能优化

- 使用批量操作减少数据库访问
- 合理使用缓存提高查询性能
- 异步处理耗时操作
- 分页查询大数据量

### 2. 安全考虑

- 输入参数验证
- SQL 注入防护
- XSS 攻击防护
- 敏感信息脱敏

### 3. 可维护性

- 代码注释清晰
- 业务逻辑单一
- 避免硬编码
- 模块职责明确

### 4. 遗留代码兼容

- **风格一致性**：在维护遗留模块时，优先遵循该模块现有的命名风格（如 DTO vs PO），除非进行全面重构

## 八、验收清单

### 开发流程自检

- [ ] 包结构符合 `com.example.module.{module-name}.*` 约定
- [ ] 常量与枚举已创建（状态/类型字段用英文字符串值，见名知意）
- [ ] PO → Param/VO 对象模型已创建，接口未直接暴露 PO
- [ ] Mapper/Repository 接口与 XML 已创建
- [ ] Service 接口与实现已创建，方法顺序与 Controller 对齐
- [ ] Controller 已创建，只做参数校验与转发
- [ ] 单元测试覆盖率不低于 70%

### 代码规范自检

- [ ] 遵循命名规范（包/类/方法/Param/VO）
- [ ] 代码风格与编码规范遵循 `java-coding-standard`（注释、导入、格式、常量枚举、工具类等）
- [ ] 按项目 Java 版本使用最佳实践语法（不越级使用高版本特性）
- [ ] 使用正确的注解（构造器注入、`@Slf4j`、`@Transactional(rollbackFor = Exception.class)`）
- [ ] 事务管理正确（写操作回滚、只读事务标注 `readOnly`）
- [ ] 异常处理完善（Service 抛业务异常，不吞异常；记录与抛出择一，无重复日志）
- [ ] 日志记录规范（占位符、上下文、堆栈、无敏感信息）
- [ ] 关联数据更新遵循 null/[]/非空列表语义
- [ ] 级联删除同步删除子实体
- [ ] 数据库变更为每种目标数据库提供脚本
- [ ] 使用 `git add <file>` 精确添加，提交前 `git status` 检查

## References（完整代码示例）

- `references/layer-examples.md` — 分层架构各层完整代码（Controller/Service/ServiceImpl/Mapper/XML）
- `references/object-model-examples.md` — PO/DTO/Param/VO 完整类定义与 Lombok 示例
- `references/annotation-examples.md` — Spring 注解、字段注入、事务管理完整示例
