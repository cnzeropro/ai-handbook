---
name: "java-code-review"
description: "审查 Java 代码（Spring Boot/Spring MVC、MyBatis(-Plus)、JPA、Lombok、Hutool 等技术栈）的安全性、框架最佳实践、代码质量与性能。当用户请求审查、分析或审计代码、检查最佳实践时使用。"
---

# 代码审查

此 skill 用于全面审查 Java 代码，确保代码质量、安全性和可维护性。

## 何时使用

- 审查代码质量
- 分析代码安全性
- 审计代码规范符合性
- 检查代码最佳实践
- 评估代码可维护性

## 适用范围

### 适用对象（本规则适用于）

**技术栈**

- Java 8 / 11 / 17 / 21 / 25（按项目版本评估语法使用是否得当，见 `java-coding-standard`）
- Spring Boot / Spring MVC
- 持久层：MyBatis / MyBatis-Plus / MyBatis-Flex、Spring Data JPA / Hibernate
- Lombok、Hutool / Guava 等常用工具库

**代码对象**

- Controller / Service / ServiceImpl / Mapper（Repository）/ Mapper XML
- 实体类（PO(Persistent)/DTO/PO(Param)/VO）、工具类、配置类
- 枚举、常量、自定义异常

**审查场景**

- 提交前代码审查（PR review）
- 安全审计
- 规范符合性检查（以 `java-coding-standard`、`java-feature-standard`、`java-method-ordering`、`db-design-standard` 为检查依据）
- 重构前的代码健康度评估

### 不适用（边界）

- 非 Java 项目（前端、Python、Go 等）
- 数据库表设计与 SQL 脚本审查（参见 `db-design-standard` skill）
- 编码规范本身（参见 `java-coding-standard` skill）
- 模块结构、对象模型与命名约定本身（参见 `java-feature-standard` skill）
- 接口方法排序检查（参见 `java-method-ordering` skill）
- 运行时故障排查（线上问题定位、JVM 调优）

## 核心工作流

1. **阅读变更（diff）**：以本次变更为审查对象（只评变更，不通读全文件），理解业务场景与调用链，避免脱离上下文提建议
2. **逐项检查**：按“一、审查维度”逐项核对（安全 → 框架 → 质量 → 性能 → 持久层 → 日志）
3. **规范比对**：结合项目实际技术栈，参照 `java-coding-standard`、`java-feature-standard` 等 skill 判断符合性
4. **输出报告**：按“二、审查输出格式”组织结果，严重问题优先

## 一、审查维度

### 1. 安全

按以下优先级排查（完整检查点见 `references/security-checklist.md`）：

- **访问控制**：认证、权限注解、越权（水平/垂直——按 `id` 操作的接口是否校验资源归属，最高频）
- **注入类**：SQL 注入（`#{}` vs `${}`）、反序列化、日志注入、正则 ReDoS、命令注入
- **输出安全**：XSS、敏感数据脱敏与存储、超大整数 String 返回、异常信息不泄露内部细节
- **网络与资源**：SSRF、文件上传、开放重定向、防重放/防刷
- **参数校验**：外部输入有效性（pageSize / orderBy / 批量大小上限）

### 2. Spring 框架最佳实践

#### 依赖注入

- 检查是否使用构造器注入（推荐）而非字段注入
- 验证是否使用 `@RequiredArgsConstructor`（Lombok）简化构造器

**示例：**

```java
// ✅ 推荐：构造器注入
@RequiredArgsConstructor
@Service
public class UserService {
    private final UserMapper userMapper;
}

// ❌ 不推荐：字段注入（难以测试、隐藏依赖）
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
}
```

#### 事务管理

- 检查是否正确使用 `@Transactional` 注解
- 验证事务传播行为是否合理
- 检查写操作是否添加 `@Transactional(rollbackFor = Exception.class)`；只读查询是否标注 `readOnly = true`
- 检查是否存在**大事务**：事务内包含远程调用、循环、慢 IO（长时间持锁与连接，应拆分或移出事务）
- 检查**事务失效**场景：同类内部自调用（this 调用不经过代理）、非 public 方法、异常被 catch 后未手动回滚

**示例：**

```java
// ✅ 推荐
@Override
@Transactional(rollbackFor = Exception.class)
public boolean saveWithInfo(UserSavePO userSaveParam) {
    // 业务逻辑
}

// ❌ 不推荐：缺事务注解 / 缺 rollbackFor / 写操作误标 readOnly
@Override
public boolean saveWithInfo(UserSavePO userSaveParam) {
    // 业务逻辑
}
```

#### Service 层设计

- 检查 Service 接口和实现类是否分离
- 验证是否使用 `@Service` 注解
- 检查是否避免在 Service 层中直接使用 `SpringUtil.getBean()`（除非必要）

#### Controller 层设计

- 检查是否使用 `@RestController` 或 `@Controller`
- 验证是否使用 `@RequestMapping` 或其变体（`@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping` 等）
- 检查返回值是否统一使用 `Result` 包装类
- 检查 Controller 是否只做参数校验与转发，不含业务逻辑

**示例：**

```java
// ✅ 推荐
@GetMapping("getWithInfo")
public Result<UserGetWithInfoVO> getWithInfo(@RequestParam Long id) {
    Optional<UserGetWithInfoVO> userOpt = userService.getOptWithInfo(id);
    if (userOpt.isPresent()) {
        return Result.ok("Query succeeded", userOpt.get());
    }
    return Result.error("No data found");
}
```

### 3. 代码质量

#### 空指针异常（NPE）防护

- 检查是否正确使用 `Optional` 处理可能为 null 的值
- 验证是否在使用对象前进行空值检查
- 检查是否使用 `Objects.nonNull()` 或 `Objects.isNull()` 进行空值判断

**示例：**

```java
// ✅ 推荐
public Optional<UserGetWithInfoVO> getOptWithInfo(Long id) {
    return Optional.ofNullable(this.getById(id))
            .map(user -> BeanUtil.copyProperties(user, UserGetWithInfoVO.class));
}

// ❌ 不推荐：可能 NPE
public UserGetWithInfoVO getWithInfo(Long id) {
    UserPO user = this.getById(id);
    return BeanUtil.copyProperties(user, UserGetWithInfoVO.class);
}
```

#### 集合操作

- 检查是否使用 `CollUtil.isEmpty()` 或 `CollUtil.isNotEmpty()` 判断集合
- 验证是否在遍历集合前检查空值
- 检查是否使用 Stream API 进行集合操作

**示例：**

```java
// ✅ 推荐
if (CollUtil.isEmpty(userIds)) {
    return Collections.emptyList();
}
return userService.listByIds(userIds);

// ❌ 不推荐：空集合时可能出错
return userService.listByIds(userIds);
```

#### 异常处理

- 检查是否捕获了过于宽泛的异常（如 `Exception`）
- 验证是否正确处理业务异常（不吞异常、不返回 null 掩盖错误）
- 检查是否存在既记录日志又抛出的情形（记录与抛出择一，避免重复日志）
- 检查是否在适当的地方抛出自定义异常
- 检查是否通过预检查规避 `RuntimeException`（不 catch NPE / `IndexOutOfBoundsException`）
- 检查异常是否被用于流程控制（应使用条件判断）
- 检查事务场景中 catch 后是否手动回滚（否则不回滚）
- 检查 `finally` 块中是否使用 `return`（会丢弃 try 的返回值）
- 检查 RPC / 二方包 / 动态生成类调用是否 catch `Throwable` 兜底（该类调用可能抛 `NoSuchMethodError` 等 Error，仅限此场景；普通代码仍遵循上一条不宽捕）

**示例：**

```java
// ✅ 推荐：业务异常直接抛出；系统异常包装后抛出（保留 cause）——抛出时不记录日志，
// 由全局异常处理器统一记录，避免重复日志
try {
    // 业务逻辑
} catch (BusinessException e) {
    throw e;
} catch (Exception e) {
    throw new ExampleException("System error, please contact administrator", e);
}

// ✅ 推荐：可自行处理（无需向上传播）时，记录日志后不再抛出
try {
    // 业务逻辑
} catch (Exception e) {
    log.error("System error, fallback applied", e);
}

// ❌ 不推荐：既记录日志又抛出（含包装抛出），上层处理器会再记录一次，形成重复日志
try {
    // 业务逻辑
} catch (Exception e) {
    log.error("System error", e);
    throw new ExampleException("System error, please contact administrator");
}

// ❌ 不推荐：printStackTrace 不走日志框架、无上下文；吞掉异常掩盖错误
try {
    // 业务逻辑
} catch (Exception e) {
    e.printStackTrace();
}
```

#### 命名规范

- 检查变量、方法、类名是否符合 Java 命名规范
- 验证是否使用有意义的名称
- 检查是否避免使用缩写（除非是通用缩写）

**示例：**

```java
// ✅ 推荐
private final UserMapper userMapper;
public List<UserListVO> listActiveUsers();

// ❌ 不推荐
private final UserMapper ur;
public List<UserListVO> listActUsr();
```

#### 代码重复

- 检查是否有重复的代码逻辑
- 验证是否可以提取公共方法
- 检查是否可以使用工具类或公共组件

#### 方法复杂度

- 检查方法是否过长（超过 50 行）
- 验证方法是否承担单一职责
- 检查是否可以拆分复杂方法

#### 注释和文档

- 检查复杂逻辑是否有必要的注释
- 验证公共 API 是否有 Javadoc
- 检查是否避免无意义的注释与代码后缀注释

#### Java 语言陷阱

- 按 `references/java-pitfalls.md` 的 7 类陷阱清单逐类排查（数值包装类、集合、日期时间、并发锁、类设计、控制语句、其他）

#### 资源管理

- 检查流、连接、锁等资源是否释放（优先 try-with-resources；JDK 8+）
- 检查 `finally` 块中是否确保关闭（含关闭失败的兜底）
- 检查 `ThreadLocal` 是否在使用后 `remove()`（线程池复用场景易内存泄漏）

#### API 设计

- 检查方法语义与命名是否一致（返回值、副作用是否清晰）
- 验证参数设计是否合理（超过 2 个参数的查询是否封装对象、避免 Map 传参）
- 检查对外接口是否有参数校验与边界处理

#### 测试

- 检查变更是否附带单元测试；测试是否覆盖正常路径与边界（BCDE：边界/正确/设计/错误）
- 验证测试是否遵守 AIR 原则（自动化、独立、可重复）、代码是否位于 `src/test/java`
- 验证测试是否独立（不依赖执行顺序、不依赖外部环境）、是否使用断言而非打印输出
- 检查核心业务增量是否保证测试通过；覆盖率参考 70%（核心模块分支覆盖 100%）
- 检查数据库相关测试是否自动回滚、或数据带前缀标识（避免脏数据）

### 4. 性能

#### 数据库查询

- 检查是否避免 N+1 查询问题
- 验证是否使用批量操作
- 检查是否合理使用索引
- 检查循环内是否存在 DB / RPC 调用（逐条查询与远程调用，应批量或提前查询）
- 检查远程调用（HTTP / RPC）是否设置超时（无超时是故障高发根源）

#### 内存使用

- 检查是否避免创建不必要的对象
- 验证是否使用对象池或缓存
- 检查是否及时释放资源

#### 并发处理

- 检查共享可变状态：静态可变集合、静态 `SimpleDateFormat`（线程不安全）是否并发访问
- 验证并发场景的数据结构选择（`HashMap` 并发替换为 `ConcurrentHashMap`）
- 检查锁使用：加锁顺序是否一致（防死锁）、`lock()` 是否在 try 外、锁内是否避免远程调用
- 检查双重检查锁（DCL）目标字段是否 `volatile`
- 检查更新竞争是否使用乐观锁（version）或悲观锁

### 5. 持久层最佳实践（MyBatis / MyBatis-Plus / JPA）

#### Mapper XML

- 检查是否使用 `#{}` 而非 `${}`（见"1. 安全 → 注入类"）
- 验证是否使用 `resultMap` 映射复杂对象
- 检查动态 SQL 使用是否规范（`<if>` 条件遗漏、`<foreach>` 空集合处理）

#### 查询构造

- 检查是否使用 `LambdaQueryWrapper` / `Wrappers` 构建查询条件（MyBatis-Plus）；MyBatis-Flex 对应 `QueryWrapper.create()` + APT 生成的列定义（如 `ACCOUNT.ID.ge(100)`）
- 验证条件拼接是否正确处理空值

**示例：**

```java
// ✅ 推荐：Lambda 方法引用（类型安全）+ 空值条件内置 + 状态用英文字符串值
LambdaQueryWrapper<UserPO> queryWrapper = Wrappers.lambdaQuery(UserPO.class)
        .eq(UserPO::getStatus, "ACTIVE")
        .like(CharSequenceUtil.isNotEmpty(name), UserPO::getName, name);
List<UserPO> users = this.list(queryWrapper);

// ❌ 不推荐：字符串列名（重构易漏改）+ 手动判空 + 状态存数字值（见名不知意）
QueryWrapper<UserPO> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("status", 1);
if (name != null) {
    queryWrapper.like("name", name);
}
List<UserPO> users = this.list(queryWrapper);
```

### 6. 日志规范

- 检查是否使用 `@Slf4j` 注解
- 验证是否使用合适的日志级别（DEBUG, INFO, WARN, ERROR）
- 检查是否在日志中使用占位符而非字符串拼接
- 检查异常日志是否包含堆栈与上下文、不含敏感信息
- 检查 trace/debug/info 级别日志是否有开关判断（`isDebugEnabled` 等，避免昂贵的参数求值）
- 检查是否使用 `System.out` / `System.err`（生产环境禁止）
- 检查日志打印对象是否直接 JSON 序列化（对象覆写的 get 方法抛异常会中断业务流程，应打印业务属性或 toString）

**示例：**

```java
// ✅ 推荐
log.info("User login succeeded: userId={}, username={}", userId, username);
log.error("User login failed: userId={}", userId, e);

// ❌ 不推荐：字符串拼接
log.info("User login succeeded: userId=" + userId);
```

## 二、审查输出格式

### 问题分类

1. **🔴 严重问题** - 必须立即修复
   - 安全漏洞
   - 数据丢失风险
   - 系统崩溃风险

2. **⚠️ 重要问题** - 应该尽快修复
   - 性能问题
   - 代码质量问题
   - 可维护性问题

3. **💡 优化建议** - 可以改进
   - 代码风格
   - 最佳实践
   - 性能优化

> 优化建议不阻塞合并；可用 `nit:` 前缀标记纯属个人偏好的意见（对齐 Google 审查惯例）。

### 输出模板

按以下模板输出报告（`[ ]` 为占位符）：

```markdown
## 代码审查报告

### 🔴 严重问题

#### 1. [问题描述]
- **位置**: [文件路径:行号]
- **问题**: [详细说明]
- **风险**: [可能的影响]
- **建议**: [修复建议]
- **示例**: 修复前后代码对比（当前代码 → 修复后代码）

### ⚠️ 重要问题

#### 1. [问题描述]
- **位置**: [文件路径:行号]
- **问题**: [详细说明]
- **建议**: [修复建议]

### 💡 优化建议

#### 1. [建议描述]
- **位置**: [文件路径:行号]
- **说明**: [详细说明]
- **示例**: 优化前后代码对比

## 总结

- **结论**: [✅ 通过 / ⚠️ 需修改 / ❌ 需返工]
- 严重问题: X 个
- 重要问题: Y 个
- 优化建议: Z 个
```

## 三、审查检查清单

按以下顺序检查：

- [ ] 安全：越权、SQL 注入（`#{}` vs `${}`）、XSS、反序列化、日志注入、敏感信息、参数校验（见 `references/security-checklist.md`）
- [ ] Spring 框架：依赖注入（构造器优先）、事务管理（`rollbackFor`、无大事务/事务失效）、Service/Controller 分层职责
- [ ] 空指针异常防护（`Optional`、空值检查）
- [ ] 异常处理（不吞异常、不返回 null 掩盖错误、记录与抛出择一、异常含堆栈）
- [ ] 集合操作（遍历前判空、Stream 使用）
- [ ] 资源管理（流/连接/锁释放、ThreadLocal 清理）
- [ ] 命名规范（类/方法/变量、无缩写）
- [ ] Java 语言陷阱（包装类 `==`、BigDecimal、集合陷阱、日期陷阱、锁与并发陷阱等，见 `references/java-pitfalls.md`）
- [ ] API 设计（语义与命名一致、参数校验与边界）
- [ ] 测试（变更带测试、覆盖边界、测试独立、使用断言）
- [ ] 代码重复（提取公共方法/工具类）
- [ ] 方法复杂度（不超过 50 行、单一职责）
- [ ] 注释和文档（Javadoc、无尾随注释）
- [ ] 性能（N+1 查询、循环内 DB/RPC 调用、批量操作、内存）
- [ ] 并发（共享可变状态、线程安全数据结构、锁顺序、乐观锁）
- [ ] 持久层（`resultMap`、动态 SQL、Lambda 查询构造）
- [ ] 日志规范（`@Slf4j`、级别、占位符、堆栈、无敏感信息）
- [ ] 项目规范符合性（对照 `java-coding-standard` / `java-feature-standard` / `java-method-ordering` / `db-design-standard`）

## 四、注意事项

1. **提供具体的代码示例**：不要只说"这里有问题"，要给出修复前后的代码对比
2. **引用具体的代码位置**：使用文件路径和行号，方便用户定位
3. **优先级排序**：将最严重的问题放在前面
4. **建设性反馈**：不仅指出问题，还要提供解决方案
5. **考虑上下文**：理解代码的业务场景，避免过度设计
6. **遵循项目规范**：如果项目有特定的编码规范，优先遵循项目规范
7. **审查范围**：聚焦变更本身（diff），存量问题只在影响变更正确性时提及

## References（补充清单）

- `references/security-checklist.md` — 安全审查深度清单（越权、注入类、输出安全、网络与资源、参数校验）
- `references/java-pitfalls.md` — Java 语言陷阱清单（数值包装类、集合、日期时间、并发锁、类设计、控制语句）
