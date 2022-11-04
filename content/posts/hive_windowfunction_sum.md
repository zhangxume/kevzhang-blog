---
title: "Hive 窗口函数 级联累加求和"
description: 窗口函数应用
toc: true
authors: [hattrick]
tags: [Hive]
date: 2022-11-04T20:33:35+08:00
lastmod: 2022-11-04T21:10:52+08:00
draft: false
---

## 级联累加求和
### 表结构
- 原始表结构
|userid|mth|money|
|-|-|-|
|A|2021-01|5|
|A|2021-01|15|
|B|2021-01|5|
|A|2021-01|8|
|B|2021-01|25|
|A|2021-01|5|
|A|2021-02|4|
|A|2021-02|6|
|B|2021-02|10|
|B|2021-02|5|
|A|2021-03|7|
|B|2021-03|9|
|A|2021-03|11|
|B|2021-03|6|

- 统计得到每个用户每个月的消费总金额
- 根据 userid，mth 分组后对 money 进行求和
|userid|mth|money|
|-|-|-|
|A|2021-01|33|
|A|2021-02|10|
|A|2021-03|18|
|B|2021-01|30|
|B|2021-02|15|
|B|2021-03|15|

### 累计总金额
- 统计每个用户每个月消费金额及累计总金额
- 根据 userid 分区，根据 mth 排序，使用窗口函数 sum 。
```sql
select userid,
       mth,
       m_money,
       sum(m_money) over (partition by userid order by mth) as t_money
from tb_money_mtn;
```
|userid|mth|m_money|t_money|
|-|-|-|-|
|A|2021-01|33|33|
|A|2021-02|10|43|
|A|2021-03|18|61|
|B|2021-01|30|30|
|B|2021-02|15|45|
|B|2021-03|15|60|
- sum 默认计算从第一行到当前行，如果希望控制累计的行范围，使用 rows between 控制。
```sql
select userid,
       mth,
       m_money,
       sum(m_money) over (partition by userid order by mth rows between 1 preceding and 2 following) as t_money
from tb_money_mtn;
```