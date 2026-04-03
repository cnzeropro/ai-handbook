---
name: "method-ordering"
description: "统一接口/方法的排序规则（Controller/Service/Mapper/Feign/@HttpExchange）。当需要重排接口顺序、统一代码结构或新建接口时调用。"
---

# 方法（接口）排序规则

本规则用于统一“接口方法/端点方法/契约方法”的排列顺序，提升可读性与可维护性。适用于：

- Spring MVC Controller（`@RestController` 等）
- Service 接口与实现类
- MyBatis Mapper 接口
- Mapper XML（`<select>/<insert>/<update>/<delete>` 语句块）
- Feign Client 契约接口（`@FeignClient`）
- Spring 6 Declarative HTTP Client 契约接口（`@HttpExchange` / `@GetExchange` / `@PostExchange` 等），例如 `InformationExchange`

## 一、排序总则（强制）

### 1. 顶层分类顺序（强制）

按语义将方法分为 5 类，并严格按以下顺序排列：

1. 查询
2. 新增
3. 更新
4. 删除
5. 其他

### 2. 全局通用顺序（强制，适用于所有类别）

在同一类别（或子类别）内，继续按以下优先级排序：

1. 单条在前，多条在后
2. 本身在前，带信息在后
3. 带信息按信息量从少到多

其中：

- “本身”通常指方法名不包含 `With` 的基本能力
- “带信息”通常指方法名包含 `WithXxx`，表示聚合/联表/附带额外信息
- “信息量”按 `With...` 中携带的信息项数量衡量（见“信息量计算”）

### 3. 查询类内部顺序（强制）

查询类进一步分层并按以下顺序排列：

1. 单条
2. 列表（多条）
3. 分页（多条）
4. 其他（如 count/exists/check 等）

在上述每一层内部，仍需遵循“全局通用顺序（单条→本身→信息量）”。

## 二、分类判定（将方法归入 5 类）

优先按**语义/命名**判定；命名不清晰时再参考注解（例如 GET/POST/PUT/DELETE）。

### 1. 查询

常见命名：

- `get* / list* / page* / count* / exists* / check* / query* / find*`

常见注解参考：

- Controller：`@GetMapping`
- Feign：`@GetMapping` 或 `@RequestMapping(method = GET)`
- Exchange：`@GetExchange`

### 2. 新增

常见命名：

- `save* / create* / add* / import*`

常见注解参考：

- Controller：`@PostMapping`
- Exchange：`@PostExchange`

### 3. 更新

常见命名：

- `update* / modify* / edit* / change* / reset*`
- 状态变更建议归为更新：`enable* / disable*`

### 4. 删除

常见命名：

- `remove* / delete* / cancel* / revoke*`

### 5. 其他

不属于以上语义的动作型方法：

- `upload / download / export / sync / refresh / execute / run / stop / start / terminate` 等

## 三、单条 vs 多条判定

### 1. 单条（优先）

通常满足其一：

- 方法名包含 `ById`、且返回单个对象/void/boolean
- 语义明显为对单个资源操作：`getById / removeById / updateById / enableById`

### 2. 多条

通常满足其一：

- 方法名包含 `ByIds`、或语义为批量：`removeByIds / saveBatch`
- 返回集合：`Collection/List/Set` 等
- 返回分页：`IPage/Page`
- 命名为 `list* / page*`

## 四、本身 vs 带信息判定

### 1. 本身（优先）

- 方法名不包含 `With`
- 示例：`getById`、`list`、`page`、`save`、`updateById`

### 2. 带信息

- 方法名包含 `With`
- 示例：`getWithInfoById`、`listWithUser`、`pageWithJobAndUser`

## 五、信息量计算（用于带信息方法的先后）

仅对包含 `With` 的方法计算信息量：

1. 取方法名中 `With` 之后的片段
2. 若包含 `By`，截断到 `By` 之前；否则截断到方法名结尾
3. 按 `And` 拆分后计数（拆分段数即信息项数量）

示例：

- `getWithJobById`：信息量 = 1（Job）
- `getWithSubjectAndUserById`：信息量 = 2（Subject、User）
- `pageWithUserAndRole`：信息量 = 2（User、Role）

信息量小的排在前，信息量大的排在后。

## 六、落地模板（不同层的排序落法）

### 1. Controller

按以下块顺序组织方法：

1. 查询：单条 → 列表 → 分页 → 其他（每组内本身→With→信息量少到多）
2. 新增：本身→With→信息量少到多
3. 更新：本身→With→信息量少到多
4. 删除：本身→With→信息量少到多（且单条在批量前）
5. 其他：按语义分组（如 upload/download/export），同组内仍按单条优先

### 2. Service（接口）

Service 接口方法顺序与 Controller 保持一致（同一组 API 在 Controller/Service 之间“上下对齐”）。

### 3. ServiceImpl（实现类）

- public 方法顺序与 Service 接口一致
- private/protected 工具方法统一放在文件末尾

### 4. Feign Client 与 Spring 6 `@HttpExchange` 契约接口

方法是“远端契约”，排序规则与 Controller 完全一致：

- 先按语义分类（查询/新增/更新/删除/其他）
- 查询内再按单条/列表/分页/其他
- 同组内按本身/With/信息量

当方法命名无法体现语义时，才用注解辅助判断（GET/POST 等）。

### 5. Mapper 接口

- 方法声明顺序与 Service 同步
- 若 Mapper 没有自定义方法，则无需调整

### 6. Mapper XML

结构建议：

1. `resultMap` / `sql` 片段放顶部
2. `<select>`：单条 → 列表 → 分页/其他（每组内本身→With→信息量少到多）
3. `<insert>`：本身 → With（With 内信息量少到多）
4. `<update>`：本身 → With（With 内信息量少到多）
5. `<delete>`：本身 → With（With 内信息量少到多），且单条在批量前

## 七、示例（方法清单排序）

以下示例展示“同一业务域”的典型排序（方法签名仅示意，重点看顺序）：

### 1. Controller / Service / Feign / Exchange（通用）

1) 查询-单条（本身→With）

- `getById`
- `getByCode`
- `getWithJobById`
- `getWithJobAndUserById`

2) 查询-列表（本身→With）

- `list`
- `listWithUser`
- `listWithUserAndRole`

3) 查询-分页（本身→With）

- `page`
- `pageWithUser`
- `pageWithUserAndRole`

4) 查询-其他

- `count`
- `existsByCode`

5) 新增（本身→With）

- `save`
- `saveWithInfo`
- `saveWithJobAndUser`

6) 更新（本身→With）

- `updateById`
- `updateWithInfoById`
- `updateWithJobAndUserById`

7) 删除（单条→批量，本身→With）

- `removeById`
- `removeWithInfoById`
- `removeByIds`
- `removeWithInfoByIds`

8) 其他

- `upload`
- `download`

### 2. Spring 6 `@HttpExchange` 示例

如果同一个 Exchange 接口同时存在查询与接收回调等方法，建议按上述模板组织。例如：

1) 查询（如有）
2) 新增/更新/删除（如有）
3) 其他（如回调接收）

## 八、执行与验收清单

- [ ] 顶层顺序：查询 → 新增 → 更新 → 删除 → 其他
- [ ] 查询内部：单条 → 列表 → 分页 → 其他
- [ ] 同组内部：单条优先、本身优先、With 信息量少→多
- [ ] 删除：单条在批量前
- [ ] 仅移动方法块，不改动注解/签名/路由/逻辑
