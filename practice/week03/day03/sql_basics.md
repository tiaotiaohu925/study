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
