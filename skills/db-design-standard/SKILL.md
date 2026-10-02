---
name: "db-design-standard"
description: "关系型数据库（MySQL、PostgreSQL、Oracle、SQL Server、GaussDB、达梦等）表设计与 SQL 脚本规范：10 条设计规则（注释、主键、字段类型、审计字段、外键、索引、主子表拆分、命名、JSON 扩展、字符集）、SQL 编写规约、辅助索引设计指南、完整建表模板与验收清单。当需要设计新表、修改表结构、生成或审查 DDL/DML 脚本时使用。"
---

# 数据库脚本设计规范与工作流

本 skill 用于指导数据库设计和 SQL 脚本生成，确保所有变更符合规范。

## 何时使用

- 设计新表或拆分主子表
- 修改现有表结构（加字段、改类型、加索引）
- 生成 DDL/DML 脚本（建表、数据变更）
- 编写或审查 SQL 查询/变更语句（count / NULL 判断 / 多表别名等书写规约）
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

**适用场景**

- 设计新表、拆分主子表
- 修改数据库结构（加字段、改类型、加索引）
- 生成或审查 SQL 脚本
- 咨询字段类型 / 命名 / 索引 / 审计字段问题

**数据库适配说明**

示例默认以 MySQL 语法呈现。各数据库类型对应关系如下，生成脚本时按目标数据库严格替换，避免混用方言语法：

| 概念 | MySQL | PostgreSQL | Oracle | SQL Server | GaussDB | 达梦 | 人大金仓 | SQLite | H2 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 主键 / 关联 ID（应用层生成） | `bigint unsigned` | `bigint` | `number(20)` | `bigint` | `bigint` | `bigint` | `bigint` | `integer` | `bigint` |
| 状态与类型（枚举值） | `varchar(32)` | `varchar(32)` | `varchar2(32)` | `nvarchar(32)` | `varchar(32)` | `varchar2(32)` | `varchar2(32)` / `varchar(32)` | `text` | `varchar(32)` |
| 逻辑删除（0/1 标志） | `tinyint unsigned` | `smallint` | `number(1)` | `tinyint` | `tinyint` | `tinyint` | `number(1)` / `smallint` | `integer` | `tinyint` |
| JSON 扩展 | `json` | `jsonb` | `clob` + IS JSON 校验 | `nvarchar(max)` | `json` | `clob` | `jsonb` / `clob` | `text` | `json` |
| 时间 | `datetime` | `timestamp` | `date` | `datetime2` | `timestamp` | `datetime` | `timestamp` / `date` | `text`（ISO 8601） | `timestamp` |
| 标识符引用符 | 反引号 | 双引号 | 双引号 | 方括号 | 双引号 | 双引号 | 双引号 | 双引号 / 方括号 / 反引号 | 双引号 |
| 表与字段注释 | 内联 `COMMENT` | `COMMENT ON` | `COMMENT ON` | 扩展属性 | `COMMENT ON` | `COMMENT ON` | `COMMENT ON` | 不支持（用 `--` 注释） | 内联 `COMMENT` |
| 建表参数 | `ENGINE=InnoDB` | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 | 省略 |

注：

- 人大金仓存在 Oracle 兼容模式与 PostgreSQL 兼容模式，类型按项目实际模式选择（表中 `/` 前后对应两种模式）
- GaussDB 指 openGauss；GaussDB for MySQL 按 MySQL 列取值
- Oracle JSON：12c+ 建议 `clob` + `CHECK (col IS JSON)` 约束；21c+ 可选原生 `json` 类型
- MySQL 的 `unsigned` 为方言：PostgreSQL、Oracle、SQL Server、SQLite 等不支持，使用对应整型并由应用层保证非负；雪花 ID 等应用层生成值本身为正，不受影响

### 不适用（边界）

- 非关系型数据库（Redis、MongoDB、Elasticsearch 等 NoSQL）
- 查询优化与慢 SQL 分析（不属于表设计范畴；辅助索引的通用设计准则见"五、辅助索引设计指南"，不替代线上慢 SQL 定位）
- 业务代码中的 ORM 映射与实体类（参见 `java-feature-coding-standard` skill）

## 核心工作流

1. **需求分析**：确认业务实体关系，确定是否需要拆分主子表（规则 7）
2. **目录确认**：确认项目约定的 SQL 脚本目录
3. **SQL 生成**：编写 DDL/DML，应用下方全部设计规则
4. **规范自检**：逐条核对 10 条规则与"三、验收清单"

## 一、设计规则（共 10 条）

### 规则 1：注释

- 表必须带 `COMMENT`（表注释）
- 字段必须带 `COMMENT`（字段注释，说明用途；状态/类型字段需说明各取值含义）
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
  `id` bigint unsigned NOT NULL COMMENT '主键ID',
  `name` varchar(100) DEFAULT NULL COMMENT '名称'
) COMMENT='用户表';
```

### 规则 2：主键

- 主键字段名为 `id`，类型 `bigint unsigned`
- 主键由应用层生成全局唯一 ID（如雪花 ID），禁止使用 `AUTO_INCREMENT`

**正反例：**

```sql
-- ✅ 正确：bigint unsigned 主键，ID 由应用层生成（雪花 ID）
`id` bigint unsigned NOT NULL COMMENT '主键ID',
PRIMARY KEY (`id`)

-- ❌ 错误：使用数据库自增
`id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
PRIMARY KEY (`id`)
```

### 规则 3：字段类型与取值

**物理类型选择：**

- 小数类型使用 `decimal`，禁止 `float` / `double`（存在精度损失，比较时结果不可靠；数据范围超出 `decimal` 时拆分为整数与小数分开存储）
- 长度几乎相等的字符串使用 `char` 定长类型
- 变长字符串使用 `varchar`，长度不超过 5000；超过时改用 `text` 并独立成表（用主键关联），避免影响其他字段的索引效率
- 非负数值使用 `unsigned`（MySQL 方言；其他数据库用对应整型并由应用层保证非负，见"数据库适配说明"）
- 时间字段使用 `datetime`；需记录时区信息时使用 `timestamp`

**状态/类型取值：**

**第一设计（默认）：英文字符串值**，见名知意：

- 使用 `varchar(32)`（各库对应类型见"数据库适配说明"）
- 取值与 Java 枚举名一致（如 `WAITING` / `STAGED` / `DEPLOYED`），见 `java-coding-standard`
- 便于排查与扩展，不依赖编号约定

**第二设计（备选）**：确有需要时（如历史兼容、强性能要求）才使用数字值：

- 使用 `tinyint`，从 0 开始编号

**正反例：**

```sql
-- ❌ 错误：金额用浮点类型（精度损失，比较结果不可靠）
`amount` double DEFAULT NULL COMMENT '金额',

-- ✅ 正确：金额使用 decimal
`amount` decimal(10,2) DEFAULT NULL COMMENT '金额',

-- ❌ 错误：状态存数字值，见名不知意（需查文档才知道 1 的含义）
`status` int DEFAULT 1 COMMENT '状态（1正常 2禁用）',

-- ✅ 正确：状态存英文字符串，见名知意
`status` varchar(32) DEFAULT NULL COMMENT '状态（NORMAL正常 DISABLED禁用）',
```

**数值类型范围参考（unsigned 可避免误存负数且扩大表示范围）：**

| 对象 | 年龄区间 | 类型 | 字节 | 表示范围 |
| --- | --- | --- | --- | --- |
| 人 | 150 岁之内 | `tinyint unsigned` | 1 | 0 到 255 |
| 龟 | 数百岁 | `smallint unsigned` | 2 | 0 到 65535 |
| 恐龙化石 | 数千万年 | `int unsigned` | 4 | 0 到约 43 亿 |
| 太阳 | 约 50 亿年 | `bigint unsigned` | 8 | 0 到约 10 的 19 次方 |

### 规则 4：通用审计字段

**所有主表**必须在末尾包含以下 5 个字段：

```sql
`is_deleted` tinyint unsigned DEFAULT 0 COMMENT '逻辑删除标识（0未删除，1删除）',
`create_by` varchar(32) DEFAULT NULL COMMENT '创建人',
`create_at` datetime DEFAULT NULL COMMENT '创建时间',
`update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
`update_at` datetime DEFAULT NULL COMMENT '更新时间'
```

> 注：使用逻辑删除（而非物理删除）后数据可追溯，但会使唯一键不再唯一（被删除记录仍占用唯一值），需根据情况酌情处理。

**正反例：**

```sql
-- ❌ 错误：主表缺少审计字段
CREATE TABLE `order` (
  `id` bigint unsigned NOT NULL COMMENT '主键ID',
  `amount` decimal(10,2) DEFAULT NULL COMMENT '金额'
);

-- ✅ 正确：末尾包含审计字段（is_deleted / create_by / create_at / update_by / update_at）
CREATE TABLE `order` (
  `id` bigint unsigned NOT NULL COMMENT '主键ID',
  `amount` decimal(10,2) DEFAULT NULL COMMENT '金额',
  `is_deleted` tinyint unsigned DEFAULT 0 COMMENT '逻辑删除标识（0未删除，1删除）',
  `create_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_at` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_at` datetime DEFAULT NULL COMMENT '更新时间'
);
```

### 规则 5：外键

**严禁**使用 `FOREIGN KEY` 约束，关联关系由应用层维护。

说明：外键与级联更新适用于单机低并发场景，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库的插入速度。

**正反例：**

```sql
-- ❌ 错误：使用外键约束（限制扩展、影响写入性能）
`user_id` bigint unsigned DEFAULT NULL COMMENT '用户ID',
CONSTRAINT `fk_order_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`),

-- ✅ 正确：仅保留关联字段，关系由应用层维护
`user_id` bigint unsigned DEFAULT NULL COMMENT '用户ID',
```

### 规则 6：索引

建表时**仅创建主键索引与唯一约束索引**（唯一索引承担数据正确性职责，随建表一并创建）。普通辅助索引业务上线后根据慢查询分析再添加。

> 普通辅助索引的设计准则（组合索引、索引长度等）见"五、辅助索引设计指南"。

**正反例：**

```sql
-- ❌ 错误：建表时预建大量普通辅助索引（可能无效且拖慢写入）
PRIMARY KEY (`id`),
KEY `idx_user_id` (`user_id`),
KEY `idx_status` (`status`),
KEY `idx_create_at` (`create_at`)

-- ✅ 正确：仅主键索引 + 唯一约束索引；普通辅助索引待慢查询分析后按需添加
PRIMARY KEY (`id`),
UNIQUE KEY `uk_code` (`code`)
```

### 规则 7：主子表分离

禁止宽表。一对多关系必须拆分主子表。例如：订单头信息与订单明细必须分为 `order` 和 `order_item` 两张表。

**正反例：**

```sql
-- ❌ 错误：宽表（明细以编号列平铺，行数受限、扩展困难）
CREATE TABLE `order` (
  `id` bigint unsigned NOT NULL COMMENT '主键ID',
  `item1_name` varchar(100) DEFAULT NULL COMMENT '明细1名称',
  `item1_price` decimal(10,2) DEFAULT NULL COMMENT '明细1价格',
  `item2_name` varchar(100) DEFAULT NULL COMMENT '明细2名称',
  `item2_price` decimal(10,2) DEFAULT NULL COMMENT '明细2价格'
);

-- ✅ 正确：拆分为主子表
CREATE TABLE `order` (
  `id` bigint unsigned NOT NULL COMMENT '主键ID',
  `total_amount` decimal(10,2) DEFAULT NULL COMMENT '总金额'
);

CREATE TABLE `order_item` (
  `id` bigint unsigned NOT NULL COMMENT '主键ID',
  `order_id` bigint unsigned NOT NULL COMMENT '订单ID',
  `name` varchar(100) DEFAULT NULL COMMENT '明细名称',
  `price` decimal(10,2) DEFAULT NULL COMMENT '明细价格'
);
```

**补充（参考）：**

- 单表行数预计超过 500 万行或容量超过 2GB 时，才推荐分库分表；若三年内达不到该量级，不要在创建表时就分库分表
- 字段允许适当冗余以提升查询性能，但必须满足：不是频繁修改的字段、不是唯一索引字段、不是 `varchar` 超长或 `text` 字段，且须考虑数据一致性

### 规则 8：表与字段命名

- 表名、字段名使用小写字母与数字；禁止数字开头；禁止两个下划线中间只出现数字（如 `level_3_name`）
- 表名使用单数名词，禁用复数
- 禁用数据库保留字（如 `desc`、`range`、`match`、`delayed`，以目标数据库官方保留字清单为准）
- 去除冗余表名前缀（表 `sys_user` 的字段不加 `user_` 前缀）
- 表达是与否的布尔字段使用 `is_xxx` 命名，类型为 `tinyint unsigned`（1 表示是，0 表示否），如 `is_deleted`（Java 侧字段命名与映射约定见 `java-feature-coding-standard`）
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

不确定或易变的属性集合使用 JSON 类型（MySQL `json` / PostgreSQL `jsonb`），避免频繁改表。

**正反例：**

```sql
-- ❌ 错误：易变属性做成固定列（每新增一个属性都要 ALTER TABLE）
`ext_field1` varchar(100) DEFAULT NULL COMMENT '扩展字段1',
`ext_field2` varchar(100) DEFAULT NULL COMMENT '扩展字段2',

-- ✅ 正确：使用 json 类型承载易变配置
`extra_config` json DEFAULT NULL COMMENT '扩展配置',
```

### 规则 10：字符集与排序

表和字段**禁止**显式设置 `CHARACTER SET` 和 `COLLATE`，应使用数据库默认配置（其他数据库同理：不逐表覆盖库级默认设置）。

> 库级默认字符集建议采用 `utf8mb4`（覆盖国际化字符与 emoji 存储；注意 `LENGTH()` 按字节计数，`CHARACTER_LENGTH()` 按字符计数）。

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
  `id` bigint unsigned NOT NULL COMMENT '主键ID',
  `name` varchar(100) DEFAULT NULL COMMENT '名称',
  `type` varchar(32) DEFAULT NULL COMMENT '类型（TYPE_A类型A TYPE_B类型B）',
  `config` json DEFAULT NULL COMMENT '配置信息',
  -- 审计字段
  `is_deleted` tinyint unsigned DEFAULT 0 COMMENT '逻辑删除标识（0未删除，1删除）',
  `create_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_at` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_at` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='示例表';
```

## 三、验收清单

- [ ] 规则 1：表与字段均带 COMMENT，状态字段注释说明取值含义；字段含义变更时同步更新注释
- [ ] 规则 2：主键 `id` 为 `bigint unsigned`（应用层生成 ID，无 `AUTO_INCREMENT`）
- [ ] 规则 3：小数用 `decimal`、`varchar` 不超过 5000、非负字段 `unsigned`
- [ ] 规则 3：状态/类型取值使用英文字符串值（数字值仅作第二设计备选）
- [ ] 规则 4：主表末尾包含 5 个审计字段（is_deleted / create_by / create_at / update_by / update_at）
- [ ] 规则 5：无 `FOREIGN KEY` 约束
- [ ] 规则 6：建表仅主键索引与唯一约束索引，无预建普通辅助索引
- [ ] 规则 7：一对多关系已拆分主子表，无宽表
- [ ] 规则 8：表与字段小写下划线、无冗余前缀；表名单数、无保留字；布尔字段 `is_xxx`
- [ ] 规则 9：易变属性集合使用 `json` 类型
- [ ] 规则 10：未显式设置 `CHARACTER SET` / `COLLATE`
- [ ] SQL 编写：`count(*)` 统计行数、`ISNULL()` 判空、多表列加别名、订正先 `SELECT`
- [ ] 索引设计（上线后）：varchar 索引指定长度、组合索引区分度最左、join 字段有索引

## 四、SQL 编写规约

> 示例以 MySQL 语法呈现；其他数据库按"数据库适配说明"对应替换（如 `ISNULL()` 对应 PostgreSQL 的 `IS NULL` / `COALESCE`，`IFNULL()` 对应 `COALESCE`）。

1. **统计行数使用 `count(*)`**：不使用 `count(列名)` 或 `count(常量)`。`count(*)` 是 SQL92 标准语法，与数据库无关，统计含 NULL 的行；`count(列名)` 不统计该列 NULL 行
2. **`count(distinct col1, col2)` 注意 NULL**：其中一列全为 NULL 时，即使另一列有不同的值，结果也为 0
3. **`sum(col)` 注意 NPE**：该列全为 NULL 时返回 NULL 而非 0，兜底写法：

   ```sql
   SELECT IFNULL(SUM(column), 0) FROM table;
   ```

4. **NULL 判断使用 `ISNULL()`**：NULL 与任何值的直接比较结果均为 NULL（`NULL <> NULL` 返回 NULL 而非 false，`NULL <> 1` 返回 NULL 而非 true），用 `ISNULL(column)` 简洁易懂且执行效率更高
5. **分页查询 count 为 0 时直接返回**：避免执行后面的分页语句
6. **禁止使用存储过程**：难以调试与扩展，且没有移植性
7. **数据订正先 `SELECT` 确认**：删除或修改记录前先查询确认无误，再执行更新语句
8. **多表操作列名必须加表别名限定**：多表查询/更新/删除时操作列不加表名限定，一旦操作列在多个表中存在同名字段即抛异常：

   ```sql
   -- ❌ 错误：多表关联未加表别名限定，出现同名字段时报错
   -- Column 'name' in field list is ambiguous

   -- ✅ 正确：列名加表别名限定
   SELECT t1.name FROM first_table AS t1, second_table AS t2 WHERE t1.id = t2.id;
   ```

9. **表别名前加 `as`，按 `t1` / `t2` / `t3` 顺序命名**（推荐）
10. **`in` 集合元素控制在 1000 个以内**（推荐）：若实在避免不了 `in`，需仔细评估集合元素数量
11. **不建议在开发代码中使用 `TRUNCATE TABLE`**（参考）：无事务且不触发 trigger，有可能造成事故

## 五、辅助索引设计指南

> 遵循规则 6：建表时创建主键索引与唯一约束索引；普通辅助索引在业务上线后根据慢查询分析按需添加。本指南为索引设计准则，不替代线上慢 SQL 定位分析。示例以 MySQL 呈现。

1. **唯一索引随建表创建**：业务上具有唯一特性的字段（含组合字段）必须建成唯一索引，随建表一并创建（规则 6 例外）。即使应用层做了校验，没有唯一索引也必然产生脏数据（墨菲定律）；唯一索引对插入速度的损耗可忽略，换取的可靠性明显
2. **join 约束**：超过三个表禁止 join；join 字段数据类型保持绝对一致；被关联字段必须有索引
3. **varchar 字段建索引必须指定索引长度**：按文本区分度决定长度（一般长度为 20 区分度可达 90% 以上），可用 `count(distinct left(列名, 索引长度)) / count(*)` 计算区分度
4. **页面搜索严禁左模糊或全模糊**：B-Tree 最左前缀特性决定左模糊无法使用索引，需要时走搜索引擎解决
5. **order by 利用索引有序性**：order by 字段应是组合索引的一部分并放在组合顺序最后，避免 filesort：

   ```sql
   -- ✅ 正确：where a = ? and b = ? order by c；索引：a_b_c

   -- ❌ 错误：范围查询后索引有序性无法利用
   -- WHERE a > 10 ORDER BY b；索引 a_b 无法用于排序
   ```

6. **利用覆盖索引避免回表**：查询列均在索引内时直接从索引返回（explain 的 extra 列出现 `using index`）
7. **超多分页用延迟关联优化**：MySQL 分页是取 offset+N 行再丢弃前 offset 行，offset 特别大时效率低下；要么控制返回总页数，要么先快速定位 id 段再关联：

   ```sql
   SELECT t1.* FROM 表1 AS t1, (SELECT id FROM 表1 WHERE 条件 LIMIT 100000, 20) AS t2 WHERE t1.id = t2.id;
   ```

8. **SQL 性能目标**：至少要达到 `range` 级别，要求 `ref` 级别，最好 `const`（explain 的 type 为 `index` 表示索引物理文件全扫描，速度很慢，仅比全表扫描好）
9. **组合索引区分度最高的在最左边**：存在等号与不等号混合条件时，等号条件的列前置（如 `where c > ? and d = ?`，即使 c 区分度更高，也建 `idx_d_c`）
10. **防止隐式转换导致索引失效**：比较双方字段类型不同会触发隐式转换，导致索引失效
11. **避免索引极端误区**（参考）：不宁滥勿缺（一个查询就建一个索引）、不吝啬创建（认为索引只消耗空间、拖慢写入）、不抵制唯一索引（认为唯一索引必须在应用层"先查后插"解决）
