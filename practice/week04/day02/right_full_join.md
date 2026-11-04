# 第 4 周第 2 天：RIGHT JOIN、FULL OUTER JOIN 与 COALESCE

## 一、RIGHT JOIN

`RIGHT JOIN` 保留右表全部行。右表记录如果找不到左表匹配，左表字段显示为 NULL。

```sql
SELECT C.customer, M.level
FROM customers AS C
RIGHT JOIN memberships AS M
  ON C.customer_id = M.customer_id;
```

结果：

| customer | level |
|---|---|
| 张三 | gold |
| 王五 | silver |
| NULL | bronze |

`RIGHT JOIN` 通常可以改成把两表顺序互换后的 `LEFT JOIN`：

```sql
SELECT C.customer, M.level
FROM memberships AS M
LEFT JOIN customers AS C
  ON C.customer_id = M.customer_id;
```

## 二、FULL OUTER JOIN

`FULL OUTER JOIN` 同时保留左右两边所有行：

- 两边匹配：正常连接。
- 只有左表存在：右表字段为 NULL。
- 只有右表存在：左表字段为 NULL。

标准 SQL 写法：

```sql
SELECT COALESCE(C.customer_id, M.customer_id) AS customer_id,
       C.customer,
       M.level
FROM customers AS C
FULL OUTER JOIN memberships AS M
  ON C.customer_id = M.customer_id
ORDER BY customer_id;
```

结果：

| customer_id | customer | level |
|---:|---|---|
| 1 | 张三 | gold |
| 2 | 李四 | NULL |
| 3 | 王五 | silver |
| 4 | NULL | bronze |

## 三、COALESCE

`COALESCE(表达式1, 表达式2, ...)` 返回第一个非 NULL 值。

```sql
COALESCE(C.customer_id, M.customer_id)
```

- 客户存在于左表时，返回 `C.customer_id`。
- 只有右表成员存在时，`C.customer_id` 为 NULL，于是返回 `M.customer_id`。

FULL OUTER JOIN 后如果直接选某一侧连接键，另一侧独有记录可能无法显示编号，因此常用 `COALESCE`。

## 四、数据库兼容性

`RIGHT JOIN` 和 FULL OUTER JOIN 的支持程度取决于数据库版本。MySQL 通常不支持 `FULL OUTER JOIN`，可用 `LEFT JOIN ... UNION ALL RIGHT JOIN ...` 并过滤右侧未匹配记录来模拟。具体写法要按实际数据库版本验证。

## 练习数据

### customers

| customer_id | customer | city |
|---:|---|---|
| 1 | 张三 | 上海 |
| 2 | 李四 | 北京 |
| 3 | 王五 | 上海 |

### memberships

| customer_id | level | points |
|---:|---:|---:|
| 1 | gold | 800 |
| 3 | silver | 400 |
| 4 | bronze | 100 |
| 5 | gold | 900 |

## 本轮 3 题

7. 以 `customers` 为左表、`memberships` 为右表使用 `RIGHT JOIN`，查询所有会员的 `customer` 和 `level`，包括没有客户记录的会员。按会员客户编号升序排列。
8. 使用标准 `FULL OUTER JOIN` 查询所有客户和会员，返回合并后的 `customer_id`、客户姓名、等级和积分。合并编号使用 `COALESCE`，按客户编号升序排列。
9. 把第 7 题改写成两表交换顺序后的 `LEFT JOIN`，保持返回结果含义相同。

先独立作答，不运行或核对答案。
