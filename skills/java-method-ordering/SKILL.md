---
name: "java-method-ordering"
description: "统一接口/端点/契约方法的排序规则（Controller/Service/Mapper/Feign/@HttpExchange/Dubbo/gRPC 等）。当需要新建接口、重排方法顺序、统一各层代码结构、向现有接口添加方法或检查方法排序是否符合规范时使用。"
---

# 方法（接口）排序规则

本规则用于统一“接口方法 / 端点方法 / 契约方法”的排列顺序，保证同一业务在各层之间顺序一致，提升可读性与可维护性。

## 何时使用

- 新建接口（Controller/Service/Mapper/Feign/Exchange 等），需要布局方法顺序
- 重排现有接口中顺序混乱的方法
- 向现有接口新增方法，需要确定插入位置（而非一律追加到末尾）
- 代码审查时检查方法排序是否符合规范
- 需要让同一组 API 在 Controller/Service/Mapper/Feign 各层“上下对齐”

## 适用范围

### 适用对象（本规则适用于）

**Web 层**

- Spring MVC Controller（`@RestController` / `@Controller`）

**RPC 层**

- Dubbo API 接口与实现（`@DubboService` / `@DubboReference`）
- gRPC Service 实现类
- 其他内部 RPC / SDK 契约接口

**业务层**

- Service 接口与实现类（`ServiceImpl`）

**数据访问层**

- MyBatis Mapper 接口
- Mapper XML（`<select>/<insert>/<update>/<delete>` 语句块）
- MyBatis-Plus 扩展接口（继承 `BaseMapper` / `IService`）
- Spring Data JPA Repository（含 `@Query` 声明的方法）

**远程契约层**

- Feign Client（`@FeignClient`）
- Spring 6 Declarative HTTP Client（`@HttpExchange` / `@GetExchange` / `@PostExchange` / `@PutExchange` / `@DeleteExchange`），例如 `UserExchange`
- Retrofit 等声明式 HTTP API 接口

**消息与事件层**

- MQ 消息处理类（含多个处理端点方法）
- 事件监听器（`@EventListener`，含多个监听方法）

**接口文档层**

- OpenAPI / Swagger 定义的接口端点（`@Operation` 注解的方法）

### 不适用（边界）

- 工具类（Util）、静态工厂：按职责分组即可，不套用五类
- 枚举类、PO/DTO/VO/Param 等纯数据对象
- 构造器、getter/setter
- Spring 生命周期回调（`@PostConstruct` / `@PreDestroy` / `InitializingBean` 等）
- `ServiceImpl` 内部的 private/protected 工具方法：统一放文件末尾
- 测试类

## 一、排序总则（强制）

### 1. 顶层分类顺序（强制）

按语义将方法分为 5 类，严格按以下顺序排列：

1. 查询
2. 新增
3. 更新
4. 删除
5. 其他

### 2. 组内通用顺序（强制，适用于所有类别）

同一类别内按以下优先级排序：

1. 单条在前，多条在后
2. 本身在前，带信息在后
3. 带信息按信息量从少到多

其中：

- “本身”通常指方法名不含 `With` 的基本能力
- “带信息”通常指方法名含 `WithXxx`，表示聚合/联表/附带额外信息；`getDetailById`、`listFull` 等表达聚合语义的命名视同带信息
- “信息量”按 `With...` 中携带的信息项数量衡量（见“六、信息量计算”）

### 3. 查询类内部顺序（强制）

查询类进一步分层，按以下顺序排列：

1. 单条
2. 列表（多条）
3. 分页（多条）
4. 其他（count/exists/check 等）

每一层内部仍遵循“组内通用顺序”。

### 4. 排序键总结

每个方法的最终位置由以下排序键决定：

> 排序键 = 类别（查询/新增/更新/删除/其他） > 查询子层（单条/列表/分页/其他） > 单条/多条 > 本身/带信息 > 信息量

## 二、判定流程

对每个方法依次执行四步，得到排序键：

1. **分类**：按语义/命名归入五类之一（见“三、分类判定”）
2. **单条/多条**：判定操作对象数量（见“四、单条 vs 多条”）
3. **本身/带信息**：方法名是否含 `With`（见“五、本身 vs 带信息”）
4. **信息量**：带信息时计算 `With` 后的信息项数量（见“六、信息量计算”）

## 三、分类判定

**判定原则：优先按语义/命名判定；命名不清晰时再参考注解（GET/POST/PUT/DELETE）。**

| 类别 | 常见命名前缀 | Controller 注解 | Feign / Exchange 注解 |
| --- | --- | --- | --- |
| 查询 | `get*` `list*` `page*` `query*` `find*` `search*` `count*` `exists*` `check*` `detail*` | `@GetMapping` | `@GetMapping` / `@GetExchange` |
| 新增 | `save*` `create*` `add*` `insert*` `import*` `register*` | `@PostMapping` | `@PostMapping` / `@PostExchange` |
| 更新 | `update*` `modify*` `edit*` `change*` `reset*` `enable*` `disable*` `approve*` `reject*` `audit*` `publish*` `renew*` `activate*` `deactivate*` | `@PutMapping` / `@PatchMapping` | `@PutMapping` / `@PutExchange` |
| 删除 | `remove*` `delete*` `cancel*` `revoke*` `invalidate*` | `@DeleteMapping` | `@DeleteMapping` / `@DeleteExchange` |
| 其他 | `upload` `download` `export` `sync` `execute` `run` `start` `stop` `terminate` `send` `push` `notify` `trigger` `refresh` | 视语义 | 视语义 |

补充规则：

- 所有“状态变更”动作（启用/禁用/审批/发布/续期等）归**更新**，与其 HTTP 注解无关
- `import*` 归新增，`export*` 归其他
- 无法判定语义时归“其他”，并尽量给出合理分组

## 四、单条 vs 多条判定

### 1. 单条（优先）

通常满足其一：

- 方法名含 `ById`，且返回单个对象/void/boolean
- 语义明显为对单个资源的操作：`getById` / `removeById` / `updateById` / `enableById`

### 2. 多条

通常满足其一：

- 方法名含 `ByIds` / `batch*` / `All*`，或语义为批量：`removeByIds` / `saveBatch`
- 返回集合：`Collection/List/Set/Map`
- 返回分页：`Page/IPage`
- 命名为 `list*` / `page*`

注：`count/exists/check` 归查询-其他，不参与单条/多条排序。

## 五、本身 vs 带信息判定

### 1. 本身（优先）

- 方法名不含 `With`
- 示例：`getById`、`list`、`page`、`save`、`updateById`

### 2. 带信息

- 方法名含 `With`
- 示例：`getWithInfoById`、`listWithUser`、`pageWithJobAndUser`
- 视同带信息的聚合命名：`getDetailById`、`listFull`

## 六、信息量计算（用于带信息方法的先后）

仅对带信息方法计算信息量：

1. 取方法名中 `With` 之后的片段
2. 若含 `By`，截断到 `By` 之前；否则截断到方法名结尾
3. 按 `And` 拆分后计数（拆分段数即信息项数量）

示例：

- `getWithJobById`：信息量 = 1（Job）
- `getWithSubjectAndUserById`：信息量 = 2（Subject、User）
- `pageWithUserAndRole`：信息量 = 2（User、Role）

约定：多个信息项必须用 `And` 显式连接；`WithUserRole` 这类连续驼峰复合词计 1 项，需要区分时改写为 `WithUserAndRole`。

信息量小的排在前，信息量大的排在后。

## 七、各层落地模板

### 1. Controller

按以下块顺序组织方法：

1. 查询：单条 → 列表 → 分页 → 其他（每组内本身 → 带信息 → 信息量少到多）
2. 新增：本身 → 带信息 → 信息量少到多
3. 更新：本身 → 带信息 → 信息量少到多
4. 删除：本身 → 带信息 → 信息量少到多（且单条在批量前）
5. 其他：按语义分组（如 upload/download/export），同组内仍单条优先

### 2. Service（接口）

Service 接口方法顺序与 Controller 保持一致（同一组 API 在 Controller/Service 之间“上下对齐”）。

### 3. ServiceImpl（实现类）

- public 方法顺序与 Service 接口一致
- private/protected 工具方法统一放在文件末尾

### 4. Feign Client 与 Spring 6 `@HttpExchange` 契约接口

方法是“远端契约”，排序规则与 Controller 完全一致：

- 先按语义分类（查询/新增/更新/删除/其他）
- 查询内再按单条/列表/分页/其他
- 同组内按本身/带信息/信息量

当方法命名无法体现语义时，才用注解辅助判断（GET/POST 等）。

### 5. Dubbo / gRPC 等 RPC 契约接口

与 Controller 同规则：先分类，查询内再分层，同组内本身 → 带信息 → 信息量。

### 6. Mapper 接口 / JPA Repository

- 方法声明顺序与 Service 同步
- 若 Mapper 没有自定义方法，则无需调整

### 7. Mapper XML

结构建议：

1. `resultMap` / `sql` 片段放顶部
2. `<select>`：单条 → 列表 → 分页/其他（每组内本身 → 带信息 → 信息量少到多）
3. `<insert>`：本身 → 带信息（带信息内信息量少到多）
4. `<update>`：本身 → 带信息（带信息内信息量少到多）
5. `<delete>`：本身 → 带信息（带信息内信息量少到多），且单条在批量前

### 8. MQ 处理类 / 事件监听器

多个处理端点方法时，按“查询 → 新增 → 更新 → 删除 → 其他”排列；回调/事件接收类方法归“其他”，按语义分组。

## 八、正反例

以下示例中方法签名仅示意，重点看顺序。

### 1. 顶层分类：五类顺序不可颠倒

❌ 错误（删除在前、查询在后）：

- `removeById` → `getById` → `save` → `updateById`

✅ 正确（查询 → 新增 → 更新 → 删除）：

- `getById` → `save` → `updateById` → `removeById`

### 2. 查询类内部：单条 → 列表 → 分页 → 其他

❌ 错误（分页在前、count 穿插）：

- `page` → `getById` → `count` → `list`

✅ 正确：

- `getById` → `list` → `page` → `count`

### 3. 组内：本身 → 带信息 → 信息量少到多

❌ 错误（带信息插在本身之前，信息量倒序）：

- `listWithUserAndRole` → `list` → `listWithUser`

✅ 正确：

- `list` → `listWithUser` → `listWithUserAndRole`

### 4. 删除：单条在批量前

❌ 错误：

- `removeByIds` → `removeById`

✅ 正确：

- `removeById` → `removeWithInfoById` → `removeByIds` → `removeWithInfoByIds`

### 5. 分类判定：命名优先于注解

```java
// ❌ 错误：confirmById 命名不在常见前缀表中，被误归为“其他”类
// ✅ 正确：状态变更语义归“更新”类；命名不清晰时可参考注解（@PutMapping）辅助判断
@PutMapping("/{id}/confirm")
public void confirmById(Long id) { ... }
```

同理，`enableById`、`approveById`、`publishById`、`renewById`、`disableById` 等状态变更方法均归**更新**，与其 HTTP 注解无关。

### 6. Controller 完整对照

❌ 错误顺序（删除在前、分页插在单条前、新增夹在中间）：

```java
@DeleteMapping("/{id}")
public void removeById(Long id) {}

@PostMapping
public void save(UserSavePO po) {}

@GetMapping
public Page<UserVO> page(UserPagePO po) {}

@GetMapping("/{id}")
public UserVO getById(Long id) {}

@PutMapping("/{id}")
public void updateById(Long id, UserUpdatePO po) {}
```

✅ 正确顺序：

```java
// 1) 查询-单条
@GetMapping("/{id}")
public UserVO getById(Long id) {}

// 2) 查询-分页
@GetMapping
public Page<UserVO> page(UserPagePO po) {}

// 3) 新增
@PostMapping
public void save(UserSavePO po) {}

// 4) 更新
@PutMapping("/{id}")
public void updateById(Long id, UserUpdatePO po) {}

// 5) 删除
@DeleteMapping("/{id}")
public void removeById(Long id) {}
```

### 7. Mapper XML 对照

❌ 错误（`<insert>` 在 `<select>` 之前，`<delete>` 夹在 `<select>` 中间）：

```xml
<insert id="save"> ... </insert>
<select id="getById"> ... </select>
<delete id="removeById"> ... </delete>
<select id="page"> ... </select>
```

✅ 正确（`resultMap` → `<select>` → `<insert>` → `<update>` → `<delete>`）：

```xml
<resultMap id="BaseResultMap" type="..."> ... </resultMap>

<!-- 查询：单条 → 列表 → 分页 → 其他 -->
<select id="getById"> ... </select>
<select id="list"> ... </select>
<select id="listWithUser"> ... </select>
<select id="page"> ... </select>
<select id="count"> ... </select>

<!-- 新增 -->
<insert id="save"> ... </insert>

<!-- 更新 -->
<update id="updateById"> ... </update>

<!-- 删除：单条 → 批量 -->
<delete id="removeById"> ... </delete>
<delete id="removeByIds"> ... </delete>
```

### 8. 边界：ServiceImpl 的 private 方法

❌ 错误：private 辅助方法按五类穿插在 public 方法之间

✅ 正确：public 方法按规则排序，private/protected 工具方法统一放文件末尾

### 9. 边界：不适用对象

❌ 错误：强行给工具类、枚举、PO/DTO 套用五类排序

✅ 正确：工具类按职责分组；枚举按值语义排列；纯数据对象无需排序规则

### 10. 综合示例：完整方法清单排序

同一业务域（如 `User`）的典型完整排序：

1) 查询-单条（本身 → 带信息）

- `getById`
- `getByCode`
- `getWithJobById`
- `getWithJobAndUserById`

2) 查询-列表（本身 → 带信息）

- `list`
- `listWithUser`
- `listWithUserAndRole`

3) 查询-分页（本身 → 带信息）

- `page`
- `pageWithUser`
- `pageWithUserAndRole`

4) 查询-其他

- `count`
- `existsByCode`

5) 新增（本身 → 带信息）

- `save`
- `saveWithInfo`
- `saveWithJobAndUser`

6) 更新（本身 → 带信息）

- `updateById`
- `updateWithInfoById`
- `updateWithJobAndUserById`

7) 删除（单条 → 批量，本身 → 带信息）

- `removeById`
- `removeWithInfoById`
- `removeByIds`
- `removeWithInfoByIds`

8) 其他

- `upload`
- `download`

## 九、执行与验收清单

- [ ] 顶层顺序：查询 → 新增 → 更新 → 删除 → 其他
- [ ] 查询内部：单条 → 列表 → 分页 → 其他
- [ ] 同组内部：单条优先、本身优先、With 信息量少 → 多
- [ ] 删除：单条在批量前
- [ ] 状态变更类方法（enable/approve/publish/renew 等）归更新
- [ ] 跨层对齐：Controller/Service/Mapper/Feign 同组 API 顺序一致
- [ ] 新增方法插入了正确位置，而非追加到末尾
- [ ] 仅移动方法块，不改动注解/签名/路由/逻辑
- [ ] private/protected 工具方法位于文件末尾
