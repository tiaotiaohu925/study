# SQL 子查询、CASE、NULL 与 UNION

> 整理日期：2026-11-11
>
> 来源：第 5 周集中练习、学习者原答与修订记录。
>
> 说明：本笔记由助手整理，不能单独作为独立掌握证据。

## 1. 标量子查询

标量子查询返回一行一列，可以放在比较符一侧。

```sql
SELECT sale_id, amount
FROM sales
WHERE amount > (
  SELECT AVG(amount)
  FROM sales
);
```

常见错误：

- 子查询返回多行，不能直接放在 `>`、`=` 等标量位置。
- 子查询没有行时可能返回 NULL，比较结果为 UNKNOWN。
- 必须说明平均值口径，例如全表平均还是某一门店平均。

## 2. IN 子查询

```sql
SELECT customer, city
FROM customers
WHERE customer_id IN (
  SELECT customer_id
  FROM orders
  WHERE status = 'paid'
);
```

IN 用来判断外层值是否属于子查询结果集合。子查询中的重复值不会让外层客户重复返回。

注意：`NOT IN` 遇到子查询中的 NULL 时，可能因三值逻辑返回意外结果；找“不存在匹配记录”时优先考虑 `NOT EXISTS`。

## 3. EXISTS / NOT EXISTS

```sql
SELECT A.customer, A.city
FROM customers AS A
WHERE NOT EXISTS (
  SELECT 1
  FROM orders AS B
  WHERE B.customer_id = A.customer_id
    AND B.status = 'refunded'
);
```

- `EXISTS` 只关心是否存在至少一行。
- `NOT EXISTS` 表示不存在符合条件的行。
- 相关条件 `B.customer_id = A.customer_id` 必须写在子查询中。

## 4. CASE WHEN

```sql
CASE status
  WHEN 'paid' THEN 'success'
  WHEN 'refunded' THEN 'refund'
  ELSE 'unknown'
END AS status_label
```

CASE 从上到下判断，命中第一个条件后停止。

按标签聚合：

```sql
SELECT CASE
         WHEN status = 'paid' THEN 'success'
         WHEN status = 'refunded' THEN 'refund'
         ELSE 'unknown'
       END AS status_label,
       COUNT(*) AS payment_count,
       SUM(amount) AS total_amount
FROM payments
GROUP BY status_label
ORDER BY payment_count DESC;
```

## 5. NULL

- `NULL` 不是 0，也不是空字符串。
- 判断未知值使用 `IS NULL` / `IS NOT NULL`。
- `COUNT(*)` 统计行数。
- `COUNT(column)` 忽略该列的 NULL。
- `SUM(column)` 忽略 NULL；如果整组没有非 NULL 值，结果为 NULL。

### `NOT IN` 的 NULL 风险

```sql
3 NOT IN (1, 2, NULL)
```

等价于：

```text
3 <> 1 AND 3 <> 2 AND 3 <> NULL
= TRUE AND TRUE AND UNKNOWN
= UNKNOWN
```

`WHERE` 只保留 TRUE，因此可能返回空集。找“没有匹配记录”时优先使用 `NOT EXISTS`。

## 6. UNION 与 UNION ALL

- 两个查询列数必须相同，对应位置类型兼容。
- `UNION ALL` 保留所有行。
- `UNION` 去掉完全相同的重复行。
- 结果列名通常由第一个 SELECT 决定。
- ORDER BY 放在整个 UNION 之后。

```sql
SELECT customer_id FROM orders WHERE status = 'paid'
UNION
SELECT customer_id FROM orders WHERE status = 'refunded'
ORDER BY customer_id;
```

## 7. 查询前检查

- [ ] 标量子查询是否只返回一行一列？
- [ ] IN 子查询是否只返回一列？
- [ ] NOT EXISTS 是否包含正确的内外表关联？
- [ ] CASE 条件顺序是否符合业务边界？
- [ ] NULL 是否使用 IS NULL 判断？
- [ ] COUNT 和 SUM 的 NULL 语义是否符合业务要求？
- [ ] UNION 两边列数和类型是否兼容？
- [ ] ORDER BY 是否位于整个 UNION 后？
- [ ] `JOIN`、`FROM`、`GROUP BY` 是否拼写正确？

## 8. 当前进度

- 第 5 周完成 20/25 题。
- 第 5 周首次提交完全正确 15/20。
- 已覆盖标量子查询、IN、NOT EXISTS、CASE、NULL、UNION 和 UNION ALL。
- 下一步完成 5 道混合业务场景题，并为每题说明结果粒度。
