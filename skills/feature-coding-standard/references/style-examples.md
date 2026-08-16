# 注解与代码风格完整示例

> 本文件为 `../SKILL.md` 的补充示例。核心规则见主文件“四、注解与代码风格”至“九、工具类使用”。

## 1. Spring 注解完整示例

```java
// Controller
@RestController
@RequestMapping("model")
public class ModelController {
}

// Service 接口
public interface ModelService extends IService<ModelPO> {
}

// Service 实现
@Service
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {
}

// Mapper
public interface ModelMapper extends BaseMapper<ModelPO> {
}

// 事务
@Transactional(rollbackFor = Exception.class)
public boolean saveWithJob(ModelSaveWithJobPO modelSaveWithJobParam) {
}

// 只读事务
@Transactional(readOnly = true)
public Collection<ModelPO> list(ModelListPO modelListParam, Query query) {
}
```

**字段注入正反例：**

```java
// ❌ 错误：字段注入（难以测试、隐藏依赖）
@RestController
public class ModelController {
    @Autowired
    private ModelService modelService;
}

// ✅ 正确：构造器注入
@RequiredArgsConstructor
@RestController
public class ModelController {
    private final ModelService modelService;
}
```

## 2. 注释规范完整示例

```java
/**
 * 模型服务实现类
 *
 * @author example
 */
@Service
public class ModelServiceImpl implements ModelService {

    /**
     * 根据 ID 获取模型详情
     *
     * @param id 模型 ID
     * @return 模型视图对象
     */
    @Override
    public ModelGetByIdVO getById(String id) {
        // 校验 ID 是否为空
        if (StrUtil.isBlank(id)) {
            return null;
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
    public ModelGetByIdVO getById(String id) { ... }
}

// ✅ 正确：注释独立成行，类与方法均有 Javadoc
/**
 * 模型服务实现类
 */
@Service
public class ModelServiceImpl implements ModelService {

    /**
     * 根据 ID 获取模型
     */
    @Override
    public ModelGetByIdVO getById(String id) {
        // 转换为视图对象
        return BeanUtil.copyProperties(this.getOne(id), ModelGetByIdVO.class);
    }
}
```

## 3. 导入顺序完整示例

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

## 4. 代码格式完整示例

```java
@Service
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {

    private final ModelJobService modelJobService;

    @Override
    public ModelGetWithUserVO getWithUserById(String id) {
        return this.getOptById(id).map(model -> BeanUtil.copyProperties(model, ModelGetWithUserVO.class))
                .map(model -> {
                    UserInfoUtil.setCreator(model);
                    return model;
                })
                .orElse(null);
    }

    protected Optional<ModelPO> getOptById(String id) {
        return Optional.ofNullable(this.getById(id));
    }
}
```

## 5. 枚举与常量完整示例

### 枚举

```java
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
    public static final String MODEL_TYPE_LOOKALIKE = "lookalike";
    public static final String MODEL_TYPE_RS = "rs";
    public static final Integer MODEL_TYPE_LOOKALIKE_VALUE = 1;
    public static final Integer MODEL_TYPE_RS_VALUE = 2;
}
```

**正反例：**

```java
// ❌ 错误：魔法值散落在业务代码中
if ("lookalike".equals(model.getType())) { ... }
model.setStatus(1);

// ✅ 正确：使用常量
if (ModelConstants.MODEL_TYPE_LOOKALIKE.equals(model.getType())) { ... }
model.setStatus(ModelStatus.STAGED.name());
```

## 6. 异常处理完整示例

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
public void enableById(String id) {
    ModelPO model = this.getOptById(id)
            .map(m -> {
                m.setStatus(true);
                return m;
            })
            .orElseThrow(() -> new ExampleException("Model not found"));
    this.updateById(model);
}
```

**正反例：**

```java
// ❌ 错误：Service 层吞掉异常返回 null，调用方无法区分"不存在"与"系统错误"
public ModelPO getById(String id) {
    try {
        return this.getById(id);
    } catch (Exception e) {
        return null;
    }
}

// ❌ 错误：Controller 层堆叠 try-catch，逐接口重复处理
@GetMapping("getById")
public Result<ModelGetByIdVO> getById(@RequestParam String id) {
    try {
        ModelGetByIdVO model = modelService.getById(id);
        return Result.ok(model);
    } catch (Exception e) {
        return Result.error("Failed to query model");
    }
}

// ✅ 正确：Service 抛业务异常，Controller 简洁，交由全局异常处理器兜底
@GetMapping("getById")
public Result<ModelGetByIdVO> getById(@RequestParam String id) {
    ModelGetByIdVO model = modelService.getById(id);
    if (Objects.isNull(model)) {
        return Result.error("No data found");
    }
    return Result.ok("Query succeeded", model);
}
```

## 7. 日志完整示例

```java
@Slf4j
@Service("modelService")
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {

    @Override
    public void run(String id) {
        log.info("Start running model job: jobId={}", id);
        try {
            this.getOptById(id).map(ModelPO::getCode)
                    .ifPresent(code -> {
                        JobClientUtil.sendRunRequest(code);
                        log.info("Model job ran successfully: jobId={}", id);
                    });
        } catch (Exception e) {
            log.error("Model job failed: jobId={}", id, e);
            throw new ExampleException("Model job failed");
        }
    }
}
```

**正反例：**

```java
// ❌ 错误：字符串拼接（即使不输出也计算成本）、异常无堆栈、无上下文
log.info("模型任务执行成功: " + id);
try {
    // ...
} catch (Exception e) {
    log.error("执行失败");                      // 无堆栈、无上下文
}

// ❌ 错误：敏感信息直接入日志
log.info("login success: password={}", password);

// ✅ 正确：占位符 + 上下文 + 堆栈
log.info("Model job ran successfully: jobId={}", id);
log.error("Model job failed: jobId={}", id, e);
```

## 8. 事务管理完整示例

```java
@Override
@Transactional(rollbackFor = Exception.class)
public boolean saveWithInfo(ModelSavePO param) {
    ModelPO model = BeanUtil.copyProperties(param, ModelPO.class);
    String code = CodeUtil.createCode(CodeUtil.CodeType.MODEL);
    model.setCode(code);
    boolean saved = super.save(model);
    return saved;
}

@Override
@Transactional(readOnly = true)
public Collection<ModelPO> list(ModelListPO param, Query query) {
    // 查询逻辑
}
```

**正反例：**

```java
// ❌ 错误：不指定 rollbackFor，受检异常与 RuntimeException 之外的异常不触发回滚
@Transactional
public boolean saveWithInfo(ModelSavePO param) { ... }

// ❌ 错误：写操作标注 readOnly，事务无效或行为异常
@Transactional(readOnly = true)
public boolean removeById(String id) { ... }

// ✅ 正确
@Transactional(rollbackFor = Exception.class)
public boolean saveWithInfo(ModelSavePO param) { ... }

@Transactional(readOnly = true)
public ModelGetByIdVO getById(String id) { ... }
```

## 9. 工具类使用完整示例

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
Date startTime = DateUtil.beginOfDay(param.getStartTime());
Date endTime = DateUtil.endOfDay(param.getEndTime());

// JSON 操作
JSONObject config = JSONUtil.parseObj(configStr);
String modelId = config.getStr("model_id");

// Optional 处理（JDK 标准库）
return Optional.ofNullable(this.getById(id))
        .map(model -> BeanUtil.copyProperties(model, ModelGetWithUserVO.class))
        .orElse(null);
```

### 项目工具类（示例，实际以项目内已有工具类为准）

```java
// 编码生成
String code = CodeUtil.createCode(CodeUtil.CodeType.MODEL);

// 用户信息设置
UserInfoUtil.setCreator(models);

// 集合工具
List<String> ids = CollectionUtil.mapDistinct(models, ModelGetWithUserVO::getId);

// 任务调度客户端
JobClientUtil.sendRunRequest(jobCode);
```
