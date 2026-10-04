---
name: "db-design-standard"
description: "关系型数据库（MySQL、PostgreSQL、Oracle、SQL Server、GaussDB、达梦等）表设计与 SQL 脚本规范：建表设计规则、SQL 编写规约、辅助索引设计指南、DDL 变更规范与验收清单。当需要设计新表、修改表结构、生成或审查 DDL/DML 脚本时使用。"
---

# 数据库脚本设计规范与工作流

本 skill 用于指导数据库设计和 SQL 脚本生成，确保所有变更符合规范。

## 何时使用

- 设计新表或拆分主子表
- 修改现有表结构（加字段、改类型、加索引，遵循 `references/ddl-change-guide.md`）
- 生成 DDL/DML 脚本（建表、数据变更）
- 编写或审查 SQL 查询/变更语句（遵循 `references/sql-writing-rules.md`）
- 审查数据库脚本是否符合规范
- 咨询字段类型 / 命名 / 索引 / 审计字段等问题

## 适用范围

### 适用对象（本规则适用于）

**数据库类型（全部关系型数据库）**

- MySQL 5.7 / 8.x
- PostgreSQL
- Oracle、SQL Server
- GaussDB、达梦、人大金仓等国产数据库
- SQLite、H2 等嵌入式数据库
- 其他关系型数据库

> 10 条设计规则本身与数据库无关；示例默认以 MySQL 语法呈现，使用其他数据库时按下方映射表替换对应类型。

**脚本类型**

- 建表 DDL（`CREATE TABLE` / `ALTER TABLE`）
- 数据变更 DML（`INSERT` / `UPDATE` / `DELETE`）

**数据库适配说明**

示例默认以 MySQL 语法呈现。各数据库类型对应关系如下，生成脚本时按目标数据库严格替换，避免混用方言语法：

**类型降级原则**：目标数据库不支持的类型均可退化为字符串类型（按目标库选用 `varchar` / `varchar2` / `nvarchar` / `text` / `clob` 等，长度按内容评估），由应用层负责格式解析与校验；各类型的优先降级方案见对应规则（布尔/枚举见规则 3、JSON 见规则 9）。

| 概念 | MySQL | PostgreSQL | Oracle | SQL Server | GaussDB | 达梦 | 人大金仓 | SQLite | H2 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 主键 / 关联编号（应用层生成） | `bigint unsigned` | `bigint` | `number(20)` | `bigint` | `bigint` | `bigint` | `bigint` | `integer` | `bigint` |
| 枚举取值（字符串设计，默认） | `varchar(32)` | `varchar(32)` | `varchar2(32)` | `nvarchar(32)` | `varchar(32)` | `varchar2(32)` | `varchar2(32)` / `varchar(32)` | `text` | `varchar(32)` |
| 布尔标志（如 is_deleted） | `tinyint unsigned` | `boolean` | `number(1)` | `bit` | `boolean` | `bit` | `boolean` / `number(1)` | `integer` | `boolean` |
| JSON 扩展 | `json` | `jsonb` | `clob` + IS JSON 校验 | `nvarchar(max)` | `json` | `clob` | `jsonb` / `clob` | `text` | `json` |
| 时间 | `datetime` | `timestamp` | `date` | `datetime2` | `timestamp` | `datetime` | `timestamp` / `date` | `text`（ISO 8601） | `timestamp` |
| 标识符引用符 | 反引号 | 双引号 | 双引号 | 方括号 | 双引号 | 双引号 | 双引号 | 双引号 / 方括号 / 反引号 | 双引号 |
| 表与字段注释 | 内联 `COMMENT` | `COMMENT ON` | `COMMENT ON` | 扩展属性 | `COMMENT ON` | `COMMENT ON` | `COMMENT ON` | 不支持（用 `--` 注释） | 内联 `COMMENT` |
| 建表参数 | `ENGINE=InnoDB` | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 |

注：

- 人大金仓存在 Oracle 兼容模式与 PostgreSQL 兼容模式，类型按项目实际模式选择（表中 `/` 前后对应两种模式）
- GaussDB 指 openGauss；GaussDB for MySQL 按 MySQL 列取值
- Oracle JSON：12c+ 建议 `clob` + `CHECK (col IS JSON)` 约束；21c+ 可选原生 `json` 类型
- MySQL 的 `unsigned` 为方言：PostgreSQL、Oracle、SQL Server、SQLite 等不支持，使用对应整型并由应用层保证非负；雪花编号等应用层生成值本身为正，不受影响
- PostgreSQL 时区场景：需记录时区使用 `timestamptz`（`timestamp` 不带时区）；MySQL 的 `timestamp` 自带时区转换，规则 3 的表述以 MySQL 为例，其他库按映射表替换
- 布尔：MySQL 的 `bool` / `boolean` 为 `tinyint(1)` 别名（无原生布尔类型），统一使用 `tinyint unsigned`；其余数据库直接用表中布尔类型
- 枚举：MySQL `enum`、PostgreSQL / H2 `enum` 等原生枚举可直接选用；不支持原生枚举的数据库使用字符串设计（英文枚举值）
- JSON：原生支持 JSON 类型的数据库直接使用（MySQL `json`、PostgreSQL `jsonb` 等）；不支持的（Oracle、SQL Server、SQLite、达梦等）使用字符串类型存放（见规则 9）
- MySQL 8.0.17+ 官方对 `unsigned` 有保留意见（隐式转换风险），规则 3 的 `unsigned` 为本规范强制约定；涉及跨库迁移时评估

### 不适用（边界）

- 非关系型数据库（Redis、MongoDB、Elasticsearch 等 NoSQL）
- 查询优化与慢 SQL 分析（不属于表设计范畴；辅助索引的通用设计准则见 `references/index-design-guide.md`，不替代线上慢 SQL 定位）
- 业务代码中的 ORM 映射与实体类（参见 `java-feature-standard` skill）

## 核心工作流

1. **需求分析**：确认业务实体关系，确定是否需要拆分主子表（规则 7）
2. **目录确认**：确认项目约定的 SQL 脚本目录
3. **SQL 生成**：编写 DDL/DML，应用下方全部设计规则
4. **规范自检**：逐条核对 10 条规则与"三、验收清单"

## 一、设计规则（共 10 条）

### 规则 1：注释

- 表必须带 `COMMENT`（表注释）
- 字段必须带 `COMMENT`（字段注释，说明用途；布尔/枚举字段需说明各取值含义）
- 布尔/枚举字段注释按 `字段说明：值1（值1说明）、值2（值2说明）` 格式写明各取值，如 `状态：RUNNING（运行中）、STOPPED（已停止）`、`是否删除：0（未删除）、1（删除）`
- 修改字段含义或对状态字段追加取值时，必须同步更新字段注释

**正反例：**

```sql
-- ❌ 错误：表与字段均缺少注释
CREATE TABLE `user` (
  `id` bigint unsigned NOT NULL,
  `name` varchar(100) DEFAULT NULL
);

-- ✅ 正确：表与字段均带 COMMENT
CREATE TABLE `user` (
  `id` bigint unsigned NOT NULL COMMENT '编号',
  `name` varchar(100) DEFAULT NULL COMMENT '名称'
) COMMENT='用户表';
```

### 规则 2：主键

- 主键字段名为 `id`，类型 `bigint unsigned`
- 主键由应用层生成全局唯一编号（如雪花编号），禁止使用 `AUTO_INCREMENT`（其他数据库的对应自增机制同样禁止：PostgreSQL `IDENTITY` / `SERIAL`、Oracle `SEQUENCE` 等）
- 雪花编号生成器需处理时钟回拨（如备份节点容错、记录上次时间戳兜底）

**正反例：**

```sql
-- ✅ 正确：bigint unsigned 主键，编号由应用层生成（雪花编号）
`id` bigint unsigned NOT NULL COMMENT '编号',
PRIMARY KEY (`id`)

-- ❌ 错误：使用数据库自增
`id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '编号',
PRIMARY KEY (`id`)
```

### 规则 3：字段类型与取值

**物理类型选择：**

- 小数类型使用 `decimal`，禁止 `float` / `double`（存在精度损失，比较时结果不可靠；数据范围超出 `decimal` 时拆分为整数与小数分开存储）；精度按业务量级选择（如金额常用 `decimal(10,2)` / `decimal(19,4)`）
- 长度几乎相等的字符串使用 `char` 定长类型
- 变长字符串使用 `varchar`，长度不超过 5000；超过时改用 `text` 并独立成表（用主键关联），避免影响其他字段的索引效率；注意行大小限制（MySQL 单行不超过 65535 字节，utf8mb4 下每字符最多 4 字节）
- 非负数值使用 `unsigned`（MySQL 方言；其他数据库用对应整型并由应用层保证非负，见"数据库适配说明"）
- 时间字段使用 `datetime`，纯日期（无时间）使用 `date`；需记录时区信息时使用 `timestamp`（PostgreSQL 用 `timestamptz`）；禁止使用字符串类型存储时间日期

**非空与默认值：**

- 业务字段原则上 `NOT NULL` 并给出合理默认值（数值 `0`、布尔标志 `0`、枚举的默认状态值等；不建议字符串使用空串默认值，避免与 NULL 语义混淆）；仅当确需表达"未知 / 不适用"时才允许 `NULL`

**布尔与枚举取值**（布尔可视为二值枚举）：

**布尔字段：**

- 数据库支持布尔类型时直接使用（如 PostgreSQL / GaussDB `boolean`、SQL Server `bit`、H2 `boolean`）
- 不支持时使用 `tinyint unsigned`（0 表示否，1 表示是；其他数据库按"数据库适配说明"替换），并在注释中写明取值
- 命名见规则 8（`is_xxx` 建议）

**枚举字段**（按优先级选用）：

1. **原生枚举**：数据库支持枚举类型时可直接使用（如 MySQL `enum`、PostgreSQL / H2 `enum`）
2. **字符串（默认）**：不支持或需跨库一致时，使用 `varchar(32)` 存放英文枚举值（见名知意），取值与 Java 枚举名一一对应（见 `java-coding-standard`）
3. **数字（兜底）**：需要绝对性能的场景，使用 `tinyint unsigned` / `smallint unsigned` 等存放数字枚举值，从 0 开始编号

布尔/枚举字段的注释须按规则 1 的格式写明各取值。

**正反例：**

```sql
-- ❌ 错误：金额用浮点类型（精度损失，比较结果不可靠）
`amount` double DEFAULT NULL COMMENT '金额',

-- ✅ 正确：金额使用 decimal
`amount` decimal(10,2) DEFAULT NULL COMMENT '金额',

-- ❌ 错误：时间日期用字符串存储（无法利用时间函数与范围索引）
`birthday` varchar(20) DEFAULT NULL COMMENT '生日',

-- ✅ 正确：时间日期用时间类型
`birthday` date DEFAULT NULL COMMENT '生日',

-- ✅ 正确：布尔字段——支持布尔的数据库直接用布尔类型，注释写明取值
`is_enabled` boolean DEFAULT NULL COMMENT '是否启用：true（是）、false（否）',

-- ✅ 正确：布尔字段——不支持布尔的数据库用 tinyint unsigned，注释写明取值
`is_enabled` tinyint unsigned DEFAULT NULL COMMENT '是否启用：0（否）、1（是）',

-- ❌ 错误：枚举字段注释未写明取值含义（看到值无从得知其含义）
`status` tinyint unsigned DEFAULT NULL COMMENT '状态',

-- ✅ 正确：枚举字段——字符串设计（默认），英文枚举值 + 注释写明取值
`status` varchar(32) DEFAULT NULL COMMENT '状态：RUNNING（运行中）、STOPPED（已停止）',

-- ✅ 正确：枚举字段——数字设计（兜底，绝对性能场景），注释同样写明取值
`status` tinyint unsigned DEFAULT NULL COMMENT '状态：0（运行中）、1（已停止）',
```

**数值类型范围参考（unsigned 可避免误存负数且扩大表示范围）：**

| 对象   | 年龄区间    | 类型                  | 字节  | 表示范围            |
| ---- | ------- | ------------------- | --- | --------------- |
| 人    | 150 岁之内 | `tinyint unsigned`  | 1   | 0 到 255         |
| 龟    | 数百岁     | `smallint unsigned` | 2   | 0 到 65535       |
| 恐龙化石 | 数千万年    | `int unsigned`      | 4   | 0 到约 43 亿       |
| 太阳   | 约 50 亿年 | `bigint unsigned`   | 8   | 0 到约 10 的 19 次方 |

### 规则 4：审计字段

审计字段按表特性选用（不要求每张表全部包含），统一置于字段末尾：

| 字段 | 适用范围 |
| --- | --- |
| `is_deleted` | 启用逻辑删除的表；不做逻辑删除的表可省略 |
| `created_by` / `created_at` | 需记录创建信息的表（业务表默认包含） |
| `updated_by` / `updated_at` | 存在更新操作的表；只增不改的表（日志表、流水表等）可省略 |
| `deleted_by` / `deleted_at` | 可选扩展：需追溯"谁删除、何时删除"的表（在启用逻辑删除的前提下） |

字段定义：

```sql
`is_deleted` tinyint unsigned DEFAULT 0 COMMENT '是否删除：0（未删除）、1（删除）',
`created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
`created_at` datetime DEFAULT NULL COMMENT '创建时间',
`updated_by` varchar(32) DEFAULT NULL COMMENT '更新人',
`updated_at` datetime DEFAULT NULL COMMENT '更新时间'
```

可选扩展字段（需追溯"谁删除、何时删除"时，置于 `updated_at` 之后）：

```sql
`deleted_by` varchar(32) DEFAULT NULL COMMENT '删除人',
`deleted_at` datetime DEFAULT NULL COMMENT '删除时间'
```

> 注：使用逻辑删除（而非物理删除）后数据可追溯，但被删除记录仍占用唯一值、同值无法再次插入，需根据情况酌情处理（常见解法：唯一索引纳入 `is_deleted` 列、删除时改写唯一值等）。
> 注：`is_deleted = 1` 时同步写入 `deleted_by` / `deleted_at`；恢复（取消删除）时应一并清空。
> 注：审计字段由持久层框架统一自动填充（如 MyBatis-Plus `MetaObjectHandler` 填充 `created_at` / `updated_at`、逻辑删除拦截），避免应用层遗漏（Java 实现见 `java-feature-standard`）。

**正反例：**

```sql
-- ❌ 错误：可更新业务表缺少审计字段
CREATE TABLE `trade_order` (
  `id` bigint unsigned NOT NULL COMMENT '编号',
  `amount` decimal(10,2) DEFAULT NULL COMMENT '金额'
);

-- ✅ 正确：可更新业务表包含 5 个审计字段（is_deleted / created_by / created_at / updated_by / updated_at）
CREATE TABLE `trade_order` (
  `id` bigint unsigned NOT NULL COMMENT '编号',
  `amount` decimal(10,2) DEFAULT NULL COMMENT '金额',
  `is_deleted` tinyint unsigned DEFAULT 0 COMMENT '是否删除：0（未删除）、1（删除）',
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `created_at` datetime DEFAULT NULL COMMENT '创建时间',
  `updated_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `updated_at` datetime DEFAULT NULL COMMENT '更新时间'
);

-- ✅ 正确：只增不改且不做逻辑删除的表，仅含创建字段
CREATE TABLE `operation_log` (
  `id` bigint unsigned NOT NULL COMMENT '编号',
  `content` varchar(500) DEFAULT NULL COMMENT '内容',
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `created_at` datetime DEFAULT NULL COMMENT '创建时间'
);
```

### 规则 5：外键

**严禁**使用 `FOREIGN KEY` 约束，关联关系由应用层维护。

说明：外键与级联更新适用于单机低并发场景，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库的插入速度。

**正反例：**

```sql
-- ❌ 错误：使用外键约束（限制扩展、影响写入性能）
`user_id` bigint unsigned DEFAULT NULL COMMENT '用户编号',
CONSTRAINT `fk_order_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`),
-- 注：示例表名使用 trade_order（order 为 MySQL 保留字，见规则 8）

-- ✅ 正确：仅保留关联字段，关系由应用层维护
`user_id` bigint unsigned DEFAULT NULL COMMENT '用户编号',
```

### 规则 6：索引

建表时**仅创建主键索引与唯一约束索引**（唯一索引承担数据正确性职责，随建表一并创建）。普通辅助索引业务上线后根据慢查询分析再添加。

> 普通辅助索引的设计准则（组合索引、索引长度等）见 `references/index-design-guide.md`。

**正反例：**

```sql
-- ❌ 错误：建表时预建大量普通辅助索引（可能无效且拖慢写入）
PRIMARY KEY (`id`),
KEY `idx_user_id` (`user_id`),
KEY `idx_status` (`status`),
KEY `idx_created_at` (`created_at`)

-- ✅ 正确：仅主键索引 + 唯一约束索引；普通辅助索引待慢查询分析后按需添加
PRIMARY KEY (`id`),
UNIQUE KEY `uk_code` (`code`)
```

### 规则 7：主子表分离

禁止宽表。一对多关系必须拆分主子表。例如：订单头信息与订单明细必须分为 `order` 和 `order_item` 两张表。

**正反例：**

```sql
-- ❌ 错误：宽表（明细以编号列平铺，行数受限、扩展困难）
CREATE TABLE `trade_order` (
  `id` bigint unsigned NOT NULL COMMENT '编号',
  `item1_name` varchar(100) DEFAULT NULL COMMENT '明细1名称',
  `item1_price` decimal(10,2) DEFAULT NULL COMMENT '明细1价格',
  `item2_name` varchar(100) DEFAULT NULL COMMENT '明细2名称',
  `item2_price` decimal(10,2) DEFAULT NULL COMMENT '明细2价格'
);

-- ✅ 正确：拆分为主子表
CREATE TABLE `trade_order` (
  `id` bigint unsigned NOT NULL COMMENT '编号',
  `total_amount` decimal(10,2) DEFAULT NULL COMMENT '总金额'
);

CREATE TABLE `trade_order_item` (
  `id` bigint unsigned NOT NULL COMMENT '编号',
  `order_id` bigint unsigned NOT NULL COMMENT '订单编号',
  `name` varchar(100) DEFAULT NULL COMMENT '明细名称',
  `price` decimal(10,2) DEFAULT NULL COMMENT '明细价格'
  -- 子表按规则 4 按需含审计字段
);
```

**补充（参考）：**

- 多对多关系使用中间表，命名"两个表名拼接"（如 `user_role`），中间表按规则 4 按需含审计字段
- 单表行数预计超过 500 万行或容量超过 2GB 时，才推荐分库分表；若三年内达不到该量级，不要在创建表时就分库分表
- 字段允许适当冗余以提升查询性能，但必须满足：不是频繁修改的字段、不是唯一索引字段、不是 `varchar` 超长或 `text` 字段，且须考虑数据一致性

### 规则 8：表与字段命名

- 表名、字段名使用小写字母与数字；禁止数字开头；禁止两个下划线中间只出现数字（如 `level_3_name`）
- 表名使用单数名词，禁用复数
- 禁用数据库保留字（如 `desc`、`range`、`match`、`delayed`，以目标数据库官方保留字清单为准）
- 去除冗余表名前缀（表 `sys_user` 的字段不加 `user_` 前缀）
- 布尔字段建议使用 `is_xxx` 命名（非绝对，按实际情况设计），如 `is_deleted`；类型与注释见规则 3、规则 1（Java 侧字段命名与映射约定见 `java-feature-standard`）
- 表命名推荐"业务名称_表的作用"（如 `trade_config`）；库名与应用名尽量一致

**正反例：**

```sql
-- ❌ 错误：大写、驼峰、两个下划线中间只出现数字
-- 表 AliyunAdmin；字段 rdcConfig、level_3_name

-- ✅ 正确：小写字母与数字、数字不直接跟在字母段之间
-- 表 aliyun_admin；字段 rdc_config、level3_name

-- ❌ 错误：字段名冗余表名前缀
-- 表 sys_user：`user_name` varchar(100), `user_age` tinyint

-- ✅ 正确：无冗余前缀
-- 表 sys_user：`name` varchar(100), `age` tinyint
```

### 规则 9：JSON 扩展

不确定或易变的属性集合使用 JSON 承载，避免频繁改表：

- 数据库原生支持 JSON 类型时直接使用（如 MySQL `json`、PostgreSQL `jsonb`、GaussDB `json`、H2 `json`）
- 不支持时使用字符串类型存放（如 Oracle / 达梦 `clob`、SQL Server `nvarchar(max)`、SQLite `text`），必要时辅以格式校验（如 Oracle `IS JSON`、SQL Server `ISJSON`）

**检索边界**：JSON 字段只放"整体读写"的易变属性；频繁检索、排序、聚合的属性必须提升为独立列（必要时使用生成列建立索引），避免 JSON 内属性无法利用普通索引。

**正反例：**

```sql
-- ❌ 错误：易变属性做成固定列（每新增一个属性都要 ALTER TABLE）
`ext_field1` varchar(100) DEFAULT NULL COMMENT '扩展字段1',
`ext_field2` varchar(100) DEFAULT NULL COMMENT '扩展字段2',

-- ✅ 正确：原生支持 JSON 的数据库直接使用 json 类型
`extra_config` json DEFAULT NULL COMMENT '扩展配置',

-- ✅ 正确：不支持原生 JSON 的数据库使用字符串类型
`extra_config` text DEFAULT NULL COMMENT '扩展配置',

-- ❌ 错误：频繁检索的属性藏在 JSON 里，无法利用普通索引
`extra_config` json DEFAULT NULL COMMENT '扩展配置（含检索属性）',

-- ✅ 正确：频繁检索的属性提升为独立列
`extra_config` json DEFAULT NULL COMMENT '扩展配置',
`channel` varchar(32) DEFAULT NULL COMMENT '渠道：ONLINE（线上）、OFFLINE（线下）',
```

### 规则 10：字符集与排序

表和字段**禁止**显式设置 `CHARACTER SET` 和 `COLLATE`，应使用数据库默认配置（其他数据库同理：不逐表覆盖库级默认设置）。

> 库级默认字符集建议采用 `utf8mb4`，排序规则建议 `utf8mb4_0900_ai_ci`（MySQL 8 默认）；旧版本或特殊需求按项目统一（注意 `LENGTH()` 按字节计数，`CHARACTER_LENGTH()` 按字符计数）。

**正反例：**

```sql
-- ❌ 错误：显式设置字符集与排序规则（与库默认不一致时反而引发问题）
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户表';

-- ✅ 正确：使用数据库默认配置
) ENGINE=InnoDB COMMENT='用户表';
```

## 二、完整建表模板

```sql
CREATE TABLE `example_table` (
  `id` bigint unsigned NOT NULL COMMENT '编号',
  `code` varchar(32) NOT NULL COMMENT '编码',
  `name` varchar(64) NOT NULL COMMENT '名称',
  `description` varchar(256) DEFAULT NULL COMMENT '描述',
  `type` varchar(32) NOT NULL COMMENT '类型：TYPE_A（类型A）、TYPE_B（类型B）、TYPE_C（类型C）',
  `config` json DEFAULT NULL COMMENT '配置信息',
  -- 审计字段（按表特性增减，见规则 4）
  `is_deleted` tinyint unsigned DEFAULT 0 COMMENT '是否删除：0（未删除）、1（删除）',
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `created_at` datetime DEFAULT NULL COMMENT '创建时间',
  `updated_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `updated_at` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_code` (`code`) -- 业务唯一键（按需，见规则 6）
) ENGINE=InnoDB COMMENT='示例表';
```

## 三、验收清单

### 设计规则自检

- [ ] 规则 1：表与字段均带 COMMENT，布尔/枚举字段按『值（值说明）』格式写明各取值；字段含义变更时同步更新注释
- [ ] 规则 2：主键 `id` 为 `bigint unsigned`（应用层生成编号，无 `AUTO_INCREMENT`；雪花编号已处理时钟回拨）
- [ ] 规则 3：小数用 `decimal`、`varchar` 不超过 5000、非负字段 `unsigned`；业务字段 `NOT NULL` + 默认值；时间日期不用字符串存储
- [ ] 规则 3：布尔/枚举取值按优先级（原生布尔/枚举 → 字符串英文枚举值（默认）→ 数字（兜底））
- [ ] 规则 4：审计字段按表特性选用并置于末尾（可更新的表含 updated_by / updated_at；启用逻辑删除的表含 is_deleted；需追溯删除时含 deleted_by / deleted_at）
- [ ] 规则 5：无 `FOREIGN KEY` 约束
- [ ] 规则 6：建表仅主键索引与唯一约束索引，无预建普通辅助索引
- [ ] 规则 7：一对多关系已拆分主子表，无宽表；多对多使用中间表
- [ ] 规则 8：表与字段小写下划线、无冗余前缀；表名单数、无保留字；布尔字段建议 `is_xxx`
- [ ] 规则 9：易变属性集合使用 JSON（原生支持用 JSON 类型，不支持用字符串类型）；频繁检索属性已提升为独立列
- [ ] 规则 10：未显式设置 `CHARACTER SET` / `COLLATE`
- [ ] 通用：目标库不支持的类型均按降级原则处理（退化为字符串，或按规则指定的优先降级方案）

### SQL 编写自检（详见 `references/sql-writing-rules.md`）

- [ ] 无 `SELECT *`，查询列明确列出
- [ ] `count(*)` / `ISNULL()` 等 NULL 语义正确；多表列加别名限定
- [ ] 大批量 UPDATE/DELETE 分批执行；上线前 `EXPLAIN` 检查通过

### 索引设计自检（详见 `references/index-design-guide.md`）

- [ ] varchar 索引指定长度；组合索引等值条件前置、无等值条件时按区分度排序
- [ ] join 字段有索引；OR 条件已拆 UNION

### DDL 变更自检（详见 `references/ddl-change-guide.md`）

- [ ] 有备份/回滚方案；变更经评审
- [ ] 大表在线变更；变更后验证通过

## References（补充规范）

- `references/sql-writing-rules.md` — SQL 编写规约（SELECT *、count/NULL 判断、别名、大批量分批、EXPLAIN 等）
- `references/index-design-guide.md` — 辅助索引设计指南（唯一索引、组合索引、覆盖索引、OR 拆 UNION 等）
- `references/ddl-change-guide.md` — DDL 变更规范（备份回滚、评审、大表在线变更、变更后验证）
