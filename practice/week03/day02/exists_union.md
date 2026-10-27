# 第 3 周第 2 天：EXISTS 与 UNION

## 一、EXISTS / NOT EXISTS

`EXISTS` 判断子查询是否至少返回一行，只关心“有没有”，不关心里面具体返回哪个字段。

使用另一组演示数据：

```text
members: member_id | name
         11        | Ada
         12        | Bo
         13        | Cy

logs: log_id | member_id | action
      1      | 11        | login
      2      | 11        | purchase
      3      | 12        | login
```

查询有登录记录的会员：

```sql
SELECT M.name
FROM members AS M
WHERE EXISTS (
  SELECT 1
  FROM logs AS L
  WHERE L.member_id = M.member_id
    AND L.action = 'login'
);
```

结果只有 Ada 和 Bo。Ada 虽然有多条记录，外层结果仍只返回一次。

查询没有登录记录的会员：

```sql
SELECT M.name
FROM members AS M
WHERE NOT EXISTS (
  SELECT 1
  FROM logs AS L
  WHERE L.member_id = M.member_id
    AND L.action = 'login'
);
```

结果是 Cy。

关键点：

- 子查询中的 `M.member_id` 来自外层表，这种写法叫相关子查询。
- 一旦逻辑上能确定存在匹配行，`EXISTS` 就不需要继续证明更多匹配；物理执行可能提前停止，但不要依赖某个数据库的具体执行顺序。
- `SELECT 1` 是常见写法；换成 `SELECT *` 通常不改变结果，可能增加误读。
- `NOT IN` 的子查询如果包含 NULL，可能因为三值逻辑返回意外结果；典型的“不存在匹配记录”需求优先考虑 `NOT EXISTS`。

模板：

```sql
SELECT A.需要的字段
FROM 外表 AS A
WHERE [NOT] EXISTS (
  SELECT 1
  FROM 子表 AS B
  WHERE B.关联字段 = A.关联字段
);
```

## 二、UNION / UNION ALL

`UNION` 和 `UNION ALL` 用于把两个查询结果上下拼接，不是左右连接。

```text
east_team: Ada, Bo
west_team: Bo, Cy
```

```sql
SELECT name FROM east_team
UNION ALL
SELECT name FROM west_team;
```

返回 `Ada, Bo, Bo, Cy`，保留重复的 Bo。

```sql
SELECT name FROM east_team
UNION
SELECT name FROM west_team;
```

返回 `Ada, Bo, Cy`，去掉重复行。

要求与注意点：

- 两个查询的列数必须相同。
- 对应位置的数据类型必须兼容。
- 结果列名通常由第一个 `SELECT` 决定。
- `UNION` 按整行去重，不是只看第一列。
- `UNION ALL` 保留重复行，通常比 `UNION` 少一次去重成本。
- 对最终结果排序时，`ORDER BY` 放在最后一个 `SELECT` 之后。

## 三、练习题

继续使用 `customers` 和 `orders`：

### customers

| customer_id | customer | city |
|---:|---|---|
| 1 | 张三 | 上海 |
| 2 | 李四 | 北京 |
| 3 | 王五 | 上海 |
| 4 | 赵六 | 上海 |
| 5 | 孙七 | 深圳 |
| 6 | 周八 | 北京 |
| 7 | 吴九 | 杭州 |

### orders

| order_id | customer_id | amount | status | order_date |
|---:|---:|---:|---|---|
| 101 | 1 | 120 | paid | 2026-10-01 |
| 102 | 2 | 80 | paid | 2026-10-02 |
| 103 | 3 | 200 | refunded | 2026-10-03 |
| 104 | 4 | 150 | paid | 2026-10-04 |
| 105 | 5 | 90 | paid | 2026-10-05 |
| 106 | 6 | 300 | paid | 2026-10-06 |
| 107 | 1 | 50 | paid | 2026-10-07 |

1. 用 `EXISTS` 查询有过已支付订单的客户姓名和城市。
2. 用 `NOT EXISTS` 查询没有任何退款订单的客户姓名和城市。
3. 分别写出以下两种查询，并说明结果行数：
   - 用 `UNION ALL` 拼接两批 `customer_id`：第一批来自金额 `>= 100` 的订单，第二批来自 `paid` 订单。
   - 把 `UNION ALL` 改成 `UNION`。
4. 下面的语句为什么不可靠？请改成两列都兼容的查询，输出上海客户和北京客户，并保留重复行：

```sql
SELECT customer, '上海客户'
FROM customers
WHERE city = '上海'
UNION ALL
SELECT customer
FROM customers
WHERE city = '北京';
```

5. 查询既有 `paid` 订单、又没有任何 `refunded` 订单的客户姓名和城市。要求同时使用 `EXISTS` 与 `NOT EXISTS`。

请先独立作答，不要运行或核对答案。完成后把 5 条 SQL 和两道说明题的结论发给我。
