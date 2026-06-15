# 临时虚表 WITH ... AS

## 1. 核心定义

`WITH AS` 定义了一个**临时的结果集**，它仅在当前查询（`SELECT`, `INSERT`, `UPDATE` 或 `DELETE`）执行期间存在。可以把它看作是一个“只用一次的临时视图”。

## 2. 标准语法结构
```sql
WITH 临时表名1 AS (
    -- 逻辑 1：比如统计球员获奖数
    SELECT playerID, COUNT(*) as awards FROM awardsplayers GROUP BY playerID
),
临时表名2 AS (
    -- 逻辑 2：引用临时表 1，或者查别的
    SELECT * FROM 临时表名1 WHERE awards > 5
)
-- 最终查询：把上面的积木拼起来
SELECT * FROM 临时表名2 JOIN people USING (playerID);
```

# 递归 CTE
**递归 CTE**是在查询中引用自身定义的公用表表达式，使用 `RECURSIVE` 关键字标识。它适用于处理树形结构（组织架构、菜单）、图结构（路径查询、网络拓扑）以及具有层级关系的数据。

## 初始值
- **定义**：递归的起点，也称为“种子查询`（Seed Query）`”
- **执行次数**：仅在整个递归过程中**执行一次**，且最先执行
- **书写位置**：位于 `UNION ALL` 关键字**上方**
- **约束**：不能引用 `CTE` 自身（即不能在初始值部分出现对递归 `CTE` 的引用）
```sql
WITH RECURSIVE emp_tree AS (
    -- 初始值：选取根节点
    SELECT id, name, manager_id, 1 AS level
    FROM employee
    WHERE manager_id IS NULL
    
    UNION ALL
    -- 递归部分：后续将继续填充
    ...
)
```
## UNION ALL
 - **定义**：将初始值与每一步递归产生的新结果进行**合并（追加）** 的纽带。
- **执行次数**：**循环执行**，直到某一轮不再产生新数据为止。
- **工作机制**：
    1. **读取上轮结果**：在每一次循环中，`UNION ALL` **下方**的递归查询会将“**上一轮迭代刚产生的新数据**”（即上一次迭代输出的结果集）作为输入，用于与原表进行连接或筛选。
    2. **向下拼接数据**：每轮迭代产生的新行，都会以追加的方式，**一行一行** 追加到最终结果集的底部。
    3. **终止条件**：
        - **自然终止**：当某次迭代的查询结果为空集（没有产生任何新行）时，循环自动结束。
        - **安全终止（推荐）**：通常在递归查询的 `WHERE` 子句中明确限制递归深度（例如 `level < 5` 或 `depth < 10`），避免因数据环路或异常造成无限循环。
```sql
WITH RECURSIVE emp_tree AS (
    -- 初始值
    SELECT id, name, manager_id, 1 AS level
    FROM employee
    WHERE manager_id IS NULL

    UNION ALL

    -- 递归部分：上一轮结果（e）作为驱动，查找其直接下属
    SELECT e.id, e.name, e.manager_id, t.level + 1
    FROM employee e
    JOIN emp_tree t ON e.manager_id = t.id
    WHERE t.level < 5   -- 安全终止，最多递归4层
)
SELECT * FROM emp_tree;
```

## 总结

| 组成部分      | 作用                                | 注意事项                        |
| --------- | --------------------------------- | --------------------------- |
| 初始值       | 确定递归的起点、定义结果集结构                   | 只执行一次，不能引用 CTE 自身           |
| UNION ALL | 连接初始值与每一轮递归结果，实现结果集的逐轮追加          | 必须使用 `UNION ALL`（而非 `UNION` |
| 递归查询      | 利用上一轮的新数据，迭代地产生下一层数据              | 需要加上深度限制或有效的终止条件            |
| 终止条件      | 当递归查询返回空集时自然结束，或通过 `WHERE` 主动限制深度 | 防止无限循环，提升性能                 |
掌握递归 `CTE` 的关键在于理解“每一轮只处理上一轮新产生的数据”，这与编程语言中的广度优先搜索（`BFS`）思想类似。通过初始值设定起点，然后反复利用自连接或子查询向下（或向上）遍历，即可轻松处理层级数据。