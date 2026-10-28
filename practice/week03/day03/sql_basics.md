# 第 3 周第 3 天：SQL 基础查询与汇总

## 本块目标

- 正确组合 `WHERE`、`GROUP BY`、`HAVING` 和 `ORDER BY`。
- 分清行级过滤与分组级过滤。
- 用聚合函数回答业务问题，并写出完整可执行查询。

## 小例子：先确定执行顺序

使用另一组 `tickets` 数据：

| ticket_id | priority | minutes | status |
|---:|---|---:|---|
| 201 | normal | 30 | resolved |
| 202 | urgent | 45 | resolved |
| 203 | urgent | 20 | resolved |
| 204 | low | 10 | open |

查询至少解决两张已处理工单的优先级：

```sql
SELECT priority,
       COUNT(*) AS resolved_count,
       ROUND(AVG(minutes), 2) AS avg_minutes
FROM tickets
WHERE status = 'resolved'
GROUP BY priority
HAVING COUNT(*) >= 2
ORDER BY avg_minutes DESC;
```

结果：

| priority | resolved_count | avg_minutes |
|---|---:|---:|
| urgent | 2 | 32.5 |

逻辑处理顺序：

```text
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

- `WHERE` 在分组前过滤明细行。
- `GROUP BY` 把明细行分组。
- `HAVING` 在分组后过滤组。
- `ORDER BY` 最后排序结果。

## 本轮练习数据

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

## 本轮 3 题

1. 查询金额不低于 100 的 `paid` 订单，返回 `order_id`、`amount`、`order_date`。先按金额降序，金额相同时按订单号升序。
2. 按 `status` 汇总所有订单，返回状态、订单数、总金额和平均金额；平均金额保留两位小数，并按状态升序排列。
3. 只考虑金额不低于 90 的订单，按 `status` 汇总总金额，只保留总金额不低于 300 的状态，并按总金额降序排列。

先独立写出 3 条完整 SQL，不要先运行或核对答案。完成后发我批改。

## 第 3 轮：去重计数与多条件排序

### 小例子：`COUNT(*)` 与 `COUNT(DISTINCT 列)`

使用另一组访问数据：

| visit_id | user_id |
|---:|---:|
| 301 | 1 |
| 302 | 1 |
| 303 | 2 |
| 304 | 3 |

```sql
SELECT COUNT(*) AS visit_count,
       COUNT(DISTINCT user_id) AS user_count
FROM visits;
```

结果是 `4` 次访问、`3` 名不同用户。`COUNT(*)` 数行，`COUNT(DISTINCT user_id)` 先对非 NULL 用户编号去重，再计数。

### 第 3 轮 3 题

7. 查询有过 `paid` 订单的不同客户数量，只返回这个数量，并把结果列命名为 `paid_customer_count`。
8. 只考虑金额不低于 90 的订单，按 `status` 返回订单数、最高金额和最低金额；先按订单数降序，订单数相同时按状态升序。
9. 查询 `paid` 和 `refunded` 订单，返回 `order_id`、`status`、`amount`、`customer_id`；先按状态升序，再按金额降序，最后按订单号升序。

## 第 4 轮：聚合、去重计数与 IN 筛选

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

继续使用上面的 `orders` 表。

10. 查询所有 `paid` 订单的总订单数和不同客户数，结果列分别命名为 `order_count` 和 `customer_count`。
11. 按 `status` 统计每个状态的订单数和不同客户数，先按订单数降序，订单数相同时按状态升序。
12. 从 `customers` 中查询上海和北京客户，返回 `customer_id`、`customer`、`city`；先按城市升序，再按客户编号降序。

## 第 5 轮：分组口径与结果别名

继续使用上面的 `customers` 和 `orders` 表。

13. 只看 `paid` 订单，按 `customer_id` 汇总订单数和总金额；只保留至少有 2 笔 paid 订单的客户，并按总金额降序排列。
14. 按城市统计客户数量，返回城市和客户数；先按客户数降序，客户数相同时按城市升序。
15. 查询所有 `paid` 订单的订单数、总金额和平均金额，并计算涉及的不同客户数。结果列分别命名为 `order_count`、`total_amount`、`avg_amount`、`customer_count`；平均金额保留两位小数。
