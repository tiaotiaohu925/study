# 第 4 周第 3 天：JOIN 与 GROUP BY / HAVING / 反连接

## 一、先确定结果粒度

JOIN 后先问：

- 一行代表什么？
- 一条左表记录可能匹配几条右表记录？
- 最终需要明细粒度还是汇总粒度？

明细 JOIN 的结果可能是“客户 × 订单”或“门店 × 销售记录”；按门店汇总后，才会变成“一家门店一行”。

## 二、工作示例：LEFT JOIN 后聚合

### teams

| team_id | team |
|---:|---|
| 1 | Alpha |
| 2 | Beta |
| 3 | Gamma |

### players

| player_id | team_id | goals |
|---:|---:|---:|
| 201 | 1 | 2 |
| 202 | 1 | 3 |
| 203 | 2 | 1 |

```sql
SELECT M.team,
       COUNT(P.player_id) AS player_count,
       COALESCE(SUM(P.goals), 0) AS total_goals
FROM teams AS M
LEFT JOIN players AS P
  ON M.team_id = P.team_id
GROUP BY M.team_id, M.team
ORDER BY M.team_id;
```

结果：

| team | player_count | total_goals |
|---|---:|---:|
| Alpha | 2 | 5 |
| Beta | 1 | 1 |
| Gamma | 0 | 0 |

关键点：

- `GROUP BY` 包含分组键 `team_id` 和要输出的普通字段 `team`。
- `COUNT(P.player_id)` 只统计匹配到的球员编号；Gamma 得到 0。
- 如果使用 `COUNT(*)`，Gamma 也会因为 LEFT JOIN 产生一行而被误计为 1。
- `SUM(P.goals)` 对完全无匹配记录的分组返回 NULL；需要显示为 0 时使用 `COALESCE(..., 0)`。

## 三、HAVING 过滤分组

```sql
HAVING COUNT(P.player_id) >= 2
```

这是对聚合后的组进行过滤，不能写在 `WHERE` 中。

## 四、LEFT JOIN 反连接

目标：找右表没有匹配的左表记录。

```sql
WHERE P.player_id IS NULL
```

LEFT JOIN 产生未匹配行时，右侧主键为 NULL。判断右侧主键或连接键是否为 NULL，可以找出没有任何匹配记录的左侧对象。

## 练习数据

### shops

| shop_id | manager | city |
|---:|---|---|
| 1 | 小林 | 上海 |
| 2 | 小周 | 北京 |
| 3 | 小陈 | 上海 |

### sales

| sale_id | shop_id | amount |
|---:|---:|---:|
| 101 | 1 | 120 |
| 102 | 1 | 80 |
| 103 | 2 | 200 |

## 本轮 3 题

10. 查询所有门店的店长、销售笔数和销售总额。没有销售的门店必须保留：销售笔数显示 0，销售总额显示 0。按门店编号升序排列。
11. 查询销售笔数至少为 2 的门店，返回店长、销售笔数和销售总额，按销售总额降序排列。
12. 使用 LEFT JOIN 反连接查询没有任何销售记录的门店店长和城市。

先独立作答，不运行或核对答案。

## 第二部分：三表连接与结果粒度

### customers

| customer_id | customer |
|---:|---|
| 1 | 张三 |
| 2 | 李四 |
| 3 | 王五 |

### orders

| order_id | customer_id | status |
|---:|---:|---|
| 101 | 1 | paid |
| 102 | 1 | refunded |
| 103 | 2 | paid |

### order_items

| item_id | order_id | quantity | unit_price |
|---:|---:|---:|---:|
| 201 | 101 | 2 | 50 |
| 202 | 101 | 1 | 20 |
| 203 | 102 | 1 | 50 |
| 204 | 103 | 3 | 100 |

三表连接涉及两个连接条件：

```sql
FROM customers AS C
LEFT JOIN orders AS O
  ON C.customer_id = O.customer_id
LEFT JOIN order_items AS I
  ON O.order_id = I.order_id
```

连接后一行代表一条订单商品明细。商品金额不是现成字段，需要计算：

```sql
I.quantity * I.unit_price
```

聚合提示：

- 一张订单可能有多条商品明细，所以统计订单数要使用 `COUNT(DISTINCT O.order_id)`。
- 若需要保留没有 paid 订单的客户，应把 `O.status = 'paid'` 放在第一个 `ON` 中。
- `SUM(I.quantity * I.unit_price)` 对没有匹配明细的客户返回 NULL；需要显示 0 时使用 `COALESCE`。

13. 使用三表 `INNER JOIN` 查询客户姓名、订单号、商品编号和商品金额。商品金额使用 `quantity * unit_price`，结果列命名为 `line_amount`；按订单号升序、商品编号升序排列。
14. 查询所有客户的 paid 订单数和 paid 商品总额。没有 paid 订单的客户也要保留，订单数和金额均显示 0；按客户编号升序排列。
