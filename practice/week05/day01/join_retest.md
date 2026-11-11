# 第 5 周起点：JOIN 错题复测

## 目标

用新数据检查第 4 周最不稳定的四项：

- `JOIN` / `GROUP BY` 拼写
- 表别名和字段限定
- 分组字段完整性
- `COUNT(DISTINCT ...)` 与 SUM 的零匹配处理

## 数据

### regions

| region_id | region_name |
|---:|---|
| 1 | East |
| 2 | West |
| 3 | North |

### stores

| store_id | region_id | store_name |
|---:|---:|---|
| 10 | 1 | Store A |
| 11 | 1 | Store B |
| 12 | 2 | Store C |
| 13 | 3 | Store D |

### sales

| sale_id | store_id | customer_id | amount |
|---:|---:|---:|---:|
| 101 | 10 | 1 | 100 |
| 102 | 10 | 2 | 50 |
| 103 | 11 | 1 | 70 |
| 104 | 12 | 3 | 200 |

连接链：

```text
regions.region_id → stores.region_id
stores.store_id   → sales.store_id
```

## 复测题

1. 查询每个区域的区域名称、门店数和销售总额。没有销售的区域必须保留，销售总额显示 0；按区域编号升序排列。
2. 查询至少覆盖 2 名不同购买客户的区域，返回区域名称和不同客户数，并按客户数降序、区域名称升序排列。

## 提交前检查

- [ ] `JOIN` 是否拼写正确？
- [ ] 每个字段是否使用正确表别名？
- [ ] `SELECT` 的普通字段是否进入 `GROUP BY`？
- [ ] 统计门店数时是否使用 `COUNT(DISTINCT store_id)`？
- [ ] 无销售区域的总金额是否用 `COALESCE(SUM(...), 0)`？
