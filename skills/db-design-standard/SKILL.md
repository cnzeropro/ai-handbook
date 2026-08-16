---
name: "db-design-standard"
description: "关系型数据库（MySQL、PostgreSQL、Oracle、SQL Server、GaussDB、达梦等）表设计与 SQL 脚本规范：11 条设计规则（交付物、注释、主键、字段类型、审计字段、约束索引、主子表拆分、命名、JSON 扩展、字符集）、完整建表模板与验收清单。当需要设计新表、修改表结构、生成或审查 DDL/DML 脚本时使用。"
---

# 数据库脚本设计规范与工作流

本 skill 用于指导数据库设计和 SQL 脚本生成，确保所有变更符合规范。

## 何时使用

- 设计新表或拆分主子表
- 修改现有表结构（加字段、改类型、加索引）
- 生成 DDL/DML 脚本（建表、数据变更、数据迁移）
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

> 11 条设计规则本身与数据库无关；示例默认以 MySQL 语法呈现，使用其他数据库时按下方映射表替换对应类型。

**脚本类型**

- 建表 DDL（`CREATE TABLE` / `ALTER TABLE`）
- 数据变更 DML（`INSERT` / `UPDATE` / `DELETE`，含存量数据迁移）
- 初始化脚本（全量 schema）
- 增量升级脚本（版本间变更）

**适用场景**

- 设计新表、拆分主子表
- 修改数据库结构（加字段、改类型、加索引）
- 生成或审查 SQL 脚本
- 咨询字段类型 / 命名 / 索引 / 审计字段问题

**数据库适配说明**

示例默认以 MySQL 语法呈现。各数据库类型对应关系如下，生成脚本时按目标数据库严格替换，避免混用方言语法：

| 概念 | MySQL | PostgreSQL | Oracle | SQL Server | GaussDB | 达梦 | 人大金仓 | SQLite | H2 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 字符串主键 / 状态与类型（枚举值） | `varchar(32)` | `varchar(32)` | `varchar2(32)` | `nvarchar(32)` | `varchar(32)` | `varchar2(32)` | `varchar2(32)` / `varchar(32)` | `text` | `varchar(32)` |
| 逻辑删除（0/1 标志） | `tinyint` | `smallint` | `number(1)` | `tinyint` | `tinyint` | `tinyint` | `number(1)` / `smallint` | `integer` | `tinyint` |
| JSON 扩展 | `json` | `jsonb` | `clob` + IS JSON 校验 | `nvarchar(max)` | `json` | `clob` | `jsonb` / `clob` | `text` | `json` |
| 时间 | `datetime` | `timestamp` | `date` | `datetime2` | `timestamp` | `datetime` | `timestamp` / `date` | `text`（ISO 8601） | `timestamp` |
| 标识符引用符 | 反引号 | 双引号 | 双引号 | 方括号 | 双引号 | 双引号 | 双引号 | 双引号 / 方括号 / 反引号 | 双引号 |
| 表与字段注释 | 内联 `COMMENT` | `COMMENT ON` | `COMMENT ON` | 扩展属性 | `COMMENT ON` | `COMMENT ON` | `COMMENT ON` | 不支持（用 `--` 注释） | 内联 `COMMENT` |
| 建表参数 | `ENGINE=InnoDB` | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 |

注：

- 人大金仓存在 Oracle 兼容模式与 PostgreSQL 兼容模式，类型按项目实际模式选择（表中 `/` 前后对应两种模式）
- GaussDB 指 openGauss；GaussDB for MySQL 按 MySQL 列取值
- Oracle JSON：12c+ 建议 `clob` + `CHECK (col IS JSON)` 约束；21c+ 可选原生 `json` 类型

### 不适用（边界）

- 非关系型数据库（Redis、MongoDB、Elasticsearch 等 NoSQL）
- 查询优化与慢 SQL 分析（不属于表设计范畴）
- 业务代码中的 ORM 映射与实体类（参见 `feature-coding-standard` skill）

## 核心工作流

1. **需求分析**：确认业务实体关系，确定是否需要拆分主子表（规则 8）
2. **目录确认**：确认项目约定的 SQL 脚本目录；全量脚本与增量脚本同目录管理（规则 1）
3. **SQL 生成**：编写 DDL/DML，应用下方全部设计规则
4. **规范自检**：逐条核对 11 条规则与"三、验收清单"

## 一、设计规则（共 11 条）

### 规则 1：脚本交付物

每次变更必须同时提供：

1. **增量脚本**：仅包含本次变更的 DDL/DML
2. **全量脚本**：将变更合并到项目的初始化 SQL 文件中

多数据库支持：项目同时使用多种数据库时，每种目标数据库的脚本均需提供。

**正反例：**

```
❌ 错误：只提交增量脚本，全量初始化脚本未同步
db/
└── v1.2-increment.sql

✅ 正确：增量 + 全量同时交付，并按数据库类型分开管理（目录名按项目实际数据库调整）
db/
├── mysql/
│   ├── schema.sql           # 全量（已合并本次变更）
│   └── v1.2-increment.sql   # 增量（本次变更）
└── gaussdb/
    ├── schema.sql
    └── v1.2-increment.sql
```

> 目录结构与脚本命名遵循项目现有约定（如初始化文件可能名为 `init.sql`），规则要点是"增量 + 全量成对交付"。

### 规则 2：注释

- 表必须带 `COMMENT`（表注释）
- 字段必须带 `COMMENT`（字段注释，说明用途；状态/类型字段需说明各取值含义）

**正反例：**

```sql
-- ❌ 错误：表与字段均缺少注释
CREATE TABLE `user` (
  `id` varchar(32) NOT NULL,
  `name` varchar(100) DEFAULT NULL
);

-- ✅ 正确：表与字段均带 COMMENT
CREATE TABLE `user` (
  `id` varchar(32) NOT NULL COMMENT '主键',
  `name` varchar(100) DEFAULT NULL COMMENT '名称'
) COMMENT='用户表';
```

### 规则 3：主键

- 主键使用 `varchar(32)`，由应用层生成（雪花 ID 等）
- 禁止使用 `AUTO_INCREMENT`

**正反例：**

```sql
-- ❌ 错误：自增整型主键
`id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键ID',

-- ✅ 正确：varchar(32) 字符串主键
`id` varchar(32) NOT NULL COMMENT '主键ID',
PRIMARY KEY (`id`)
```

### 规则 4：状态/类型字段

**第一设计（默认）：英文字符串值**，见名知意：

- 使用 `varchar(32)`（各库对应类型见"数据库适配说明"）
- 取值与 Java 枚举名一致（如 `WAITING` / `STAGED` / `DEPLOYED`），见 `feature-coding-standard`
- 便于排查与扩展，不依赖编号约定

**第二设计（备选）**：确有需要时（如历史兼容、强性能要求）才使用数字值：

- 使用 `tinyint`，从 0 开始编号

**正反例：**

```sql
-- ❌ 错误：状态存数字值，见名不知意（需查文档才知道 1 的含义）
`status` int DEFAULT 1 COMMENT '状态（1正常 2禁用）',

-- ✅ 正确：状态存英文字符串，见名知意
`status` varchar(32) DEFAULT NULL COMMENT '状态（NORMAL正常 DISABLED禁用）',
```

### 规则 5：通用审计字段

**所有主表**必须在末尾包含以下 5 个字段：

```sql
`is_deleted` tinyint DEFAULT 0 COMMENT '逻辑删除标识（0未删除，1删除）',
`create_by` varchar(32) DEFAULT NULL COMMENT '创建人',
`create_at` datetime DEFAULT NULL COMMENT '创建时间',
`update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
`update_at` datetime DEFAULT NULL COMMENT '更新时间'
```

**正反例：**

```sql
-- ❌ 错误：主表缺少审计字段
CREATE TABLE `order` (
  `id` varchar(32) NOT NULL COMMENT '主键',
  `amount` decimal(10,2) DEFAULT NULL COMMENT '金额'
);

-- ✅ 正确：末尾包含审计字段（is_deleted / create_by / create_at / update_by / update_at）
CREATE TABLE `order` (
  `id` varchar(32) NOT NULL COMMENT '主键',
  `amount` decimal(10,2) DEFAULT NULL COMMENT '金额',
  `is_deleted` tinyint DEFAULT 0 COMMENT '逻辑删除标识（0未删除，1删除）',
  `create_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_at` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_at` datetime DEFAULT NULL COMMENT '更新时间'
);
```

### 规则 6：外键

**严禁**使用 `FOREIGN KEY` 约束，关联关系由应用层维护。

**正反例：**

```sql
-- ❌ 错误：使用外键约束（限制扩展、影响写入性能）
`user_id` varchar(32) DEFAULT NULL COMMENT '用户ID',
CONSTRAINT `fk_order_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`),

-- ✅ 正确：仅保留关联字段，关系由应用层维护
`user_id` varchar(32) DEFAULT NULL COMMENT '用户ID',
```

### 规则 7：索引

建表时**仅创建主键索引**。业务上线后根据慢查询分析再添加辅助索引。

**正反例：**

```sql
-- ❌ 错误：建表时预建大量辅助索引（可能无效且拖慢写入）
PRIMARY KEY (`id`),
KEY `idx_user_id` (`user_id`),
KEY `idx_status` (`status`),
KEY `idx_create_at` (`create_at`)

-- ✅ 正确：仅主键索引；辅助索引待慢查询分析后按需添加
PRIMARY KEY (`id`)
```

### 规则 8：主子表分离

禁止宽表。一对多关系必须拆分主子表。例如：订单头信息与订单明细必须分为 `order` 和 `order_item` 两张表。

**正反例：**

```sql
-- ❌ 错误：宽表（明细以编号列平铺，行数受限、扩展困难）
CREATE TABLE `order` (
  `id` varchar(32) NOT NULL COMMENT '主键',
  `item1_name` varchar(100) DEFAULT NULL COMMENT '明细1名称',
  `item1_price` decimal(10,2) DEFAULT NULL COMMENT '明细1价格',
  `item2_name` varchar(100) DEFAULT NULL COMMENT '明细2名称',
  `item2_price` decimal(10,2) DEFAULT NULL COMMENT '明细2价格'
);

-- ✅ 正确：拆分为主子表
CREATE TABLE `order` (
  `id` varchar(32) NOT NULL COMMENT '主键',
  `total_amount` decimal(10,2) DEFAULT NULL COMMENT '总金额'
);

CREATE TABLE `order_item` (
  `id` varchar(32) NOT NULL COMMENT '主键',
  `order_id` varchar(32) NOT NULL COMMENT '订单ID',
  `name` varchar(100) DEFAULT NULL COMMENT '明细名称',
  `price` decimal(10,2) DEFAULT NULL COMMENT '明细价格'
);
```

### 规则 9：字段命名

- 使用小写字母和下划线
- 去除冗余表名前缀

**正反例：**

```sql
-- ❌ 错误：字段名冗余表名前缀
-- 表 sys_user：`user_name` varchar(100), `user_age` tinyint

-- ✅ 正确：无冗余前缀
-- 表 sys_user：`name` varchar(100), `age` tinyint
```

### 规则 10：JSON 扩展

不确定或易变的属性集合使用 JSON 类型（MySQL `json` / PostgreSQL `jsonb`），避免频繁改表。

**正反例：**

```sql
-- ❌ 错误：易变属性做成固定列（每新增一个属性都要 ALTER TABLE）
`ext_field1` varchar(100) DEFAULT NULL COMMENT '扩展字段1',
`ext_field2` varchar(100) DEFAULT NULL COMMENT '扩展字段2',

-- ✅ 正确：使用 json 类型承载易变配置
`extra_config` json DEFAULT NULL COMMENT '扩展配置',
```

### 规则 11：字符集与排序

表和字段**禁止**显式设置 `CHARACTER SET` 和 `COLLATE`，应使用数据库默认配置（其他数据库同理：不逐表覆盖库级默认设置）。

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
  `id` varchar(32) NOT NULL COMMENT '主键',
  `name` varchar(100) DEFAULT NULL COMMENT '名称',
  `type` varchar(32) DEFAULT NULL COMMENT '类型（TYPE_A类型A TYPE_B类型B）',
  `config` json DEFAULT NULL COMMENT '配置信息',
  -- 审计字段
  `is_deleted` tinyint DEFAULT 0 COMMENT '逻辑删除标识（0未删除，1删除）',
  `create_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_at` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_at` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='示例表';
```

## 三、验收清单

- [ ] 规则 1：增量 + 全量脚本成对交付（每种目标数据库各一份）
- [ ] 规则 2：表与字段均带 COMMENT，状态字段注释说明取值含义
- [ ] 规则 3：主键 `varchar(32)`，无 `AUTO_INCREMENT`
- [ ] 规则 4：状态/类型字段使用英文字符串值（数字值仅作第二设计备选）
- [ ] 规则 5：主表末尾包含 5 个审计字段（is_deleted / create_by / create_at / update_by / update_at）
- [ ] 规则 6：无 `FOREIGN KEY` 约束
- [ ] 规则 7：建表仅主键索引，无预建辅助索引
- [ ] 规则 8：一对多关系已拆分主子表，无宽表
- [ ] 规则 9：字段名小写下划线，无冗余表名前缀
- [ ] 规则 10：易变属性集合使用 `json` 类型
- [ ] 规则 11：未显式设置 `CHARACTER SET` / `COLLATE`
