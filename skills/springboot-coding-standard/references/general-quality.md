# Spring Boot 通用质量规范

> 本文件为 springboot-coding-standard 的**版本通用**工程规约（Boot 2.x 与 3.x 通用部分，示例以 3.x 语法呈现）。版本特有约定见分册：Boot 3.x 见 `springboot3.md`，Boot 2.x 遗留见 `springboot-legacy.md`。来源：Spring Boot 官方参考文档（docs.spring.io/spring-boot）。完整正反例见 `style-examples.md`（同目录）。被测与业务代码写法规约见 `java-coding-standard` / `java-feature-standard`。

## 目录

- 一、配置管理
- 二、依赖注入与 Bean 管理
- 三、全局异常处理
- 四、Web 层约定
- 五、缓存
- 六、异步与调度
- 七、Actuator 与可观测
- 八、安全基线

## 一、配置管理

### 1. @ConfigurationProperties 优于散落 @Value

类型安全绑定 + 启动期校验 + IDE 元数据支持；`@Value` 仅限单点简单值：

```java
// ❌ 散落 @Value：来源分散、无校验、难追踪
@Service
public class ModelService {
    @Value("${model.upload.max-size}") private long maxSize;
    @Value("${model.upload.allowed-types}") private List<String> types;
}

// ✅ 集中绑定 + 启动期校验
@Validated
@ConfigurationProperties(prefix = "model.upload")
public record ModelUploadProperties(
    @NotNull @Positive Long maxSize,
    @NotEmpty List<String> allowedTypes
) {}
```

- 属性类放独立文件并注册（`@EnableConfigurationProperties(ModelUploadProperties.class)` 或 `@ConfigurationPropertiesScan`）
- 配置按 profile 分层：`application.yml` 放公共，`application-{profile}.yml` 放环境差异；禁在代码里按环境 if-else
- 秘密走环境变量 / 配置中心注入，禁进代码仓库

### 2. 配置项校验

启动即失败（fail fast）：`@Validated` + Jakarta Validation 注解，缺失或非法配置在启动期报错，而非运行期 NPE。

## 二、依赖注入与 Bean 管理

- **构造器注入为默认**（官方推荐）：依赖显式、利于测试与不可变；单构造器可省略 `@Autowired`
- **字段注入禁用**（`@Autowired` 直接标字段）：隐藏依赖、无法构造测试
- `@Configuration(proxyBeanMethods = false)`：无 Bean 间互调时关闭代理提性能
- `@Component` 扫描边界收敛：应用根包；禁扫描到三方库包
- **循环依赖出现即设计坏味道**：`@Lazy` / setter 注入 / 静态上下文都是掩盖——回到职责拆分（官方已默认禁止循环依赖，Boot 2.6+）

## 三、全局异常处理

`@RestControllerAdvice` 统一错误响应，禁各 Controller 自行 try-catch 拼错误体：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // ✅ 业务异常 → 统一错误码
    @ExceptionHandler(BizException.class)
    ResponseEntity<ErrorResponse> handleBiz(BizException e) {
        return ResponseEntity.status(e.getStatus())
            .body(new ErrorResponse(e.getCode(), e.getMessage(), traceId()));
    }

    // ✅ 校验异常：聚合字段错误
    @ExceptionHandler(MethodArgumentNotValidException.class)
    ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException e) { … }

    // ✅ 兜底：未知异常记录日志（含堆栈），对外只给通用提示 + traceId
    @ExceptionHandler(Exception.class)
    ResponseEntity<ErrorResponse> handleUnknown(Exception e) {
        log.error("Unhandled exception", e);
        return ResponseEntity.internalServerError()
            .body(new ErrorResponse("INTERNAL_ERROR", "系统繁忙，请稍后重试", traceId()));
    }
}
```

- 禁吞异常；**禁把堆栈与内部消息返回给客户端**（信息泄露）
- 错误响应结构全局统一（错误码 / 消息 / traceId），与前端约定对齐

## 四、Web 层约定

- 参数校验：`@Valid` + `@Validated` 分组校验；`@PathVariable` / `@RequestParam` 标 `@Validated` 后类级约束生效
- 状态码语义化：200 查询成功、201 创建、400 客户端错误、404 不存在、409 冲突、500 兜底——禁全部 200 包错误体（除非项目有既定包装约定且全局一致）
- 分页参数统一（page/size 封装为分页对象，size 上限保护）
- API 版本：URI 前缀（`/api/v1`）或 Header，项目级统一一种
- Controller 保持薄：只做参数绑定与委派，业务在 Service（详见 `java-feature-standard` 分层规范）

## 五、缓存

- 声明式缓存 `@Cacheable` / `@CacheEvict` / `@CachePut`；**key 必须显式且含全部区分维度**（禁裸 `@Cacheable` 无 key）
- TTL 必设（Redis 缓存管理器统一配置），禁永不过期（除非明确只读字典）
- 一致性约定：**先更新库，后删缓存**；禁"更新库 + 更新缓存"双写（并发写覆盖）

```java
// ❌ 无 key、无 TTL
@Cacheable("models")
public Model getById(Long id) { … }

// ✅
@Cacheable(cacheNames = "models", key = "#id")
public Model getById(Long id) { … }
```

## 六、异步与调度

- `@Async` 必须自定义线程池（默认 `SimpleAsyncTaskExecutor` 每任务新建线程，不复用）；标注线程池名
- `@Scheduled` 任务线程池默认单线程——多任务需池化配置；任务体必须**幂等**（多实例部署配合分布式锁，如 ShedLock）
- 优雅停机：`server.shutdown: graceful` + 生命周期超时配置；`@PreDestroy` 释放资源
- 禁在异步线程使用 `HttpServletRequest` 等请求绑定对象（需显式传递拷贝）

```java
// ❌ 默认执行器 + 非幂等任务
@Async
@Scheduled(cron = "0 * * * * *")
void syncModels() { … }

// ✅
@Async("bizExecutor")
@Scheduled(cron = "0 0 * * * *")
@SchedulerLock(name = "syncModels", lockAtMostFor = "PT10M")
void syncModels() { … }
```

## 七、Actuator 与可观测

- 暴露端点**最小化**：`management.endpoints.web.exposure.include: health,info,metrics`；prometheus 端点按需并走内网/鉴权
- 禁暴露 `env` / `beans` / `heapdump` 到非受信网络（信息泄露）
- 健康检查分组（liveness / readiness，K8s 场景）
- 指标命名 Micrometer 规范（`.` 分隔、小写）；日志含 traceId（MDC），与错误响应 traceId 对齐排障

## 八、安全基线

- Spring Security 起步配置：端点**默认拒绝、显式放行**（白名单：健康检查、登录、静态资源）
- 密码必须编码存储：`BCryptPasswordEncoder`（或项目统一方案），禁明文 / MD5 / SHA1
- CSRF：浏览器表单开启，纯 API（无状态 token）可关闭并注明
- 细粒度权限、OAuth2 资源服务器配置：跟随项目安全方案（本 skill 只约束基线）
