# 第 5 周业务场景：存在性、CASE 聚合与 UNION

## 本轮目标

- 用 `EXISTS` / `NOT EXISTS` 表达业务筛查条件。
- 用 CASE 条件聚合生成多状态客户摘要。
- 用 UNION 合并两个业务来源并去重。
- 每道题说明结果粒度和业务含义。

## 数据

### customers

| customer_id | customer | city |
|---:|---|---|
| 1 | 小林 | 上海 |
| 2 | 小周 | 北京 |
| 3 | 小陈 | 上海 |
| 4 | 小赵 | 上海 |
| 5 | 小孙 | 深圳 |

### orders

| order_id | customer_id | status | amount |
|---:|---:|---|---:|
| 201 | 1 | completed | 500 |
| 202 | 1 | cancelled | 100 |
| 203 | 2 | completed | 300 |
| 204 | 3 | cancelled | 200 |
| 205 | 4 | completed | 700 |

## 本轮 3 题

15. 查询“有过 completed 订单，但没有任何 cancelled 订单”的客户姓名和城市。同时使用 `EXISTS` 与 `NOT EXISTS`。

16. 生成所有客户的成交摘要：
   - `completed_count`：completed 订单数；
   - `cancelled_amount`：cancelled 订单金额；
   - 没有对应订单时显示 0；
   - 返回客户姓名、两个指标，并按客户编号升序排列。

17. 生成客户触达名单，用 `UNION` 合并并去重：
   - 有 completed 订单的客户姓名；
   - 所在城市为上海的客户姓名。

   结果按客户编号升序排列，并说明最终一行代表什么。

## 提示

- 第 15 题的两个子查询都要和外表客户编号关联。
- 第 16 题从 `customers` 开始 LEFT JOIN，CASE 只累计对应状态。
- 第 17 题两个 SELECT 都要返回客户编号和姓名，列职责一致。
