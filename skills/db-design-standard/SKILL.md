---
name: "db-design-standard"
description: "用于数据库脚本设计和规范检查。当用户需要设计新表、修改数据库结构或生成SQL脚本时调用此skill。"
---

# 数据库脚本设计规范与工作流

本 Skill 用于指导数据库设计和 SQL 脚本生成，确保所有变更符合项目规范。

## 核心工作流

1.  **需求分析**：确认业务实体关系，确定是否需要拆分主子表（Rule #8）。
2.  **路径确认**：检查 `db/mysql/tds-connector/` 目录。
    *   **全量脚本**：更新 `tds-connector.sql`。
    *   **增量脚本**：在同级目录下新建 `version-increment.sql`（需确认具体命名习惯）。
3.  **SQL 生成**：编写 DDL 语句，并应用下方所有设计规范。
4.  **规范自检**：生成后逐条核对 11 条规则。

## 详细设计规范

### 1. 基础结构与命名
*   **主子表分离**：禁止宽表。例如，订单头信息和订单明细必须分为 `order` 和 `order_item` 两张表。
*   **表命名**：使用小写字母和下划线。
*   **字段命名 (Rule #9)**：去除冗余表名前缀。
    *   🔴 错误：表 `sys_user`，字段 `user_name`, `user_age`
    *   🟢 正确：表 `sys_user`，字段 `name`, `age`
*   **字符集与排序 (Rule #11)**：表和字段**禁止**显式设置 `CHARACTER SET` 和 `COLLATE`，应使用数据库默认配置。

### 2. 字段类型规范
*   **主键 (Rule #3)**：
    ```sql
    `id` varchar(32) NOT NULL COMMENT '主键ID',
    PRIMARY KEY (`id`)
    ```
    *注意：禁止使用 AUTO_INCREMENT。*
*   **状态/类型 (Rule #4)**：使用 `tinyint`，建议从 0 开始。
    *   示例：`status` tinyint DEFAULT 0 COMMENT '状态（0正常 1禁用）'
*   **JSON 扩展 (Rule #10)**：不确定或易变的属性集合使用 `json` 类型。
    *   示例：`extra_config` json DEFAULT NULL COMMENT '扩展配置'

### 3. 通用审计字段 (Rule #5)
**所有主表**必须在末尾包含以下字段：
```sql
`del_flag` tinyint DEFAULT 0 COMMENT '逻辑删除标识（0未删除，1删除）',
`create_by` varchar(32) DEFAULT NULL COMMENT '创建人',
`create_time` datetime DEFAULT NULL COMMENT '创建时间',
`update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
`update_time` datetime DEFAULT NULL COMMENT '更新时间'
```

### 4. 约束与索引
*   **外键 (Rule #6)**：**严禁**使用 `FOREIGN KEY` 约束，关联关系由应用层维护。
*   **索引 (Rule #7)**：建表时**仅创建主键索引**。业务上线后根据慢查询分析再添加辅助索引。

### 5. 脚本交付物 (Rule #1)
每次变更必须同时提供：
1.  **增量脚本**：仅包含本次变更的 DDL/DML。
2.  **全量脚本**：将变更合并到项目的初始化 SQL 文件中。

## 完整建表模板

```sql
CREATE TABLE `example_table` (
  `id` varchar(32) NOT NULL COMMENT '主键',
  `name` varchar(100) DEFAULT NULL COMMENT '名称',
  `type` tinyint DEFAULT 0 COMMENT '类型（0类型A 1类型B）',
  `config` json DEFAULT NULL COMMENT '配置信息',
  -- 审计字段
  `del_flag` tinyint DEFAULT 0 COMMENT '逻辑删除标识（0未删除，1删除）',
  `create_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT COMMENT='示例表';
```
