# 第 5 周业务场景：NOT IN、NULL 与 CASE 聚合

## 一、`NOT IN` 遇到 NULL

```sql
3 NOT IN (1, 2, NULL)
```

等价的逻辑判断：

```text
3 <> 1  AND  3 <> 2  AND  3 <> NULL
TRUE    AND  TRUE    AND  UNKNOWN
= UNKNOWN
```

`WHERE` 只保留 TRUE，所以该条件不会选出记录。

因此，找“没有匹配记录”的客户时，优先使用 `NOT EXISTS`。

## 二、数据

### customers

| customer_id | customer |
|---:|---|
| 1 | 小林 |
| 2 | 小周 |
| 3 | 小陈 |
| 4 | 小赵 |
| 5 | 小孙 |

### orders_with_unknown

| order_id | customer_id | status | amount |
|---:|---:|---|---:|
| 301 | 1 | completed | 500 |
| 302 | 2 | cancelled | 100 |
| 303 | 4 | completed | 700 |
| 304 | NULL | completed | 300 |

小陈和小孙没有订单；订单 304 的客户编号未知。

## 本轮 3 题

18. 写出原本想用 `NOT IN` 查找“没有订单客户”的查询。不要运行，先根据订单子查询中的 NULL 说明它实际会返回什么。
19. 改写第 18 题，使用 `NOT EXISTS` 正确找出没有订单的客户。说明应该返回哪些客户，以及为什么 NULL 订单不会误匹配。
20. 将 `orders_with_unknown` 按金额分层并聚合：
   - 金额 `>= 500` 为 `high`；
   - 金额 `>= 200` 且 `< 500` 为 `medium`；
   - 其他金额为 `low`。

   返回金额层级、订单数和总金额，按层级名称升序排列。

## 检查

- [ ] `NOT IN` 的 NULL 风险是否解释为 UNKNOWN？
- [ ] `NOT EXISTS` 是否包含正确的相关条件？
- [ ] CASE 是否从上到下判断？
- [ ] GROUP BY 是否使用同一分层表达式？
