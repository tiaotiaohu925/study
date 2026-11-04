# SQL JOIN 笔记

> 整理日期：2026-11-02
>
> 来源：第 4 周第 1 天 JOIN 练习、学习者原答与修订记录。
>
> 说明：本笔记由助手整理，不能单独作为独立掌握证据。

## 1. 为什么使用 JOIN

当结果同时需要多张表的字段时，需要按共同键连接。例如客户姓名在 `customers`，订单信息在 `orders`，共同键是 `customer_id`。

```sql
SELECT C.customer, O.order_id, O.amount
FROM customers AS C
INNER JOIN orders AS O
  ON C.customer_id = O.customer_id;
```

使用别名可以减少重复表名，并让连接键更清楚。

## 2. INNER JOIN 与 LEFT JOIN

| 类型 | 保留范围 | 未匹配结果 |
|---|---|---|
| `INNER JOIN` | 两边都匹配的行 | 不保留 |
| `LEFT JOIN` | 左表全部行 | 右表字段为 NULL |

示例结果：

- 王五没有订单，INNER JOIN 中不出现。
- LEFT JOIN 中王五保留，订单号、金额和状态为 NULL。
- 张三有两笔订单，连接后按“客户 × 订单”展开为两行。

## 3. 结果粒度

JOIN 后不能只问“返回多少客户”或“返回多少订单”，应说明结果粒度。

- “每笔匹配订单一行”时，一对多关系会增加行数。
- 如果只需要每个客户一行，需要聚合和正确的 `GROUP BY`。
- 统计客户数时使用 `COUNT(DISTINCT customer_id)`，不要假设 JOIN 后每名客户只出现一次。

## 4. ON 与 WHERE

- `ON` 定义连接条件，也决定右表哪些行可以参与匹配。
- `WHERE` 过滤连接完成后的结果。

目标：保留所有客户，但只匹配 paid 订单。

```sql
SELECT C.customer, O.order_id, O.amount
FROM customers AS C
LEFT JOIN orders AS O
  ON C.customer_id = O.customer_id
 AND O.status = 'paid';
```

如果写成：

```sql
LEFT JOIN orders AS O
  ON C.customer_id = O.customer_id
WHERE O.status = 'paid'
```

没有 paid 订单的客户，其 `O.status` 为 NULL；`WHERE` 中的 paid 条件会把该行过滤掉，结果就不再保留全部客户。

## 5. JOIN 后计数与求和

```sql
SELECT C.customer,
       COUNT(O.order_id) AS paid_order_count,
       SUM(O.amount) AS paid_total
FROM customers AS C
LEFT JOIN orders AS O
  ON C.customer_id = O.customer_id
 AND O.status = 'paid'
GROUP BY C.customer_id, C.customer;
```

关键点：

- `COUNT(O.order_id)` 只统计非 NULL 订单号，未匹配客户得到 0。
- 如果写 `COUNT(*)`，未匹配客户也会因为结果中有一行而计成 1。
- `SUM(O.amount)` 会忽略 NULL；整组没有金额时返回 NULL。
- 展示层面的 0 与聚合返回的 NULL 是两种含义，需要区分。

## 6. 用 LEFT JOIN 找没有匹配的记录

目标：找没有任何 paid 订单的客户。

```sql
SELECT C.customer
FROM customers AS C
LEFT JOIN orders AS O
  ON C.customer_id = O.customer_id
 AND O.status = 'paid'
WHERE O.order_id IS NULL;
```

不能简单用 `O.status NOT IN ('paid')`：

- 王五没有订单，状态为 NULL，NOT IN 的结果是 UNKNOWN。
- 张三虽然有 paid 订单，但也有 refunded 订单，按订单行过滤可能错误保留张三。

查找反连接时，优先判断右表的连接键或主键是否为 NULL。

## 7. 当前已知错误

1. 把 `LEFT JOIN` 的右表过滤条件放进 `WHERE`，导致左表客户被删除。
2. 在 `ON` 条件前错误写 `AND WHERE`。
3. 聚合查询漏写 `GROUP BY`，或漏选/漏分组普通字段。
4. 把 NULL 写成字符串 `'NULL'`。
5. 输入 `FROM`、`JOIN` 等关键字时拼写错误。

## 8. JOIN 提交前自检

- [ ] 表别名是否定义并使用一致？
- [ ] 连接键是否正确，且类型和含义一致？
- [ ] 应该使用 INNER JOIN 还是 OUTER JOIN？
- [ ] 连接后的结果粒度是什么？
- [ ] ON 中放的是匹配条件，WHERE 中放的是最终结果过滤条件吗？
- [ ] 一对多连接是否造成行数放大或重复计数？
- [ ] 聚合时普通字段是否都在 GROUP BY 中？
- [ ] RIGHT JOIN、FULL JOIN 后要检查哪一侧会出现 NULL？

## 9. RIGHT JOIN

`RIGHT JOIN` 保留右表全部行。右表有记录而左表没有匹配时，左表字段为 NULL。

```sql
SELECT C.customer, M.level
FROM customers AS C
RIGHT JOIN memberships AS M
  ON C.customer_id = M.customer_id;
```

如果某会员的 `customer_id` 在 `customers` 中不存在，结果中 `C.customer` 为 NULL，但会员记录仍保留。

RIGHT JOIN 通常可改写为两表交换后的 LEFT JOIN：

```sql
SELECT C.customer, M.level
FROM memberships AS M
LEFT JOIN customers AS C
  ON C.customer_id = M.customer_id;
```

## 10. FULL OUTER JOIN 与 COALESCE

`FULL OUTER JOIN` 同时保留左右两边全部行：

- 两边匹配：正常返回。
- 只在左表：右侧字段为 NULL。
- 只在右表：左侧字段为 NULL。

```sql
SELECT COALESCE(C.customer_id, M.customer_id) AS customer_id,
       C.customer,
       M.level
FROM customers AS C
FULL OUTER JOIN memberships AS M
  ON C.customer_id = M.customer_id
ORDER BY customer_id;
```

`COALESCE` 返回第一个非 NULL 值：

- 正常匹配或只有左表时，使用 `C.customer_id`。
- 只有右表时，`C.customer_id` 为 NULL，因此使用 `M.customer_id`。

FULL OUTER JOIN 的支持情况取决于数据库。MySQL 通常不支持它，可用 `LEFT JOIN ... UNION ALL RIGHT JOIN ... WHERE 左表键 IS NULL` 模拟；实际项目要按数据库版本选择写法。

## 11. 当前进度

- 已学习 INNER、LEFT、RIGHT、FULL OUTER JOIN 的保留范围和未匹配行为。
- 已学习 RIGHT JOIN 到交换表顺序后 LEFT JOIN 的改写。
- 已学习 `COALESCE` 与 FULL OUTER JOIN 的组合。
- 第 4 周共 9 道题最终通过；第 1、2 天首次完全正确分别为 1/6 和 0/3，表名与列引用精度仍需加强。
- 尚未系统学习：自连接、多表连接，以及反连接与聚合的复杂组合。
