# 接口设计规范

> 本文件为 `../SKILL.md` 的补充规范：Controller 接口设计的详细规则与示例。核心条目见主文件“一、分层架构与职责 → Controller 层规范”。

## 1. 参数校验

- 请求参数使用 Jakarta Validation 注解校验：`@NotNull` / `@NotBlank` / `@Size` / `@Min` / `@Max` 等
- 校验入口：Controller 方法参数上加 `@Valid`（或类上加 `@Validated`），校验失败由全局异常处理器统一转错误响应
- 校验分组：同一 Param 对象用于多个场景时，用校验分组区分（如新增组与更新组要求不同）
- 复杂业务校验（跨字段、依赖查询）在 Service 层实现，不使用注解勉强表达

```java
// ✅ 正确：Param 对象字段注解校验 + Controller @Valid 触发
public class ModelSavePO {
    @NotBlank(message = "Model name must not be blank")
    private String name;

    @NotNull(message = "Model type must not be null")
    private String type;
}

@PostMapping("save")
public Result<Void> save(@Valid @RequestBody ModelSavePO modelSaveParam) {
    return Result.ok();
}
```

## 2. 幂等设计

写接口（新增/更新/删除）需要幂等设计，防止重复提交与网络重试导致重复数据：

- 业务唯一键兜底：入库字段建唯一索引（见 `db-design-standard` 规则 6），重复时返回明确提示
- 防重复提交 token：表单/关键操作先获取 token，提交时携带并校验一次性
- 状态机校验：更新类接口校验当前状态，非法状态转换直接拒绝（天然幂等）

## 3. 空列表返回

列表查询接口返回空集合 `[]`（而非 null），减少前端 null 判断：

```java
// ✅ 正确：无结果返回空集合
return Result.ok(Collections.emptyList());

// ❌ 错误：无结果返回 null
return Result.ok(null);
```

## 4. 错误响应结构

服务端错误响应包含四要素，各司其职：

| 要素 | 受众 | 说明 |
| --- | --- | --- |
| HTTP 状态码 | 浏览器/网关 | 200 / 401 / 403 / 404 / 500 等 |
| `errorCode` | 前端程序 | 错误码（规范见 `java-coding-standard`） |
| `errorMessage` | 排查人员 | 后端出错原因，不含敏感数据 |
| `userTip` | 用户 | 简短友好、引导下一步操作 |

## 5. 路径风格

- 路径风格按项目统一（示例默认 camelCase 动作式路径，如 `getWithUserById`；跨团队项目可统一为小写下划线）；路径必须小写
- 路径禁止携带表示内容类型的后缀（如 `.json` / `.xml`，通过 Accept 头表达）
- 路径中不加入版本号（版本控制在请求头中体现）
- URL 携带参数不超过 2048 字节（浏览器最小限制），大批量参数走 body

## 6. 分页参数边界

- 前端传入 pageSize 小于 1 时按第 1 页处理；服务端限制 pageSize 上限（防超大分页拖垮查询）
- 请求页码大于总页数时返回最后一页（或空页，按项目约定统一）

## 7. 序列化细节

- 时间字段统一 `@JsonFormat`（如 `yyyy-MM-dd HH:mm:ss`），避免默认时间戳/纳秒格式
- 敏感字段（密码、token 等）返回 VO 中加 `@JsonIgnore` 或使用脱敏序列化器
