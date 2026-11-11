# 2026-11-11：第 5 周集中补学

## 基本信息

- 实际学习：120 分钟
- 完成内容：JOIN 复测、标量子查询、IN / NOT EXISTS、CASE / NULL、UNION / UNION ALL
- 完成题量：14 道
- 首次提交完全正确：11/14

## JOIN 错题复测

1. 第一次用 `COUNT(s.store_id)` 统计门店数，没有考虑销售明细会把同一门店展开成多行，East 被错算为 3 家门店。修正为 `COUNT(DISTINCT s.store_id)` 后通过。
2. 区域客户数统计正确，LEFT JOIN、GROUP BY、COUNT DISTINCT 和排序均正确。

复测结论：JOIN 结构明显改善，但仍需警惕一对多关系导致的行数放大。

## 标量子查询

- 第 3–5 题 SQL 全部正确。
- 第 5 题解释不够完整；反馈后明确 `store_id = 10` 的两行金额必须用 `AVG` 汇总为一个值，才能放在 `>` 后比较。
- 能说明标量子查询返回一行一列的统一比较标准。

## IN 与 NOT EXISTS

- 第 6–8 题全部首次正确。
- 理解 `IN` 表示外层值是否属于子查询集合，子查询重复值不会让外层客户重复返回。
- `NOT EXISTS` 能正确关联内外表，并分别完成“没有任何订单”和“没有 refunded 订单”。

## CASE 与 NULL

- 第 9–11 题 SQL 全部正确。
- 能使用简单 CASE 把 paid、refunded 和 NULL 标成不同标签。
- 第 11 题解释最初只说明 COUNT 差异，没有完整说明 SUM 忽略 NULL；最终补充：整组没有非 NULL 金额时 SUM 返回 NULL。
- 结果：success 2 笔/370；refund 1 笔/80；unknown 1 笔/300。

## UNION 与 UNION ALL

- 第 12–14 题全部首次正确。
- `UNION ALL` 返回 4 行：1、2、4、1。
- `UNION` 去重后返回 3 行：1、2、4。
- 能正确保证两边列数一致，并把 ORDER BY 放在整个 UNION 后。

## 口头复盘

学习者总结：

- 技巧：CASE 打标签、NOT EXISTS 判断存在性、标量子查询求均值比较、UNION 合并结果且排序放末尾。
- 易错：COUNT(列) 跳过 NULL，COUNT(*) 数行；多行子查询不能直接使用 `>` / `=`；GROUP BY 要补全字段；UNION 去重，UNION ALL 保留重复。
- 刷题优先级：NOT EXISTS > CASE 聚合 > UNION 区分。

这份口头总结内容完整，逻辑正确。

## 当前进度

- 第 5 周完成 14/25 题。
- 今天首次提交完全正确 11/14。
- 已覆盖标量子查询、IN、NOT EXISTS、CASE、NULL、UNION 和 UNION ALL。
- 仍需 11 道题，优先练标量/IN/EXISTS 的业务场景、CASE 聚合和 UNION 去重。

## 下次任务

第一项任务：用 3 分钟回顾标量、IN、EXISTS 的差别，然后完成 3 道业务场景题；优先练 NOT EXISTS、CASE 聚合和 UNION 去重。
