# 编码风格完整示例

> 本文件为 `../SKILL.md` 的补充示例。核心规则见主文件“二、注释规范”至“九、工具类使用”。
> 示例业务名（Model 等）为占位，实际开发替换为项目自身业务名。

## 1. 注释规范完整示例

```java
/**
 * 模型服务实现类
 *
 * @author example
 * @date 2026/10/04
 */
@Service
public class ModelServiceImpl implements ModelService {

    /**
     * 根据编号获取模型详情
     *
     * @param id 模型编号
     * @return 模型视图对象
     */
    @Override
    public ModelGetByIdVO getById(Long id) {
        // 校验编号是否为空
        if (Objects.isNull(id)) {
            throw new ExampleException("Model id must not be null");
        }

        /*
         * 查询数据库
         * 如果不存在则抛出异常
         */
        ModelPO model = this.getOne(id);

        return BeanUtil.copyProperties(model, ModelGetByIdVO.class);
    }
}
```

**正反例：**

```java
// ❌ 错误：代码后缀注释
return BeanUtil.copyProperties(model, ModelGetByIdVO.class); // 转换为视图对象

// ❌ 错误：类、方法缺少 Javadoc
@Service
public class ModelServiceImpl implements ModelService {
    public ModelGetByIdVO getById(Long id) { ... }
}

// ✅ 正确：注释独立成行，类与方法均有 Javadoc
/**
 * 模型服务实现类
 *
 * @author example
 * @date 2026/10/04
 */
@Service
public class ModelServiceImpl implements ModelService {

    /**
     * 根据编号获取模型
     */
    @Override
    public ModelGetByIdVO getById(Long id) {
        // 转换为视图对象
        return BeanUtil.copyProperties(this.getOne(id), ModelGetByIdVO.class);
    }
}
```

## 2. 导入顺序完整示例

```java
// 1. JDK 标准库
import java.util.List;
import java.util.stream.Collectors;

// 2. 第三方库
import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.collection.CollUtil;
import com.baomidou.mybatisplus.core.metadata.IPage;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.RequestMapping;

// 3. 项目内部
import com.example.module.model.model.persistent.ModelPO;
import com.example.module.model.service.ModelService;
```

**正反例：**

```java
// ❌ 错误：导入乱序混排
import com.example.module.model.service.ModelService;
import java.util.List;
import org.springframework.web.bind.annotation.RequestMapping;
import cn.hutool.core.bean.BeanUtil;

// ✅ 正确：JDK → 第三方 → 项目内部
import java.util.List;
import cn.hutool.core.bean.BeanUtil;
import org.springframework.web.bind.annotation.RequestMapping;
import com.example.module.model.service.ModelService;
```

## 3. 代码格式完整示例

```java
@Service
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {

    private final ModelJobService modelJobService;

    @Override
    public ModelGetWithUserVO getWithUserById(Long id) {
        return this.getOptById(id).map(model -> BeanUtil.copyProperties(model, ModelGetWithUserVO.class))
                .map(model -> {
                    UserInfoUtil.setCreator(model);
                    return model;
                })
                .orElse(null);
    }

    protected Optional<ModelPO> getOptById(Long id) {
        return Optional.ofNullable(this.getById(id));
    }
}
```

## 4. 枚举与常量完整示例

### 枚举

```java
/**
 * 模型状态：WAITING（等待中）、STAGED（已暂存）、DEPLOYED（已部署）
 */
@Getter
public enum ModelStatus {
    WAITING,
    STAGED,
    DEPLOYED,
}
```

**使用方式：**

```java
// 枚举 → 字符串（存储/传输），见名知意
String status = ModelStatus.WAITING.name();           // "WAITING"

// 字符串 → 枚举
ModelStatus status = ModelStatus.valueOf(statusStr);
```

**正反例：**

```java
// ❌ 错误：数字值存储（冗余 int 成员变量 + DB 从 1 编号），见名不知意
@Getter
public enum ModelStatus {
    WAITING(1),
    STAGED(2),
    DEPLOYED(3);

    private final int value;

    ModelStatus(int value) {
        this.value = value;
    }
}
model.setStatus(1);                                    // 需查文档才知道 1 的含义

// ✅ 正确：枚举名即字符串值，见名知意
@Getter
public enum ModelStatus {
    WAITING,
    STAGED,
    DEPLOYED,
}
model.setStatus(ModelStatus.WAITING.name());           // "WAITING"
```

### 常量

```java
public class ModelConstants {
    public static final String MODEL_TYPE_PREDICTION = "prediction";
    public static final String MODEL_TYPE_CLASSIFICATION = "classification";
    public static final Integer MODEL_TYPE_PREDICTION_VALUE = 1;
    public static final Integer MODEL_TYPE_CLASSIFICATION_VALUE = 2;
}
```

**正反例：**

```java
// ❌ 错误：魔法值散落在业务代码中
if ("prediction".equals(model.getType())) { ... }
model.setStatus(1);

// ✅ 正确：使用常量
if (ModelConstants.MODEL_TYPE_PREDICTION.equals(model.getType())) { ... }
model.setStatus(ModelStatus.STAGED.name());
```

## 5. 异常处理完整示例

### 自定义异常

```java
public class ExampleException extends RuntimeException {
    public ExampleException(String message) {
        super(message);
    }

    public ExampleException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Service 层抛业务异常

```java
// Service 层：抛出业务异常
public void deployById(Long id) {
    ModelPO model = this.getOptById(id)
            .map(m -> {
                m.setStatus(ModelStatus.DEPLOYED.name());
                return m;
            })
            .orElseThrow(() -> new ExampleException("Model not found"));
    this.updateById(model);
}
```

**正反例：**

```java
// ❌ 错误：Service 层吞掉异常返回 null，调用方无法区分"不存在"与"系统错误"
public ModelPO getById(Long id) {
    try {
        return this.getById(id);
    } catch (Exception e) {
        return null;
    }
}

// ❌ 错误：Controller 层堆叠 try-catch，逐接口重复处理
@GetMapping("getById")
public Result<ModelGetByIdVO> getById(@RequestParam Long id) {
    try {
        ModelGetByIdVO model = modelService.getById(id);
        return Result.ok(model);
    } catch (Exception e) {
        return Result.error("Failed to query model");
    }
}

// ✅ 正确：Service 抛业务异常，Controller 简洁，交由全局异常处理器兜底
@GetMapping("getById")
public Result<ModelGetByIdVO> getById(@RequestParam Long id) {
    ModelGetByIdVO model = modelService.getById(id);
    if (Objects.isNull(model)) {
        return Result.error("No data found");
    }
    return Result.ok("Query succeeded", model);
}

// ❌ 错误：既记录日志又抛出（含包装抛出），上层处理器会再记录一次，形成重复日志
try {
    // ...
} catch (Exception e) {
    log.error("Deploy model failed: modelId={}", id, e);
    throw new ExampleException("Deploy failed");
}

// ✅ 正确：记录与抛出择一——抛出（含包装，保留 cause）时不记录日志，由全局异常处理器统一记录
try {
    // ...
} catch (Exception e) {
    throw new ExampleException("Deploy failed: modelId=" + id, e);
}
```

## 6. 日志完整示例

```java
@Slf4j
@Service
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {

    @Override
    public void run(Long id) {
        log.info("Start running model job: jobId={}", id);
        try {
            this.getOptById(id).map(ModelPO::getCode)
                    .ifPresent(code -> {
                        JobClientUtil.sendRunRequest(code);
                        log.info("Model job ran successfully: jobId={}", id);
                    });
        } catch (Exception e) {
            // 抛出时不记录日志（记录与抛出择一），由全局异常处理器统一记录；包装时保留 cause
            throw new ExampleException("Model job failed: jobId=" + id, e);
        }
    }
}
```

**正反例：**

```java
// ❌ 错误：字符串拼接（即使不输出也计算成本）、异常无堆栈、无上下文
log.info("Model job ran successfully: " + id);
try {
    // ...
} catch (Exception e) {
    log.error("Execution failed");                      // 无堆栈、无上下文
}

// ❌ 错误：日志与异常信息使用中文（应使用英文）
log.info("模型任务执行成功：{}", id);

// ❌ 错误：敏感信息直接入日志
log.info("login success: password={}", password);

// ✅ 正确：占位符 + 上下文 + 堆栈
log.info("Model job ran successfully: jobId={}", id);
log.error("Model job failed: jobId={}", id, e);
```

## 7. 工具类使用示例

### Hutool 工具类

```java
// 对象拷贝
ModelGetWithUserVO modelVO = BeanUtil.copyProperties(modelPO, ModelGetWithUserVO.class);

// 集合操作
if (CollUtil.isEmpty(userIds)) {
    return Collections.emptyList();
}

// 字符串操作
if (CharSequenceUtil.isNotBlank(code)) {
    queryWrapper.like(ModelPO::getCode, code);
}

// 数组操作
if (ArrayUtil.isEmpty(ids)) {
    return true;
}

// 日期操作
LocalDateTime startTime = LocalDateTimeUtil.beginOfDay(param.getStartTime());
LocalDateTime endTime = LocalDateTimeUtil.endOfDay(param.getEndTime());

// JSON 操作
JSONObject config = JSONUtil.parseObj(configStr);
String modelId = config.getStr("modelId");

// Optional 处理（JDK 标准库）
return Optional.ofNullable(this.getById(id))
        .map(model -> BeanUtil.copyProperties(model, ModelGetWithUserVO.class))
        .orElse(null);
```

### Guava 工具类

```java
// 集合创建与不可变集合
List<String> list = Lists.newArrayList("a", "b", "c");
List<String> immutableList = ImmutableList.of("a", "b", "c");
Map<String, Object> map = Maps.newHashMapWithExpectedSize(16);

// 字符串拆分与连接
List<String> parts = Splitter.on(',').omitEmptyStrings().splitToList(input);
String joined = Joiner.on(",").join(parts);

// 参数校验
Preconditions.checkNotNull(id, "id must not be null");
Preconditions.checkArgument(amount > 0, "amount must be positive");

// 字符串判空处理
String name = Strings.nullToEmpty(user.getName());
```

### Apache Commons 工具类

```java
// 字符串操作（commons-lang3）
if (StringUtils.isNotBlank(code)) {
    // ...
}
String joined = StringUtils.join(parts, ",");

// 集合操作（commons-collections4）
if (CollectionUtils.isNotEmpty(list)) {
    // ...
}

// 文件与流操作（commons-io）
String content = FileUtils.readFileToString(file, StandardCharsets.UTF_8);
IOUtils.copy(inputStream, outputStream);
```

### Spring 自带工具类

```java
// 字符串与集合判空（Spring 版）
if (StringUtils.hasText(name)) {
    // ...
}
if (CollectionUtils.isEmpty(list)) {
    // ...
}

// 断言与耗时统计
Assert.notNull(id, "id must not be null");
StopWatch stopWatch = new StopWatch();
stopWatch.start("query");
// ...
stopWatch.stop();
```

### 其他常用组件

```java
// JSON（Jackson；Spring 项目注入共享单例 ObjectMapper，复用模块配置）
ObjectMapper objectMapper = new ObjectMapper();
ModelDTO dto = objectMapper.readValue(jsonStr, ModelDTO.class);

// 本地缓存（Caffeine）
Cache<String, Object> cache = Caffeine.newBuilder()
    .maximumSize(10000)
    .expireAfterWrite(Duration.ofMinutes(10))
    .build();

// 对象映射（MapStruct，编译期生成，类型安全）
@Mapper
public interface ModelConverter {
    ModelConverter INSTANCE = Mappers.getMapper(ModelConverter.class);

    ModelDTO toDTO(ModelPO po);
}
```

### 项目工具类（示例，实际以项目内已有工具类为准）

```java
// 编码生成
String code = CodeUtil.createCode(CodeUtil.CodeType.MODEL);

// 用户信息设置
UserInfoUtil.setCreator(models);

// 集合工具
List<Long> ids = CollectionUtil.mapDistinct(models, ModelGetWithUserVO::getId);

// 任务调度客户端
JobClientUtil.sendRunRequest(jobCode);
```
