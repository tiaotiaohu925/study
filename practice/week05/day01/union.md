# 第 5 周第 1 天：UNION 与 UNION ALL

## 核心规则

- `UNION ALL`：上下拼接结果，保留所有重复行。
- `UNION`：上下拼接结果，去掉完全相同的重复行。
- 两个查询列数必须相同，对应位置的数据类型要兼容。
- 结果列名通常取自第一个 `SELECT`。
- `ORDER BY` 只能放在整个 UNION 的最后。

示例：

```text
A：1, 2
B：2, 3

UNION ALL：1, 2, 2, 3
UNION：1, 2, 3
```

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

12. 用 `UNION ALL` 拼接两组 `customer_id`：
   - paid 订单的客户编号；
   - refunded 订单的客户编号。

   返回拼接后的客户编号，并说明共有几行。

13. 使用同一组两个查询，把 `UNION ALL` 改为 `UNION`。返回结果并说明共有几行。

14. 分别连接 `customers` 和 `orders`，查询 paid 客户姓名，以及 refunded 客户姓名，再用 `UNION` 合并并去重。结果按客户编号升序排列。

## 提交前检查

- [ ] 两个查询的列数是否一致？
- [ ] 对应列是否都是客户编号或客户姓名？
- [ ] 是否选择正确的 `UNION` / `UNION ALL`？
- [ ] 去重是否发生在完整结果行上？
- [ ] `ORDER BY` 是否只在 Union 最后出现？
