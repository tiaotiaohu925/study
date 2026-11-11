# 第 5 周第 1 天：标量子查询

## 一、什么是标量子查询

标量子查询返回**一行一列**，也就是一个值。它可以放在比较符右侧，为外层查询提供统一比较标准。

### 工作示例

`daily_sales`：

| sale_id | amount |
|---:|---:|
| 201 | 20 |
| 202 | 40 |
| 203 | 60 |
| 204 | 100 |

查询高于全局平均金额的销售：

```sql
SELECT sale_id, amount
FROM daily_sales
WHERE amount > (
  SELECT AVG(amount)
  FROM daily_sales
);
```

子查询计算：

```text
(20 + 40 + 60 + 100) / 4 = 55
```

外层查询再保留 `amount > 55` 的记录，因此返回 sale_id 204、amount 100。

## 二、常见错误

### 子查询返回多行

```sql
WHERE amount > (
  SELECT amount
  FROM daily_sales
);
```

子查询返回 4 个金额，不是单值，因此不能直接放在 `>` 后。

### 空结果

如果子查询没有返回任何行，标量结果可能是 NULL。`amount > NULL` 的结果是 UNKNOWN，不会基于该比较返回行。

### 口径要明确

子查询可以有自己的 `WHERE`：

```sql
WHERE amount > (
  SELECT AVG(amount)
  FROM daily_sales
  WHERE sale_id <= 203
);
```

此时平均值不是全表平均值，而是截止到 sale_id 203 的平均值。比较结论必须说明子查询口径。

## 三、练习数据

### sales

| sale_id | store_id | customer_id | amount |
|---:|---:|---:|---:|
| 101 | 10 | 1 | 100 |
| 102 | 10 | 2 | 50 |
| 103 | 11 | 1 | 70 |
| 104 | 12 | 3 | 200 |

## 本轮 3 题

3. 查询金额高于全表销售平均金额的记录，返回 `sale_id`、`amount`，按金额降序排列。
4. 查询每条销售记录的 `sale_id`、`amount`，以及它与全表平均金额的差额。差额列命名为 `diff_from_avg`，按差额降序排列。
5. 下面的子查询会返回多行，不能直接用于比较：

```sql
WHERE amount > (
  SELECT amount
  FROM sales
  WHERE store_id = 10
);
```

请改成“金额高于 store_id 10 的平均销售金额”的正确写法，并解释为什么要用 `AVG`。
