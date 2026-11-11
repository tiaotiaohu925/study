# 第 5 周第 1 天：IN 与 NOT EXISTS

## 一、IN 子查询

当子查询返回一列多个值时，可用 `IN` 判断外层值是否属于这个集合。

```sql
SELECT customer
FROM customers
WHERE customer_id IN (
  SELECT customer_id
  FROM orders
  WHERE status = 'paid'
);
```

即使子查询中同一个客户编号出现多次，外层客户表的一行记录也只会返回一次。

## 二、NOT EXISTS

`NOT EXISTS` 判断相关子查询是否不存在匹配行。

```sql
SELECT A.customer
FROM customers AS A
WHERE NOT EXISTS (
  SELECT 1
  FROM orders AS B
  WHERE B.customer_id = A.customer_id
);
```

关键点：

- 子查询中的 `B.customer_id = A.customer_id` 把内外表关联起来。
- `SELECT 1` 只表示存在性，不关心返回什么值。
- `NOT EXISTS` 适合表达“没有某个匹配记录”。

## 练习数据

### customers

| customer_id | customer | city |
|---:|---|---|
| 1 | 张三 | 上海 |
| 2 | 李四 | 北京 |
| 3 | 王五 | 上海 |
| 4 | 赵六 | 上海 |
| 5 | 孙七 | 深圳 |

### orders

| order_id | customer_id | amount | status |
|---:|---:|---:|---|
| 101 | 1 | 120 | paid |
| 102 | 1 | 80 | refunded |
| 103 | 2 | 200 | paid |
| 104 | 4 | 50 | paid |
| 105 | 3 | 30 | pending |

## 本轮 3 题

6. 使用 `IN` 子查询查询有过 paid 订单的客户姓名和城市。
7. 使用 `NOT EXISTS` 查询没有任何订单的客户姓名和城市。
8. 使用 `NOT EXISTS` 查询没有任何 refunded 订单的客户姓名和城市。

提交前检查：

- `IN` 子查询只返回一列。
- `NOT EXISTS` 子查询是否与外表正确关联。
- 条件是否写在相关子查询内部。
- 外层结果是否按客户去重。
