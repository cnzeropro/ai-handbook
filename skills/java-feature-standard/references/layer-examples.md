# 分层架构完整代码示例

> 本文件为 `../SKILL.md` 的补充示例（默认以 Spring Boot + MyBatis-Plus 呈现）。核心规则见主文件“一、分层架构与职责”。
> 示例业务域为占位的 Model（模型管理），实际开发替换为项目自身业务域。
> 示例中的 `Query`（分页/排序参数封装，含 `Query.Collation`）、`UserInfoUtil` 等为示例项目组件，实际开发替换为项目自身的对应组件。
> 示例为聚焦分层结构省略 Javadoc，实际编码按 `java-coding-standard` 补充类与方法注释。

## 1. Controller 完整示例

方法顺序遵循 `java-method-ordering` skill（查询单条 → 列表 → 分页 → 更新 → 删除）。

```java
@RequiredArgsConstructor
@RestController
@RequestMapping("model")
public class ModelController {
    private final ModelService modelService;

    // 查询-单条
    @GetMapping("getWithUserById")
    public Result<ModelGetWithUserVO> getWithUserById(@RequestParam Long id) {
        ModelGetWithUserVO model = modelService.getWithUserById(id);
        if (Objects.isNull(model)) {
            return Result.error("No data found");
        }
        return Result.ok("Query succeeded", model);
    }

    // 查询-列表
    @GetMapping("listWithUser")
    public Result<Collection<ModelListWithUserVO>> listWithUser(ModelListWithUserPO modelListWithUserParam, Query query) {
        Collection<ModelListWithUserVO> models = modelService.listWithUser(modelListWithUserParam, query);
        return Result.ok("List query succeeded", models);
    }

    // 查询-分页
    @GetMapping("pageWithUser")
    public Result<IPage<ModelPageWithUserVO>> pageWithUser(ModelPageWithUserPO modelPageWithUserParam, Query query) {
        IPage<ModelPageWithUserVO> modelPage = modelService.pageWithUser(modelPageWithUserParam, query);
        return Result.ok("Page query succeeded", modelPage);
    }

    // 更新-启用
    @PutMapping("enableById")
    public Result<Void> enableById(@RequestParam Long id) {
        modelService.enableById(id);
        return Result.ok("Enabled");
    }

    // 删除（单字段使用 @RequestParam；部分网关会丢弃 DELETE body）
    @DeleteMapping("removeById")
    public Result<Void> removeById(@RequestParam Long id) {
        modelService.removeById(id);
        return Result.ok("Removed");
    }
}
```

## 2. Service 接口完整示例

方法顺序与 Controller 上下对齐。

```java
public interface ModelService extends IService<ModelPO> {
    ModelGetWithUserVO getWithUserById(Long id);

    Collection<ModelListWithUserVO> listWithUser(ModelListWithUserPO modelListWithUserParam, Query query);

    IPage<ModelPageWithUserVO> pageWithUser(ModelPageWithUserPO modelPageWithUserParam, Query query);

    void enableById(Long id);

    void disableById(Long id);

    boolean removeById(Long id);

    boolean removeByIds(Collection<Long> ids);
}
```

## 3. ServiceImpl 完整示例

```java
@Slf4j
@Service
public class ModelServiceImpl extends ServiceImpl<ModelMapper, ModelPO> implements ModelService {

    @Override
    public ModelGetWithUserVO getWithUserById(Long id) {
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
    public boolean removeById(Long id) {
        this.deleteModelFile(id);
        return super.removeById(id);
    }

    // 以下 private/protected 工具方法统一放文件末尾

    protected Optional<ModelPO> getOptById(Long id) {
        return Optional.ofNullable(this.getById(id));
    }

    protected void deleteModelFile(Long id) {
        // 删除相关逻辑
    }
}
```

## 4. Mapper 接口与 XML 完整示例

```java
public interface ModelMapper extends BaseMapper<ModelPO> {
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.module.model.mapper.ModelMapper">
    <resultMap type="com.example.module.model.model.persistent.ModelPO" id="ModelMap">
        <result property="id" column="id" jdbcType="BIGINT"/>
        <result property="code" column="code" jdbcType="VARCHAR"/>
        <result property="name" column="name" jdbcType="VARCHAR"/>
        <result property="status" column="status" jdbcType="VARCHAR"/>
        <!-- 其余字段省略 -->
    </resultMap>
</mapper>
```

## 5. 分层职责正反例（完整）

```java
// ❌ 错误：Controller 编写业务逻辑、直接调用 Mapper
@RestController
public class ModelController {
    private final ModelMapper modelMapper;

    @GetMapping("list")
    public Result<List<ModelPO>> list() {
        List<ModelPO> models = modelMapper.selectList(Wrappers.query());
        // 业务逻辑写在 Controller...
        return Result.ok(models);
    }
}

// ✅ 正确：Controller 只做参数校验与转发，业务逻辑在 Service
@RequiredArgsConstructor
@RestController
@RequestMapping("model")
public class ModelController {
    private final ModelService modelService;

    @GetMapping("list")
    public Result<Collection<ModelListVO>> list(ModelListPO param) {
        return Result.ok(modelService.list(param));
    }
}
```
