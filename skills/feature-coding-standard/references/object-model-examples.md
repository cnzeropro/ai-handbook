# 对象模型完整类定义

> 本文件为 `../SKILL.md` 的补充示例。核心规则见主文件“三、对象模型规范”。

## 1. PO（持久化对象）完整示例

```java
@Data
@SuperBuilder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
@TableName("example_model")
public class ModelPO {
    @TableId(type = IdType.ASSIGN_ID)
    private String id;

    private String code;
    private String name;
    private String description;
    private String jobId;
    private String type;
    private String status;
    private String config;

    @TableLogic
    private Integer isDeleted;

    private String createBy;
    private Date createAt;
    private String updateBy;
    private Date updateAt;
}
```

## 2. DTO（数据传输对象）完整示例

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
    private String type;
    private String status;
    private String config;
}
```

## 3. Param（请求参数对象）完整示例

```java
@Data
@SuperBuilder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
public class ModelListPO {
    private String code;
    private String name;
    private String status;
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
    private String type;
    private String config;
}
```

## 4. VO（响应视图对象）完整示例

用于复杂视图展示：包含多个表的关联数据、额外的计算字段或展示字段。

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
    private String groupName;   // 关联的组名称
    private String type;
    private String status;
    private String jobStatus;   // 任务执行状态
    private Date executionTime; // 任务执行时间
    private String config;
    private String createBy;
    private String createByName; // 创建人姓名
    private Date createAt;
    private String updateBy;
    private Date updateAt;
}
```

## 5. 对象模型使用边界正反例

```java
// ❌ 错误：Controller 入参出参直接用 PO/DTO
public Result<ModelPO> getById(@RequestParam String id) { ... }
public Result<Void> save(@RequestBody ModelPO model) { ... }
public Result<ModelDTO> sync(ModelDTO dto) { ... }     // 接口层不应暴露 DTO

// ✅ 正确：接口用 Param/VO，DTO 仅用于模块间（Feign/RPC）传输
public Result<ModelGetByIdVO> getById(@RequestParam String id) { ... }
public Result<Void> save(@RequestBody ModelSavePO modelSaveParam) { ... }
```

## 6. Lombok 注解完整示例

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
