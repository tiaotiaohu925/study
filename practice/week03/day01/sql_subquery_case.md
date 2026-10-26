# 第 3 周第 1 天：子查询、CASE、NULL

## 一、标量子查询

标量子查询返回单个值，可以放在 `WHERE` 或 `SELECT` 中比较。

示例，使用另一组演示数据：

```sql
SELECT order_id, amount
FROM orders
WHERE amount > (SELECT AVG(amount) FROM orders);
```

注意：子查询必须只返回一个值，否则不能直接用 `>` 比较。

## 二、IN 子查询

子查询返回一列多个值时，可以用 `IN`。

```sql
SELECT customer
FROM customers
WHERE customer_id IN (
  SELECT customer_id FROM orders WHERE status = 'paid'
);
```

## 三、EXISTS / NOT EXISTS

`EXISTS` 用来判断子查询是否至少返回一行。

```sql
SELECT A.customer
FROM customers AS A
WHERE EXISTS (
  SELECT 1 FROM orders AS B
  WHERE B.customer_id = A.customer_id
);
```

`NOT EXISTS` 用来找不存在匹配记录的客户。

## 四、CASE WHEN

```sql
CASE
  WHEN amount >= 200 THEN 'high'
  WHEN amount >= 100 THEN 'medium'
  ELSE 'low'
END
```

CASE 是从上到下判断，满足第一个条件后就不会继续。

## 五、NULL

- `NULL` 表示未知值，不是 0，也不是空字符串。
- 不能用 `status = NULL` 判断。
- 应使用 `status IS NULL` 或 `status IS NOT NULL`。
- `SUM`、`AVG`、`COUNT(column)` 通常会忽略 NULL；`COUNT(*)` 不会忽略行。

## 六、练习题

使用下面的 `customers` 和 `orders` 表：

### customers

| customer_id | customer | city |
|---:|---|---|
| 1 | 张三 | 上海 |
| 2 | 李四 | 北京 |
| 3 | 王五 | 上海 |
| 4 | 赵六 | 上海 |
| 5 | 孙七 | 深圳 |
| 6 | 周八 | 北京 |
| 7 | 吴九 | 杭州 |

### orders

| order_id | customer_id | amount | status | order_date |
|---:|---:|---:|---|---|
| 101 | 1 | 120 | paid | 2026-10-01 |
| 102 | 2 | 80 | paid | 2026-10-02 |
| 103 | 3 | 200 | refunded | 2026-10-03 |
| 104 | 4 | 150 | paid | 2026-10-04 |
| 105 | 5 | 90 | paid | 2026-10-05 |
| 106 | 6 | 300 | paid | 2026-10-06 |
| 107 | 1 | 50 | paid | 2026-10-07 |

写 5 条 SQL：

1. 查询金额高于全表平均订单金额的 `order_id`、`amount`、`status`。
2. 查询有过 `paid` 订单的客户姓名和城市。
3. 查询没有订单的客户姓名和城市。
4. 用 `CASE WHEN` 给每笔订单打标签：金额 >= 200 为 `high`，100–199 为 `medium`，小于 100 为 `low`。
5. 查询 `status IS NULL` 的订单，并说明返回结果说明了什么。

先自己写，不要查答案。完成后发我批改。
