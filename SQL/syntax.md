

# Join 连接


## CROSS JOIN
**CROSS JOIN**（交叉连接）返回两个表的**笛卡尔积**。即左表的每一行与右表的每一行进行组合，结果集的行数 = 左表行数 × 右表行数。
```sql
-- 显式写法
SELECT *
FROM table1
CROSS JOIN table2;

-- 隐式写法（不推荐，容易误写为内连接）
SELECT *
FROM table1, table2;
```

# 函数
## cast
### 使用方法
  `CAST (expression AS target_data_type)`
  •  `expression` ：要转换的值、列名或表达式。
  •  `target_data_type` ：目标数据类型（如  `INT ,  VARCHAR ,  DECIMAL ,  DATE` 等）。
### 简单示例
  - 1. 字符串转为数字（方便数值计算）
    `CAST('123' AS INT)`                        -- 结果：整数 123
    `CAST('45.67' AS DECIMAL(10,2))`  -- 结果：小数 45.67

  - 2. 数字转为字符串（方便拼接）
    `CAST(2026 AS VARCHAR(4))`          -- 结果：字符 '2026'
  
  - 3. 字符串转为日期/时间
    `CAST('2026-06-15' AS DATE)`      -- 结果：DATE 类型值