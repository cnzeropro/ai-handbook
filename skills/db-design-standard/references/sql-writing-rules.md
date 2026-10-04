# SQL 编写规约

> 本文件为 `../SKILL.md` 的补充规范。示例以 MySQL 语法呈现；其他数据库按"数据库适配说明"对应替换（如 `ISNULL()` 对应 PostgreSQL 的 `IS NULL` / `COALESCE`，`IFNULL()` 对应 `COALESCE`）。

1. **禁止 `SELECT *`**：明确列出所需字段。理由：增加查询分析器解析成本；表增减字段易与 resultMap 配置不一致；无用字段增加网络消耗（尤其 `text` 类型字段）
2. **统计行数使用 `count(*)`**：不使用 `count(列名)` 或 `count(常量)`。`count(*)` 是 SQL92 标准语法，与数据库无关，统计含 NULL 的行；`count(列名)` 不统计该列 NULL 行
3. **`count(distinct col1, col2)` 注意 NULL**：其中一列全为 NULL 时，即使另一列有不同的值，结果也为 0
4. **`sum(col)` 注意 NPE**：该列全为 NULL 时返回 NULL 而非 0，兜底写法：

   ```sql
   SELECT IFNULL(SUM(column), 0) FROM table;
   ```

5. **NULL 判断使用 `ISNULL()`**：NULL 与任何值的直接比较结果均为 NULL（`NULL <> NULL` 返回 NULL 而非 false，`NULL <> 1` 返回 NULL 而非 true），用 `ISNULL(column)` 简洁易懂；`IS NULL` 同样可用（现代 MySQL 优化器两者等价）
6. **分页查询 count 为 0 时直接返回**：避免执行后面的分页语句
7. **禁止使用存储过程**：难以调试与扩展，且没有移植性
8. **数据订正先 `SELECT` 确认**：删除或修改记录前先查询确认无误，再执行更新语句
9. **大批量 UPDATE/DELETE 分批执行**：单次操作影响行数过大时（如超过数千行）必须分批执行，避免大事务长时间锁表、binlog 膨胀。可结合唯一键范围循环或分批 LIMIT 处理
10. **新 SQL 上线前 `EXPLAIN` 检查**：新增或修改的 SQL 上线前必须 EXPLAIN 确认执行计划，type 至少达到 `range`、避免全表扫描（性能目标见 `index-design-guide.md`）
11. **多表操作列名必须加表别名限定**：多表查询/更新/删除时操作列不加表名限定，一旦操作列在多个表中存在同名字段即抛异常：

    ```sql
    -- ❌ 错误：多表关联未加表别名限定，出现同名字段时报错
    -- Column 'name' in field list is ambiguous

    -- ✅ 正确：列名加表别名限定，且使用显式 JOIN（不使用逗号隐式连接）
    SELECT t1.name FROM first_table AS t1
    JOIN second_table AS t2 ON t1.id = t2.id;
    ```

12. **表别名前加 `as`，按 `t1` / `t2` / `t3` 顺序命名**（推荐）
13. **`in` 集合元素控制在 1000 个以内**（推荐）：若实在避免不了 `in`，需仔细评估集合元素数量
14. **不建议在开发代码中使用 `TRUNCATE TABLE`**（参考）：无事务且不触发 trigger，有可能造成事故
