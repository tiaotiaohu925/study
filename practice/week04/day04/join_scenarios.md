# 第 4 周收口：多表 JOIN 业务场景

## 本轮目标

- 在四张表之间确定正确的连接链。
- 同时处理 JOIN、聚合、HAVING 和反连接。
- 区分“商品明细行”和“品类/客户汇总行”。
- 在提交前检查 `JOIN` 拼写、分组字段和 COUNT 对象。

## 数据

### customers

| customer_id | customer |
|---:|---|
| 1 | 张三 |
| 2 | 李四 |
| 3 | 王五 |
| 4 | 赵六 |

### orders

| order_id | customer_id | status | order_date |
|---:|---:|---|---|
| 101 | 1 | paid | 2026-10-01 |
| 102 | 1 | paid | 2026-10-10 |
| 103 | 2 | paid | 2026-10-02 |
| 104 | 3 | refunded | 2026-10-03 |
| 105 | 4 | paid | 2026-10-04 |

### order_items

| item_id | order_id | product_id | quantity | unit_price |
|---:|---:|---:|---:|---:|
| 1101 | 101 | 10 | 2 | 50 |
| 1102 | 101 | 20 | 1 | 80 |
| 1103 | 102 | 10 | 1 | 60 |
| 1104 | 103 | 30 | 3 | 100 |
| 1105 | 104 | 20 | 1 | 80 |
| 1106 | 105 | 20 | 2 | 70 |

### products

| product_id | product_name | category |
|---:|---|---|
| 10 | 耳机 | Electronics |
| 20 | 台灯 | Home |
| 30 | 哑铃 | Sports |
| 40 | 花盆 | Garden |

连接链：

```text
customers.customer_id → orders.customer_id
orders.order_id       → order_items.order_id
products.product_id   → order_items.product_id
```

商品金额：

```sql
order_items.quantity * order_items.unit_price
```

## 本轮 3 题

15. 只考虑 paid 订单，按商品品类汇总 paid 收入。返回品类和收入，并按收入降序排列。结果一行代表一个品类。
16. 查询至少有 2 笔 paid 订单的客户，返回客户姓名、paid 订单数和 paid 收入，并按收入降序排列。注意一张订单可能包含多条商品明细。
17. 使用 LEFT JOIN 反连接查询从未出现在 paid 订单中的商品，返回商品名称。说明最终结果行代表什么。

先独立作答，不运行或核对答案。

## 第二轮：`18–20`

继续使用上面的四张表。

18. 查询有 paid 销售的商品，返回商品名称、paid 订单数、销售件数和 paid 收入；按 paid 收入降序排列。注意一张 paid 订单可能包含多条商品明细。
19. 查询 paid 收入不低于 150 的客户，返回客户姓名和 paid 收入，并按收入降序排列。说明结果一行代表什么。
20. 查询发生退款订单（`status = 'refunded'`）的商品，返回商品名称、退款订单数、退款件数和退款金额，并按退款金额降序排列。

提示：

- 第 18 题是商品粒度，不是订单明细粒度。
- `COUNT(DISTINCT order_id)` 防止一张订单的多条商品把订单数重复计算。
- 第 20 题的退款收入仍然是 `quantity * unit_price` 的汇总。

## 第三轮：`21–23`

### 条件聚合示例

需要在一张汇总表中同时放置不同状态金额时，可以使用条件聚合：

```sql
SUM(CASE WHEN 条件 THEN 金额 ELSE 0 END)
```

例如：

```sql
SUM(CASE WHEN O.status = 'paid' THEN I.quantity * I.unit_price ELSE 0 END)
```

它只累加 paid 明细，其他状态按 0 计入。

继续使用上面的四张表。

21. 按商品品类查询 paid 客户数和 paid 收入；只保留至少有 2 名不同 paid 客户的品类，并按 paid 收入降序排列。
22. 查询所有客户的 paid 金额和 refunded 金额。没有对应订单的客户保留并显示 0；返回客户姓名、paid 金额、refunded 金额，并按客户编号升序排列。
23. 查询订单日期不早于 `2026-10-04` 的 paid 订单，按订单汇总客户姓名、订单号、商品件数和订单收入；按订单收入降序排列。

## 最后一轮：`24–25`

继续使用上面的四张表。最后两题需要检查：

- 左表是否完整保留；
- 条件聚合是否只累计目标状态；
- 多商品订单是否会重复计数；
- 每个非聚合输出字段是否进入 GROUP BY。

24. 按商品汇总 paid 表现，返回商品名称、paid 订单数、paid 销售件数和 paid 收入。没有任何 paid 销量的商品也必须保留，三个汇总值都显示 0；按商品编号升序排列。
25. 查询所有客户的 paid 金额和 refunded 金额，并计算净收入 `paid 金额 - refunded 金额`。没有对应订单的金额显示 0；返回客户姓名、paid 金额、refunded 金额和净收入，并按净收入降序排列。
