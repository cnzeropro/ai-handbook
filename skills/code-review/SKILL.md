---
name: "code-review"
description: "审查 Java 代码是否符合最佳实践、是否存在安全问题，以及是否遵循 Spring Framework 的规范。当用户请求审查、分析或审计代码时使用。"
---

# 代码审查

此 skill 用于全面审查 Java 代码，确保代码质量、安全性和可维护性。

## 使用场景

当用户请求以下操作时调用此 skill：
- 审查代码质量
- 分析代码安全性
- 审计代码规范
- 检查代码最佳实践
- 评估代码可维护性

## 审查流程

### 1. 代码安全检查

#### SQL 注入防护
- 检查是否使用参数化查询
- 验证 MyBatis Mapper XML 中是否使用 `${}`（危险）而非 `#{}`（安全）
- 检查 JPA/MyBatis 查询是否正确处理用户输入

**示例：**
```java
// 危险 ❌
String sql = "SELECT * FROM users WHERE name = '" + name + "'";

// 安全 ✅
@Select("SELECT * FROM users WHERE name = #{name}")
User findByName(@Param("name") String name);
```

#### XSS 防护
- 检查输出到前端的用户输入是否经过转义
- 验证是否使用 Spring 的 `@ResponseBody` 或模板引擎的自动转义

#### 敏感信息泄露
- 检查是否在日志中输出密码、token 等敏感信息
- 验证异常信息是否暴露内部实现细节

#### 认证和授权
- 检查是否有适当的权限控制注解（`@PreAuthorize`, `@Secured`）
- 验证敏感接口是否需要认证

### 2. Spring Framework 最佳实践

#### 依赖注入
- 检查是否使用构造器注入（推荐）而非字段注入
- 验证是否使用 `@RequiredArgsConstructor`（Lombok）简化构造器

**示例：**
```java
// 推荐 ✅
@RequiredArgsConstructor
@Service
public class UserService {
    private final UserRepository userRepository;
}

// 不推荐 ❌
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

#### 事务管理
- 检查是否正确使用 `@Transactional` 注解
- 验证事务传播行为是否合理
- 检查是否在需要事务的方法上添加了 `@Transactional(rollbackFor = Exception.class)`

**示例：**
```java
// 推荐 ✅
@Override
@Transactional(rollbackFor = Exception.class)
public boolean saveWithInfo(UserSaveDTO userSave) {
    // 业务逻辑
}

// 不推荐 ❌
@Override
public boolean saveWithInfo(UserSaveDTO userSave) {
    // 缺少事务注解
}
```

#### Service 层设计
- 检查 Service 接口和实现类是否分离
- 验证是否使用 `@Service` 注解
- 检查是否避免在 Service 层中直接使用 `SpringUtil.getBean()`（除非必要）

#### Controller 层设计
- 检查是否使用 `@RestController` 或 `@Controller`
- 验证是否使用 `@RequestMapping` 或其变体（`@GetMapping`, `@PostMapping` 等）
- 检查返回值是否统一使用 `Result` 包装类

**示例：**
```java
// 推荐 ✅
@GetMapping("getWithInfo")
public Result<UserDTO> getWithInfo(@RequestParam String id) {
    Optional<UserDTO> userOpt = userService.getOptWithInfo(id);
    if (userOpt.isPresent()) {
        return Result.ok("查询成功", userOpt.get());
    }
    return Result.error("暂无数据");
}
```

### 3. 代码质量检查

#### 空指针异常（NPE）防护
- 检查是否正确使用 `Optional` 处理可能为 null 的值
- 验证是否在使用对象前进行空值检查
- 检查是否使用 `Objects.nonNull()` 或 `Objects.isNull()` 进行空值判断

**示例：**
```java
// 推荐 ✅
public Optional<UserDTO> getOptWithInfo(String id) {
    return Optional.ofNullable(this.getById(id))
            .map(user -> BeanUtil.copyProperties(user, UserDTO.class));
}

// 不推荐 ❌
public UserDTO getWithInfo(String id) {
    UserPO user = this.getById(id);
    return BeanUtil.copyProperties(user, UserDTO.class); // 可能 NPE
}
```

#### 集合操作
- 检查是否使用 `CollUtil.isEmpty()` 或 `CollUtil.isNotEmpty()` 判断集合
- 验证是否在遍历集合前检查空值
- 检查是否使用 Stream API 进行集合操作

**示例：**
```java
// 推荐 ✅
if (CollUtil.isEmpty(userIds)) {
    return Collections.emptyList();
}
return userService.listByIds(userIds);

// 不推荐 ❌
return userService.listByIds(userIds); // 可能 NPE
```

#### 异常处理
- 检查是否捕获了过于宽泛的异常（如 `Exception`）
- 验证是否正确处理业务异常
- 检查是否在适当的地方抛出自定义异常

**示例：**
```java
// 推荐 ✅
try {
    // 业务逻辑
} catch (BusinessException e) {
    log.error("业务异常: {}", e.getMessage());
    throw e;
} catch (Exception e) {
    log.error("系统异常", e);
    throw new PersonaException("系统异常，请联系管理员");
}

// 不推荐 ❌
try {
    // 业务逻辑
} catch (Exception e) {
    e.printStackTrace(); // 不应该吞掉异常
}
```

### 4. 代码可读性和可维护性

#### 命名规范
- 检查变量、方法、类名是否符合 Java 命名规范
- 验证是否使用有意义的名称
- 检查是否避免使用缩写（除非是通用缩写）

**示例：**
```java
// 推荐 ✅
private final UserRepository userRepository;
public List<UserDTO> listActiveUsers();

// 不推荐 ❌
private final UserRepository ur;
public List<UserDTO> listActUsr();
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
- 验证公共 API 是否有 JavaDoc
- 检查是否避免无意义的注释

### 5. 性能优化建议

#### 数据库查询
- 检查是否避免 N+1 查询问题
- 验证是否使用批量操作
- 检查是否合理使用索引

#### 内存使用
- 检查是否避免创建不必要的对象
- 验证是否使用对象池或缓存
- 检查是否及时释放资源

#### 并发处理
- 检查是否正确处理并发场景
- 验证是否使用线程安全的数据结构
- 检查是否避免死锁

### 6. MyBatis/MyBatis-Plus 最佳实践

#### Mapper XML
- 检查是否使用 `#{}` 而非 `${}`
- 验证是否使用 `resultMap` 映射复杂对象
- 检查是否使用动态 SQL（`<if>`, `<foreach>` 等）

#### Service 层
- 检查是否使用 `IService` 和 `ServiceImpl`
- 验证是否使用 `LambdaQueryWrapper` 构建查询条件
- 检查是否使用 `Wrappers` 工具类

**示例：**
```java
// 推荐 ✅
LambdaQueryWrapper<UserPO> queryWrapper = Wrappers.lambdaQuery(UserPO.class)
        .eq(UserPO::getStatus, 1)
        .like(CharSequenceUtil.isNotEmpty(name), UserPO::getName, name);
List<UserPO> users = this.list(queryWrapper);

// 不推荐 ❌
QueryWrapper<UserPO> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("status", 1);
if (name != null) {
    queryWrapper.like("name", name);
}
List<UserPO> users = this.list(queryWrapper);
```

### 7. 日志规范

- 检查是否使用 `@Slf4j` 注解
- 验证是否使用合适的日志级别（DEBUG, INFO, WARN, ERROR）
- 检查是否在日志中使用占位符而非字符串拼接

**示例：**
```java
// 推荐 ✅
log.info("用户登录成功: userId={}, username={}", userId, username);
log.error("处理失败: {}", e.getMessage(), e);

// 不推荐 ❌
log.info("用户登录成功: userId=" + userId + ", username=" + username);
log.error("处理失败", e);
```

## 审查输出格式

### 问题分类

使用以下分类组织审查结果：

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

### 输出模板

```markdown
## 代码审查报告

### 🔴 严重问题

#### 1. [问题描述]
**位置**: [文件路径:行号]
**问题**: [详细说明]
**风险**: [可能的影响]
**建议**: [修复建议]
**示例**:
```java
// 当前代码
// 修复后的代码
```

### ⚠️ 重要问题

#### 1. [问题描述]
**位置**: [文件路径:行号]
**问题**: [详细说明]
**建议**: [修复建议]

### 💡 优化建议

#### 1. [建议描述]
**位置**: [文件路径:行号]
**说明**: [详细说明]
**示例**:
```java
// 当前代码
// 优化后的代码
```

## 总结

- 严重问题: X 个
- 重要问题: Y 个
- 优化建议: Z 个
```

## 审查检查清单

在审查代码时，请按以下顺序检查：

- [ ] 安全漏洞（SQL 注入、XSS、敏感信息泄露）
- [ ] Spring Framework 最佳实践（依赖注入、事务管理、Service/Controller 设计）
- [ ] 空指针异常防护
- [ ] 异常处理
- [ ] 集合操作
- [ ] 命名规范
- [ ] 代码重复
- [ ] 方法复杂度
- [ ] 注释和文档
- [ ] 性能优化（数据库查询、内存使用、并发处理）
- [ ] MyBatis/MyBatis-Plus 最佳实践
- [ ] 日志规范

## 注意事项

1. **提供具体的代码示例**：不要只说"这里有问题"，要给出修复前后的代码对比
2. **引用具体的代码位置**：使用文件路径和行号，方便用户定位
3. **优先级排序**：将最严重的问题放在前面
4. **建设性反馈**：不仅指出问题，还要提供解决方案
5. **考虑上下文**：理解代码的业务场景，避免过度设计
6. **遵循项目规范**：如果项目有特定的编码规范，优先遵循项目规范
