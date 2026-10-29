# SQL 基础查询与汇总笔记

> 整理日期：2026-10-28
>
> 来源：第 3 周第 3 天练习、学习者原答、批改与修订记录。
>
> 说明：本笔记由助手根据学习过程整理。总结本身不等于独立掌握证据；掌握判断仍以学习者独立作答和复评为准。

## 1. SQL 的书写顺序与逻辑执行顺序

常见单表查询的书写顺序：

```sql
SELECT 字段和聚合结果
FROM 表
WHERE 行级过滤
GROUP BY 分组字段
HAVING 分组级过滤
ORDER BY 排序字段
LIMIT 返回行数;
```

逻辑执行顺序：

```text
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

关键结论：

- `WHERE` 在分组和聚合之前过滤明细行。
- `GROUP BY` 把明细行划分为组。
- `HAVING` 在聚合计算之后过滤组。
- `SELECT` 计算最终返回字段，包括聚合结果。
- `ORDER BY` 最后排序结果。

因此 `WHERE` 不能直接过滤 `COUNT(*)`、`SUM(amount)` 这类聚合结果；这类条件要放到 `HAVING`。

## 2. 行级过滤与分组过滤

查询至少有两笔已支付订单的客户：

```sql
SELECT customer_id,
       COUNT(*) AS order_count,
       SUM(amount) AS total_amount
FROM orders
WHERE status = 'paid'
GROUP BY customer_id
HAVING COUNT(*) >= 2
ORDER BY total_amount DESC;
```

- `WHERE status = 'paid'`：先排除非 paid 订单。
- `GROUP BY customer_id`：每个客户形成一个组。
- `HAVING COUNT(*) >= 2`：聚合后排除订单数不足 2 的客户。
- 返回粒度是“每名客户一行”，因此 `SELECT` 要保留 `customer_id`。

## 3. 分组查询必须检查字段一致性

当 `SELECT` 同时包含普通字段和聚合函数时：

- 普通字段通常应出现在 `GROUP BY` 中。
- 聚合函数负责计算每个组的结果。
- 返回结果要保留能够标识分组的字段，否则无法判断每行属于哪个组。

例如按状态汇总：

```sql
SELECT status,
       COUNT(*) AS order_count,
       SUM(amount) AS total_amount,
       ROUND(AVG(amount), 2) AS avg_amount
FROM orders
GROUP BY status
ORDER BY status ASC;
```

## 4. `COUNT(*)` 与 `COUNT(DISTINCT 列)`

- `COUNT(*)`：统计结果行数。
- `COUNT(DISTINCT 列)`：对非 NULL 值去重后计数。

示例数据：

| visit_id | user_id |
|---:|---:|
| 301 | 1 |
| 302 | 1 |
| 303 | 2 |
| 304 | 3 |

```sql
SELECT COUNT(*) AS visit_count,
       COUNT(DISTINCT user_id) AS user_count
FROM visits;
```

结果是 4 次访问、3 名不同用户。

统计客户数时优先按稳定的唯一标识 `customer_id` 去重，不能依赖客户姓名唯一：

```sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM orders
WHERE status = 'paid';
```

## 5. 多字段排序

一个查询只能有一个 `ORDER BY`；多个排序字段放在同一子句中：

```sql
ORDER BY status ASC,
         amount DESC,
         order_id ASC
```

判断方法：

1. 明确第一排序字段。
2. 明确每个字段是升序还是降序。
3. 只在第一字段相同时使用后续字段。

错误写法：

```sql
ORDER BY amount DESC
ORDER BY order_id ASC
```

这不是两个排序层级，而是重复的 `ORDER BY`，会造成语法错误。

## 6. 函数嵌套与括号

嵌套函数要从内到外阅读，并检查每个左括号都有对应右括号：

```sql
ROUND(AVG(amount), 2)
```

含义：

1. `AVG(amount)` 先计算平均值。
2. `ROUND(..., 2)` 再把平均值保留两位小数。

错误写法：

```sql
ROUND(AVG(amount,2)
```

问题：

- `2` 被错误放进了 `AVG` 的参数。
- `ROUND(` 没有闭合。

## 7. 正确示例：工单处理

数据：

| ticket_id | priority | minutes | status |
|---:|---|---:|---|
| 201 | normal | 30 | resolved |
| 202 | urgent | 45 | resolved |
| 203 | urgent | 20 | resolved |
| 204 | low | 10 | open |

查询至少解决两张工单的优先级：

```sql
SELECT priority,
       COUNT(*) AS resolved_count,
       ROUND(AVG(minutes), 2) AS avg_minutes
FROM tickets
WHERE status = 'resolved'
GROUP BY priority
HAVING COUNT(*) >= 2
ORDER BY avg_minutes DESC;
```

结果：

| priority | resolved_count | avg_minutes |
|---|---:|---:|
| urgent | 2 | 32.50 |

业务解释：`urgent` 工单中，已处理数量达到 2 单，平均处理时间为 32.5 分钟。

## 8. 本次学习暴露的具体问题

1. 多个排序字段被写成两个 `ORDER BY`。
2. `ROUND(AVG(amount), 2)` 的括号层级写错。
3. 分组查询漏写 `GROUP BY`，或忘记把分组字段放进 `SELECT`。
4. 已有聚合函数却不必要地按状态分组，导致明细查询错误合并。
5. 统计客户数时曾按客户姓名去重；姓名不是稳定唯一标识。

## 9. 提交查询前的自检清单

- [ ] 是否写全了 `FROM` 和正确的表名？
- [ ] `SELECT` 中的字段是否都能由当前分组粒度解释？
- [ ] 行级条件是否放在 `WHERE`，组级条件是否放在 `HAVING`？
- [ ] 包含普通字段和聚合函数时，是否写了正确的 `GROUP BY`？
- [ ] 是否需要保留分组字段，让结果能说明每组属于谁？
- [ ] 是否只有一个 `ORDER BY`，排序方向和后续字段是否正确？
- [ ] 聚合函数和 `ROUND` 等嵌套函数的括号、逗号是否正确？
- [ ] 统计客户数量时是否优先使用 `customer_id`，而不是姓名？

## 10. 当前掌握状态

截至 2026-10-28：

- 第 3 周完成 15 道新的 SQL 基础题；全天首次提交完全正确 7/15，修正后 15 题均可得到正确结果。
- 已覆盖日期过滤、聚合、分组、`HAVING`、`COUNT DISTINCT`、多字段排序和结果别名。
- SQL 基础笔记交付已完成，但第 3 周仍差 5 道基础题和一次新数据混合复评。
- 当前薄弱点仍是查询结构自检：`FROM`、`GROUP BY`、分组字段、函数括号和排序子句。
- 本笔记由助手整理，不能单独证明以上能力已稳定掌握；后续仍需新数据下的独立验证。

## 11. 第 3 周收口复评

复评日期：2026-10-29

- 新数据第 16–20 题：全部首次正确。
- 混合复评第 21 题：双字段分组和排序正确。
- 业务场景第 22 题：SQL 正确；业务金额首次误判，实际 paid 总金额为 app 200、web 350、mini 360，最高为 mini，反馈后已修正。
- 口头复盘可表达完成内容，但“熟练掌握”的证据不足，下一阶段需继续用新数据验证。
- 第 3 周 20 道基础题、SQL 笔记和混合复评均已完成；进入第 4 周，不意味着所有 SQL 基础能力已经稳定。
