# Spring Boot 编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：Spring Boot 官方参考文档与工程审查常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. 配置管理完整示例（@ConfigurationProperties record + 校验）
- 2. 全局异常处理完整示例（@RestControllerAdvice）
- 3. 条件装配示例（starter 风格）
- 4. 缓存规约示例
- 5. 异步与调度（自定义线程池 + 分布式锁）
- 6. Actuator 暴露与安全
- 7. 循环依赖与 DI 正反例
- 8. 常见正反例速查

## 1. 配置管理完整示例（@ConfigurationProperties record + 校验）

```yaml
# application.yml
model:
  upload:
    max-size: 10MB
    allowed-types: jpg, png, webp
```

```java
@Validated
@ConfigurationProperties(prefix = "model.upload")
public record ModelUploadProperties(
    @NotNull
    @Positive
    DataSize maxSize,                                  // DataSize 直接解析 "10MB"

    @NotEmpty
    List<@Pattern(regexp = "[a-z]+") String> allowedTypes
) {
    public ModelUploadProperties {
        allowedTypes = List.copyOf(allowedTypes);      // 不可变化
    }
}

// 注册：应用启动类或配置类
@ConfigurationPropertiesScan
@SpringBootApplication
public class Application { … }
```

使用方构造注入：

```java
@Service
public class ModelUploadService {
    private final ModelUploadProperties properties;

    public ModelUploadService(ModelUploadProperties properties) {   // ✅ 构造注入
        this.properties = properties;
    }
}
```

❌ 反例：散落 `@Value`（来源分散、无校验、魔法默认值内联）。

## 2. 全局异常处理完整示例（@RestControllerAdvice）

```java
// ✅ 统一错误体：错误码 + 消息 + traceId
public record ErrorResponse(String code, String message, String traceId) {}

@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    // 业务异常：携带错误码与 HTTP 状态
    @ExceptionHandler(BizException.class)
    public ResponseEntity<ErrorResponse> handleBiz(BizException e) {
        return ResponseEntity.status(e.getStatus())
            .body(new ErrorResponse(e.getCode(), e.getMessage(), currentTraceId()));
    }

    // 校验异常：聚合字段错误返回
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException e) {
        var fields = e.getBindingResult().getFieldErrors().stream()
            .map(f -> new FieldErrorItem(f.getField(), f.getDefaultMessage()))
            .toList();
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("VALIDATION_FAILED", "参数校验失败", currentTraceId()));
    }

    // 兜底：记日志（含堆栈）但对客户端隐藏内部信息
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleUnknown(Exception e) {
        log.error("Unhandled exception", e);
        return ResponseEntity.internalServerError()
            .body(new ErrorResponse("INTERNAL_ERROR", "系统繁忙，请稍后重试", currentTraceId()));
    }
}
```

❌ 反例：catch 后只打 `e.getMessage()`（丢堆栈）、把 `e.getStackTrace()` 返回给客户端、每个 Controller 各自 catch。

## 3. 条件装配示例（starter 风格）

```java
// ✅ 基础设施自动装配：按配置开关，缺省关闭
@AutoConfiguration
@ConditionalOnProperty(prefix = "app.audit", name = "enabled", havingValue = "true")
public class AuditAutoConfiguration {

    @Bean
    AuditRecorder auditRecorder(AuditProperties properties, AuditSink sink) {
        return new DbAuditRecorder(properties, sink);
    }
}
```

```java
// ❌ 业务代码里的条件装配：编译期不可见、运行期行为漂移
@Service
@ConditionalOnProperty(name = "feature.export", havingValue = "true")
class ExportService { … }
```

业务开关走配置读取或策略路由，不把 Bean 装配当业务开关。

## 4. 缓存规约示例

```java
@CacheConfig(cacheNames = "models")
public class ModelService {

    // ✅ key 显式含全部区分维度
    @Cacheable(key = "#id")
    public Model getById(Long id) { … }

    // ✅ 写操作：先更库后删缓存
    @CacheEvict(key = "#model.id")
    public void update(Model model) { … }
}
```

TTL 统一在缓存管理器配置（Redis：`entryTtl` 按缓存名分组），禁裸无过期缓存。

## 5. 异步与调度（自定义线程池 + 分布式锁）

```java
// ✅ 自定义业务线程池（v4/3.x 配置示例）
@Configuration
class AsyncConfig {
    @Bean("bizExecutor")
    ThreadPoolTaskExecutor bizExecutor() {
        var executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(8);
        executor.setMaxPoolSize(16);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("biz-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        return executor;
    }
}

// ✅ 指定线程池
@Async("bizExecutor")
void sendNotification(Long userId) { … }
```

```java
// ✅ 多实例部署：调度任务加分布式锁（ShedLock 示例）
@Scheduled(cron = "0 0 2 * * *")
@SchedulerLock(name = "daily-settlement", lockAtMostFor = "PT10M")
void dailySettlement() { … }
```

❌ 反例：`@EnableScheduling` 后裸 `@Scheduled`（多实例重复执行）；`@Async` 用默认执行器。

## 6. Actuator 暴露与安全

```yaml
# ✅ 最小暴露 + 健康分组 + 独立管理端口
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true
  server:
    port: 9090                       # 管理端口与业务端口分离（内网）
```

❌ 反例：`include: "*"` 全暴露（env/heapdump 信息泄露）；管理端口公网可达。

## 7. 循环依赖与 DI 正反例

```java
// ❌ 构造器循环依赖（Boot 2.6+ 默认启动失败）
@Service
class OrderService {
    private final InventoryService inventoryService;   // 互相依赖
    OrderService(InventoryService inventoryService) { … }
}
@Service
class InventoryService {
    private final OrderService orderService;
    InventoryService(OrderService orderService) { … }
}

// ✅ 职责拆分：共享逻辑下沉第三服务，或事件解耦
@Service
class OrderService {
    private final StockDomainService stockDomainService;   // 只依赖需要的领域逻辑
}

// ❌ @Lazy 掩盖循环依赖（未解决设计问题，仅推迟爆炸）
```

## 8. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 配置散落 | 到处 `@Value` | `@ConfigurationProperties` 集中绑定 |
| 配置无校验 | 启动不校验 | `@Validated` + JSR-303 注解 fail fast |
| 字段注入 | `@Autowired` 标字段 | 构造器注入 |
| 循环依赖 | `@Lazy` 掩盖 | 职责拆分 / 事件解耦 |
| 异常各自处理 | Controller 内 catch 拼错误体 | `@RestControllerAdvice` 统一 |
| 堆栈外泄 | `e.getMessage()`/堆栈进响应 | 通用提示 + traceId，堆栈进日志 |
| @Async 默认池 | 裸 `@Async` | 自定义命名线程池 |
| 多实例重复调度 | 裸 `@Scheduled` | 池化 + 分布式锁（ShedLock） |
| 缓存无 TTL / 无 key | 裸 `@Cacheable("x")` | 显式 key + 统一 TTL |
| Actuator 全暴露 | `include: "*"` | 最小暴露 + 管理端口分离 |
| 密码明文/MD5 | 自写摘要 | BCrypt（PasswordEncoder） |
| @MockBean 泛滥 | 全量 mock 上下文 | 构造真实例；mock 仅边界 |
