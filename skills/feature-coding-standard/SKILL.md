---
name: "feature-coding-standard"
description: "提供功能模块开发规范和代码风格指南。当用户需要开发新功能、创建新模块、重构代码或需要了解项目编码规范时调用此 skill。"
---

# 功能模块开发规范

此 skill 提供完整的功能模块开发规范和代码风格指南，确保代码质量和一致性。

## 通用规范

### 命名规范

#### 包命名

- **模块包**: `com.rongan.module.{module-name}`
- **常量包**: `com.rongan.module.{module-name}.constant`
- **控制器包**: `com.rongan.module.{module-name}.controller`
- **服务包**: `com.rongan.module.{module-name}.service`
- **服务实现包**: `com.rongan.module.{module-name}.service.impl`
- **Mapper 包**: `com.rongan.module.{module-name}.mapper`
- **Mapper XML 包**: `com.rongan.module.{module-name}.mapper.xml`
- **工具类包**: `com.rongan.module.{module-name}.util`
- **持久化对象包**: `com.rongan.module.{module-name}.model.persistent`
- **数据传输对象包**: `com.rongan.module.{module-name}.model.transfer`
- **请求参数对象包**: `com.rongan.module.{module-name}.model.param`
- **响应视图对象包**: `com.rongan.module.{module-name}.model.view`

**示例（项目模块）：**

- 模块包：`com.rongan.module.project`
- 常量包：`com.rongan.module.project.constant`
- 控制器包：`com.rongan.module.project.controller`
- 服务包：`com.rongan.module.project.service`
- 服务实现包：`com.rongan.module.project.service.impl`
- Mapper 包：`com.rongan.module.project.mapper`
- Mapper XML 包：`com.rongan.module.project.mapper.xml`
- 工具类包：`com.rongan.module.project.util`
- 持久化对象包：`com.rongan.module.project.model.persistent`
- 数据传输对象包：`com.rongan.module.project.model.transfer`
- 请求参数对象包：`com.rongan.module.project.model.param`
- 响应视图对象包：`com.rongan.module.project.model.view`

#### 类命名

| 类型 | 命名规则 | 示例 |
|------|---------|------|
| Controller | `{Entity}Controller` | `ModelController`, `ModelJobController` |
| Service 接口 | `{Entity}Service` | `ModelService`, `ModelJobService` |
| Service 实现 | `{Entity}ServiceImpl` | `ModelServiceImpl`, `ModelJobServiceImpl` |
| Mapper 接口 | `{Entity}Mapper` | `ModelMapper`, `ModelJobMapper` |
| Mapper XML | `{Entity}Mapper.xml` | `ModelMapper.xml`, `ModelJobMapper.xml` |
| PO（持久化对象） | `{Entity}PO` | `ModelPO`, `ModelJobPO` |
| DTO（数据传输对象） | `{Entity}DTO` | `ModelDTO`, `ModelJobDTO` |
| PO（请求参数对象） | `{Entity}{Operation}PO` | `ModelListPO`, `ModelListWithUserPO`, `ModelSavePO` |
| VO（响应视图对象） | `{Entity}{Operation}VO` | `ModelListVO`, `ModelListWithUserVO` |
| 枚举 | `{Entity}Status`, `{Entity}Type` | `ModelStatus`, `ModelJobStatus` |

**重要规则：**

- 任何接口都必须有自己的 Param（请求参数对象）和 View（响应视图对象，如果存在）
- 禁止在接口中直接使用持久化对象
- 因此持久化对象可以放在各自的微服务模块中，无需放在 `common` 模块

**PO（请求参数对象）命名规则：**

- 列表查询：`{Entity}ListPO`
- 带其他信息的列表查询：`{Entity}ListWith{Info}PO`
- 分页查询：`{Entity}PagePO`
- 带其他信息的分页查询：`{Entity}PageWith{Info}PO`
- 根据ID查询：`{Entity}GetByIdPO`
- 带其他信息的根据ID查询：`{Entity}GetByIdWith{Info}PO`
- 统计：`{Entity}CountPO`
- 保存：`{Entity}SavePO`
- 带其他信息的保存：`{Entity}SaveWith{Info}PO`
- 通过ID更新：`{Entity}UpdateByIdPO`
- 带其他信息通过ID更新：`{Entity}UpdateByIdWith{Info}PO`
- 通过ID启用：`{Entity}EnableByIdPO`
- 通过ID停用：`{Entity}DisableByIdPO`
- 通过ID删除：`{Entity}RemoveByIdPO`
- 批量ID删除：`{Entity}RemoveByIdsPO`
- 带其他信息通过ID删除：`{Entity}RemoveByIdWith{Info}PO`
- 带其他信息批量ID删除：`{Entity}RemoveByIdsWith{Info}PO`

**VO（响应视图对象）命名规则：**

- 列表视图：`{Entity}ListVO`
- 带其他信息的列表视图：`{Entity}ListWith{Info}VO`
- 分页视图：`{Entity}PageVO`
- 带其他信息的分页视图：`{Entity}PageWith{Info}VO`
- 详情视图：`{Entity}GetByIdVO`
- 带其他信息的详情视图：`{Entity}GetByIdWith{Info}VO`

#### 方法命名

| 功能 | 命名规则 | 示例 |
|------|---------|------|
| 查询单个 | `getBy{Field}`| `getById` |
| 查询单个并带其他信息 | `getWith{Info}By{Field}`| `getWithUserById`, `getWithJobAndUserById` |
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

#### 变量命名

- 使用驼峰命名法（camelCase）
- 避免使用缩写（除非是通用缩写）
- 使用有意义的名称

**示例：**
```java
// 推荐 ✅
private final ModelService modelService;
private String modelId;
private List<ModelGetWithUserVO> models;

// 不推荐 ❌
private final ModelService ms;
private String mid;
private List<ModelGetWithUserVO> m;
```

### 代码风格规范

#### 注解使用

##### Lombok 注解

```java
// Controller 层
@RequiredArgsConstructor
public class ModelController {
    private final ModelService modelService;
}

// Service 实现层
@Slf4j
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {
}

// 枚举
@Getter
public enum ModelStatus {
    WAITING,
    RUNNING,
}
```

##### Spring 注解

```java
// Controller
@RestController
@RequestMapping("{entity}")
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

#### 注释规范

- **文档注释**：类、方法、字段必须包含清晰的文档注释（Javadoc）。
- **方法内注释**：使用单行注释 `//` 或多行注释 `/* ... */`。
- **独立成行**：注释必须另起一行，不得跟在代码后面。
- **禁止尾随注释**：严禁使用代码后缀注释。

**示例：**
```java
/**
 * 模型服务实现类
 *
 * @author rongan
 */
@Service
public class ModelServiceImpl implements ModelService {

    /**
     * 根据ID获取模型详情
     */
    @Override
    public ModelVO getById(String id) {
        // 校验ID是否为空
        if (StrUtil.isBlank(id)) {
            return null;
        }
        
        /* 
         * 查询数据库
         * 如果不存在则抛出异常
         */
        ModelPO model = this.getOne(id);
        
        return BeanUtil.copyProperties(model, ModelVO.class); 
        // ❌ 严禁使用代码后缀注释： BeanUtil.copyProperties(model, ModelVO.class); // 转换为视图对象
    }
}
```

#### 导入顺序

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
import com.rongan.common.data.model.model.po.ModelPO;
import com.rongan.module.model.service.ModelService;
```

#### 代码格式

- 使用 4 空格缩进
- 大括号不换行
- 方法之间空一行
- 类成员变量之间空一行

**示例：**
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

### 分层架构规范

#### Controller 层

**职责**：接收 HTTP 请求，参数验证，调用 Service，返回响应

**规范**：
1. 使用 `@RequiredArgsConstructor` 注入依赖
2. 使用 `@RestController` 和 `@RequestMapping` 注解
3. 方法返回统一使用 `Result` 包装类，但下载（导出）的相关方法除外，需要使用 `ResponseEntity`
4. 接口规范：仅使用 `@GetMapping`（查询）和 `@PostMapping`（修改），不强制遵循 RESTful 风格
5. 参数使用 `@RequestParam` 或 `@RequestBody`

**示例：**
```java
@RequiredArgsConstructor
@RestController
@RequestMapping("model")
public class ModelController {
    private final ModelService modelService;

    @GetMapping("getWithUserById")
    public Result<ModelGetWithUserVO> getWithUserById(@RequestParam String id) {
        ModelGetWithUserVO model = modelService.getWithUserById(id);
        if (Objects.isNull(model)) {
            return Result.error("暂无数据");
        }
        return Result.ok("查询成功", model);
    }

    @GetMapping("listWithUser")
    public Result<Collection<ModelListWithUserVO>> listWithUser(ModelListWithUserPO modelListWithUserParam, Query query) {
        Collection<ModelListWithUserVO> models = modelService.listWithUser(modelListWithUserParam, query);
        return Result.ok("查询列表成功", models);
    }

    @GetMapping("pageWithUser")
    public Result<IPage<ModelPageWithUserVO>> pageWithUser(ModelPageWithUserPO modelPageWithUserParam, Query query) {
        IPage<ModelPageWithUserVO> modelPage = modelService.pageWithUser(modelPageWithUserParam, query);
        return Result.ok("分页查询成功", modelPage);
    }

    @PostMapping("removeById")
    public Result<Void> removeById(@RequestBody ModelRemoveByIdPO modelRemoveByIdParam) {
        modelService.removeById(modelRemoveByIdParam.getId());
        return Result.ok("删除成功");
    }

    @PostMapping("enable")
    public Result<Void> enable(@RequestBody ModelEnableByIdPO modelEnableByIdParam) {
        modelService.enable(modelEnableByIdParam.getId());
        return Result.ok("启用成功");
    }
}
```

#### Service 层

**职责**：业务逻辑处理，事务管理，调用 Mapper

**规范**：
1. Service 接口继承 `IService<PO>`
2. Service 实现继承 `ServiceImpl<Mapper, PO>`
3. 使用 `@Service` 注解，指定 bean 名称
4. 使用 `@Transactional` 注解管理事务
5. 使用 `Optional` 处理可能为 null 的值
6. 使用 `Wrappers` 构建查询条件

**Service 接口示例：**
```java
public interface ModelService extends IService<ModelPO> {
    ModelGetWithUserVO getWithUserById(String id);

    Collection<ModelListWithUserVO> listWithUser(ModelListWithUserPO modelListWithUserParam, Query query);

    IPage<ModelPageWithUserVO> pageWithUser(ModelPageWithUserPO modelPageWithUserParam, Query query);

    boolean removeById(String id);

    boolean removeByIds(Collection<String> ids);

    void disable(String id);

    void enable(String id);
}
```

**Service 实现示例：**
```java
@Slf4j
@Service
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {

    @Override
    public ModelGetWithUserVO getWithUserById(String id) {
        return this.getOptById(id).map(model -> BeanUtil.copyProperties(model, ModelGetWithUserVO.class))
                .map(model -> {
                    UserInfoUtil.setCreator(model);
                    return model;
                })
                .orElse(null);
    }

    @Override
    public Collection<ModelListWithUserVO> listWithUser(ModelListWithUserPO modelListWithUserParam, Query query) {
        QueryWrapper<ModelPO> queryWrapper = Wrappers.query();
        queryWrapper.lambda()
                .eq(Objects.nonNull(modelListWithUserParam.getStatus()), ModelPO::getStatus, modelListWithUserParam.getStatus())
                .like(CharSequenceUtil.isNotBlank(modelListWithUserParam.getCode()), ModelPO::getCode, modelListWithUserParam.getCode())
                .like(CharSequenceUtil.isNotBlank(modelListWithUserParam.getName()), ModelPO::getName, modelListWithUserParam.getName());
        for (Query.Collation collation : query.getCollations()) {
            queryWrapper.orderBy(Objects.nonNull(collation.getColumn()), collation.isAsc(), collation.getColumn());
        }
        Collection<ModelPO> models = this.list(queryWrapper);
        return models.stream()
            .map(model -> BeanUtil.copyProperties(model, ModelListWithUserVO.class))
            .peek(model -> UserInfoUtil.setCreator(model))
            .collect(Collectors.toList());
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean removeById(String id) {
        this.deleteModelFile(id);
        return super.removeById(id);
    }

    protected Optional<ModelPO> getOptById(String id) {
        return Optional.ofNullable(this.getById(id));
    }

    protected void deleteModelFile(String id) {
        // 删除相关逻辑
    }
}
```

#### Mapper 层

**职责**：数据库访问，SQL 执行

**规范**：
1. Mapper 接口继承 `BaseMapper<PO>`
2. 使用 `@Mapper` 注解（可选，MyBatis-Plus 会自动扫描）
3. XML 文件位于 `mapper/xml/` 目录下
4. 使用 `resultMap` 映射结果集
5. 除了基本的 CRUD 操作，大部分联表查询语句需要在 Mapper XML 中编写

**Mapper 接口示例：**
```java
public interface ModelMapper extends BaseMapper<ModelPO> {
}
```

**Mapper XML 示例：**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.rongan.module.model.mapper.ModelMapper">
    <resultMap type="com.rongan.common.data.model.model.persistent.ModelPO" id="ModelMap">
        <result property="id" column="id" jdbcType="VARCHAR"/>
        <result property="code" column="code" jdbcType="VARCHAR"/>
        <result property="name" column="name" jdbcType="VARCHAR"/>
        <result property="description" column="description" jdbcType="VARCHAR"/>
        <result property="jobId" column="job_id" jdbcType="VARCHAR"/>
        <result property="type" column="type" jdbcType="INTEGER"/>
        <result property="status" column="status" jdbcType="INTEGER"/>
        <result property="config" column="config" jdbcType="VARCHAR"/>
        <result property="delFlag" column="del_flag" jdbcType="INTEGER"/>
        <result property="createBy" column="create_by" jdbcType="VARCHAR"/>
        <result property="createTime" column="create_time" jdbcType="TIMESTAMP"/>
        <result property="updateBy" column="update_by" jdbcType="VARCHAR"/>
        <result property="updateTime" column="update_time" jdbcType="TIMESTAMP"/>
    </resultMap>
</mapper>
```

### PO/DTO/PO/VO 规范

#### PO（持久化对象）

- 位于各模块的 `model/persistent/` 目录
- 对应数据库表
- 使用 `@TableName` 注解指定表名
- 使用 `@TableId` 注解指定主键
- 使用 `@TableField` 注解指定字段映射
- 命名规则：`{Entity}PO`

**示例：**
```java
@Data
@SuperBuilder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
@TableName("up_model")
public class ModelPO {
    @TableId(type = IdType.ASSIGN_ID)
    private String id;

    private String code;
    private String name;
    private String description;
    private String jobId;
    private Integer type;
    private Boolean status;
    private String config;

    @TableLogic
    private Integer delFlag;

    private String createBy;
    private Date createTime;
    private String updateBy;
    private Date updateTime;
}
```

#### DTO（数据传输对象）

- 位于 `common` 模块的各模块的 `model/transfer/` 目录
- 用于模块间数据传输
- 使用 `lombok` 注解
- 命名规则：`{Entity}DTO`

**示例：**
```java
@Data
@SuperBuilder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
public class ModelDTO {
    private String id;
    private String code;
    private String name;
    private String description;
    private Integer type;
    private Integer status;
    private String config;
}
```

#### PO（请求参数对象）

- 位于 `common` 模块的各模块的 `model/param/` 目录
- 用于接收请求参数
- 使用 `lombok` 注解
- 命名规则：`{Entity}{Operation}PO`

**示例：**
```java
@Data
@SuperBuilder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
public class ModelListPO {
    private String code;
    private String name;
    private Integer status;
    private Date startDeploymentTime;
    private Date endDeploymentTime;
}

@Data
@SuperBuilder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
public class ModelSavePO {
    private String name;
    private String description;
    private Integer type;
    private String config;
}
```

#### VO（响应视图对象）

- 位于 `common` 模块的各模块的 `model/view/` 目录
- 用于复杂的视图展示场景
- 包含多个表的关联数据
- 使用 `lombok` 注解
- 命名规则：`{Entity}{Operation}VO`

**示例：**
```java
@Data
@SuperBuilder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
public class ModelGetWithUserVO {
    private String id;
    private String code;
    private String name;
    private String description;
    private String groupId;
    private String groupName;  // 关联的组名称
    private Integer type;
    private Integer status;
    private Integer jobStatus;  // 任务执行状态
    private Date executionTime;  // 任务执行时间
    private String config;
    private String createBy;
    private String createByName;  // 创建人姓名
    private Date createTime;
    private String updateBy;
    private Date updateTime;
}
```

**使用场景：**
- 需要展示多个表的关联数据
- 需要额外的计算字段或展示字段
- 需要特定的视图格式

### 常量和枚举规范

#### 枚举

- 使用 `@Getter` 注解
- 枚举值使用全大写，下划线分隔
- 如果枚举含有 int 类型标识，直接使用 `ordinal()` 方法，而无需定义成员变量，因此数据库表中状态、类型等等字段建议从 0 开始编号

**示例：**

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
// 获取枚举对应的 int 值
int status = ModelStatus.WAITING.ordinal();

// 根据 int 值获取枚举
ModelStatus status = ModelStatus.values()[intValue];
```

#### 常量

- 使用 `public static final` 修饰
- 使用全大写，下划线分隔
- 集中管理在常量类中

**示例：**
```java
public class ModelConstants {
    public static final String MODEL_TYPE_LOOKALIKE = "lookalike";
    public static final String MODEL_TYPE_RS = "rs";
    public static final Integer MODEL_TYPE_LOOKALIKE_VALUE = 1;
    public static final Integer MODEL_TYPE_RS_VALUE = 2;
}
```

### 异常处理规范

#### 自定义异常

- 继承 `RuntimeException` 或项目基础异常类
- 提供有意义的错误信息

**示例：**
```java
public class PersonaException extends RuntimeException {
    public PersonaException(String message) {
        super(message);
    }

    public PersonaException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

#### 异常处理

- 在 Service 层抛出业务异常
- 在 Controller 层捕获并返回错误信息
- 使用 `Optional` 避免 NPE

**示例：**
```java
// Service 层
public void enable(String id) {
    this.getOptById(id)
            .map(model -> {
                model.setStatus(true);
                return model;
            })
            .orElseThrow(() -> new PersonaException("模型不存在"));
    this.updateById(model);
}

// Controller 层
@GetMapping("getWithUserById")
public Result<ModelGetWithUserVO> getWithUserById(@RequestParam String id) {
    ModelGetWithUserVO model = modelService.getWithUserById(id);
    if (Objects.isNull(model)) {
        return Result.error("暂无数据");
    }
    return Result.ok("查询成功", model);
}
```

### 日志规范

#### 日志注解

- 使用 `@Slf4j` 注解
- 使用 `log` 变量记录日志

#### 日志级别

- **DEBUG**: 调试信息
- **INFO**: 重要业务流程
- **WARN**: 警告信息
- **ERROR**: 错误信息

#### 日志格式

- 使用占位符而非字符串拼接
- 包含必要的上下文信息
- 异常日志包含堆栈信息
- 关键业务流程记录日志

**示例：**
```java
@Slf4j
@Service("modelService")
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {

    @Override
    public void run(String id) {
        log.info("开始执行模型任务: jobId={}", id);
        try {
            this.getOptById(id).map(ModelJobPO::getCode)
                    .ifPresent(code -> {
                        AirflowClientUtil.sendRunRequest(code);
                        log.info("模型任务执行成功: jobId={}", id);
                    });
        } catch (Exception e) {
            log.error("模型任务执行失败: jobId={}", id, e);
            throw new PersonaException("模型任务执行失败");
        }
    }
}
```

### 事务管理规范

#### 事务注解

- 使用 `@Transactional` 注解
- 指定 `rollbackFor = Exception.class`
- 只读事务使用 `readOnly = true`

**示例：**
```java
@Override
@Transactional(rollbackFor = Exception.class)
public boolean saveWithInfo(ModelSaveParam param) {
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

### 工具类使用

#### Hutool 工具类

- `BeanUtil`: 对象属性拷贝
- `CollUtil`: 集合操作
- `CharSequenceUtil`: 字符串操作
- `ArrayUtil`: 数组操作
- `DateUtil`: 日期操作
- `JSONUtil`: JSON 操作
- `Optional`: 可选值处理

**示例：**
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

// Optional 处理
return Optional.ofNullable(this.getById(id))
        .map(model -> BeanUtil.copyProperties(model, ModelGetWithUserVO.class))
        .orElse(null);
```

#### 项目工具类

- `CodeUtil`: 编码生成
- `UserInfoUtil`: 用户信息设置
- `CollectionUtil`: 集合工具
- `AirflowClientUtil`: Airflow 客户端

**示例：**
```java
// 编码生成
String code = CodeUtil.createCode(CodeUtil.CodeType.MODEL);

// 用户信息设置
UserInfoUtil.setCreator(models);

// 集合工具
List<String> ids = CollectionUtil.mapDistinct(models, ModelGetWithUserVO::getId);

// Airflow 客户端
AirflowClientUtil.sendRunRequest(jobCode);
```

## 开发流程

### 1. 创建新功能

1. 确定功能所属模块
2. 创建包结构
3. 创建常量和枚举
4. 创建 PO 类（在模块的 `model/persistent/` 目录）
5. 创建 Parameter 和 View 类
6. 创建 Mapper 接口和 XML
7. 创建 Service 接口和实现
8. 创建 Controller

### 2. 开发新功能

1. 定义接口（Controller、Service）
2. 实现业务逻辑（Service）
3. 编写单元测试
4. 代码审查
5. 提交代码
6. 推送到远程仓库合并到 release 分支（使用 git-workflow 技能）

### 3. 代码审查清单

参见：code-review 技能

- [ ] 遵循命名规范
- [ ] 遵循代码风格
- [ ] 使用正确的注解
- [ ] 事务管理正确
- [ ] 异常处理完善
- [ ] 日志记录规范
- [ ] 性能优化考虑
- [ ] 安全性考虑

### 4. 测试规范

- 单元测试覆盖率不低于 70%
- 集成测试覆盖主要业务流程
- 性能测试验证关键接口
- 安全测试检查常见漏洞

## 最佳实践与常见错误规避

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

### 4. Git 操作规范 (重要)

- **禁止使用 `git add .`**：严禁在未检查状态时使用全量添加，必须精确添加文件 (`git add <file>`)。
- **提交前检查**：必须执行 `git status` 确认暂存区内容，避免提交编译产物（如 `.class`）、IDE 配置（如 `.idea/`）、自动生成代码（如 gRPC 生成文件）。
- **忽略文件管理**：确保 `.gitignore` 覆盖所有自动生成目录。

### 5. 数据更新逻辑规范

- **关联数据全量更新策略**：
  - `null`：表示不更新，保持原状。
  - `[]` (空列表)：表示清空关联数据。
  - `Non-empty List`：先删除旧数据，再插入新数据。
- **级联删除**：删除父实体时，必须同步删除所有关联的子实体数据。

### 6. 数据库变更规范

- **多数据库支持**：必须同时提供 MySQL 和 GaussDB 的脚本。
- **增量脚本**：必须包含数据迁移逻辑（如 `UPDATE` 旧数据），不能只修改表结构。
- **默认值安全**：新增字段时，默认值应设为 `NULL` 或安全值，避免影响历史数据。
- **逻辑删除**：所有实体表应包含 `del_flag` 字段，并使用逻辑删除而非物理删除。

### 7. 遗留代码兼容

- **风格一致性**：在维护遗留模块时，优先遵循该模块现有的命名风格（如 DTO vs PO），除非进行全面重构。
