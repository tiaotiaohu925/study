# 第 5 周第 1 天：CASE 与 NULL

## 一、CASE WHEN

```sql
CASE
  WHEN score >= 90 THEN 'A'
  WHEN score >= 60 THEN 'B'
  ELSE 'C'
END
```

判断从上到下执行，命中第一个条件后不再继续。

## 二、NULL 的两个关键点

- `NULL` 表示未知值，不能用 `= NULL` 判断，必须用 `IS NULL` / `IS NOT NULL`。
- `COUNT(*)` 统计结果行；`COUNT(column)` 不统计该列中的 NULL。
- `SUM(column)` 会忽略 NULL；如果整组没有非 NULL 值，结果为 NULL。

## 练习数据

### payments

| payment_id | customer_id | amount | status |
|---:|---:|---:|---|
| 601 | 1 | 250 | paid |
| 602 | 2 | 120 | paid |
| 603 | 3 | 80 | refunded |
| 604 | 4 | 300 | NULL |
| 605 | 5 | NULL | paid |

## 本轮 3 题

9. 查询每笔支付的 `payment_id`、`amount` 和状态标签。标签列命名为 `status_label`：
   - `paid` 显示 `success`；
   - `refunded` 显示 `refund`；
   - 其他情况（包括 NULL）显示 `unknown`。

10. 在第 9 题逻辑基础上，按 `status_label` 汇总支付笔数和支付金额，返回标签、笔数、总金额；按笔数降序、标签升序排列。

11. 查询以下四个汇总值：
   - `COUNT(*)`
   - `COUNT(status)`
   - `COUNT(amount)`
   - `SUM(amount)`

说明哪两个计数不同、为什么，以及 `SUM(amount)` 为什么不包含 NULL 金额。
