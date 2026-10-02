# 注解与事务完整示例

> 本文件为 `../SKILL.md` 的补充示例。核心规则见主文件“四、注解规范”与“五、事务管理规范”。
> 示例业务域为占位的 Model（模型管理），实际开发替换为项目自身业务域。

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

## 2. 事务管理完整示例

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
public boolean removeById(Long id) { ... }

// ✅ 正确
@Transactional(rollbackFor = Exception.class)
public boolean saveWithInfo(ModelSavePO param) { ... }

@Transactional(readOnly = true)
public ModelGetByIdVO getById(Long id) { ... }
```
