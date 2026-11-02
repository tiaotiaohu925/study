# 第 4 周第 1 天：JOIN 入门

## 一、为什么需要 JOIN

客户姓名保存在 `customers`，订单金额保存在 `orders`。如果结果同时需要客户姓名和订单信息，就要按照共同字段连接两张表。

```sql
SELECT C.customer, O.order_id, O.amount
FROM customers AS C
INNER JOIN orders AS O
  ON C.customer_id = O.customer_id;
```

`ON C.customer_id = O.customer_id` 是连接条件。它说明两边哪两列代表同一个客户。

## 二、JOIN 后先看结果粒度

连接后的常见粒度是“一个客户 × 一笔订单一行”：

- 张三有 2 笔订单，所以出现 2 行。
- 李四有 1 笔订单，所以出现 1 行。
- 王五没有订单，在 `INNER JOIN` 中不会出现。

连接后的行数不能只用客户数或订单数描述，必须说明结果粒度。

## 三、INNER JOIN 与 LEFT JOIN

| 类型 | 保留什么 | 未匹配时 |
|---|---|---|
| `INNER JOIN` | 只保留两边都匹配的行 | 直接丢弃 |
| `LEFT JOIN` | 保留左表全部行 | 右表字段显示为 `NULL` |

模板：

```sql
-- 只保留匹配行
SELECT 左表字段, 右表字段
FROM 左表
INNER JOIN 右表
  ON 左表.连接键 = 右表.连接键;

-- 保留左表全部行
SELECT 左表字段, 右表字段
FROM 左表
LEFT JOIN 右表
  ON 左表.连接键 = 右表.连接键;
```

## 四、ON 与 WHERE 的区别

- `ON` 属于连接条件，决定右表哪些行可以匹配左表。
- `WHERE` 过滤连接完成后的结果。

在 `LEFT JOIN` 中，把右表条件放在 `ON` 还是 `WHERE`，结果可能不同：

- 条件放在 `ON`：不满足条件的左表行通常仍可保留，只是右表字段为 `NULL`。
- 条件放在 `WHERE`：右表字段不满足条件的行会在连接后被过滤，可能连左表未匹配行也一起删掉。

## 练习数据

### customers

| customer_id | customer |
|---:|---|
| 1 | 张三 |
| 2 | 李四 |
| 3 | 王五 |

### orders

| order_id | customer_id | amount | status |
|---:|---:|---:|---|
| 101 | 1 | 120 | paid |
| 102 | 1 | 80 | refunded |
| 103 | 2 | 200 | paid |

## 本轮 3 题

1. 用 `INNER JOIN` 查询每笔订单对应的客户姓名、订单号和金额，并按 `order_id` 升序排列。
2. 以 `customers` 为左表使用 `LEFT JOIN`，查询客户姓名、订单号和金额。请同时说明：
   - 返回几行；
   - 哪名客户对应的订单字段为 `NULL`；
   - 张三为什么会出现不止一次。
3. 业务目标是“保留所有客户，只匹配并显示 `paid` 订单”。请写出使用 `LEFT JOIN` 的完整 SQL，并说明 `status = 'paid'` 应放在 `ON` 还是 `WHERE`，为什么。

先独立作答，不运行或核对答案。

## 第 2 轮：JOIN 与聚合

`LEFT JOIN` 产生未匹配行时，右表字段为 `NULL`。这会直接影响计数和求和：

- `COUNT(*)` 数结果行，所以未匹配客户也会被计为 1 行。
- `COUNT(B.order_id)` 只数右侧非 NULL 订单号，未匹配客户会得到 0。
- `SUM(B.amount)` 会忽略 NULL；一个客户完全没有任何匹配订单时，求和结果是 NULL，不是 0。
- 要找“右表没有匹配记录”的客户，可以在 `LEFT JOIN` 后使用 `WHERE B.订单键 IS NULL`。

继续使用上面的 `customers` 和 `orders` 表。

4. 查询所有客户及其 paid 订单数量，没有 paid 订单的客户必须保留并显示数量 0。返回客户姓名和付费订单数，并按客户编号升序排列。
5. 查询所有客户及其 paid 订单总金额。没有 paid 订单的客户也必须保留；请说明该客户的总金额结果为什么是 NULL，而不是 0。
6. 使用 `LEFT JOIN` 查询没有任何 paid 订单的客户姓名。请说明 `WHERE` 中应判断哪个字段为 NULL。
