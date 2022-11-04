---
title: "Hive 窗口函数 连续登录用户"
description: 窗口函数应用
toc: true
authors: [hattrick]
tags: [Hive]
date: 2022-11-04T17:38:52+08:00
lastmod: 2022-11-04T20:26:52+08:00
draft: false
---

## 连续登录用户
- 根据用户的登录信息，找出连续两天登录的用户。
### 表结构
- 原始表结构
|userid|logintime|
|-|-|
|A|2021-03-22|
|B|2021-03-22|
|C|2021-03-22|
|A|2021-03-23|
|C|2021-03-23|
|A|2021-03-24|
|B|2021-03-24|

### 怎样才算连续两天登录
  - 对用户 ID 进行分区，按照登陆时间进行排序，通过窗口函数 lead 计算出用户下次登录时间。  
    通过日期函数计算出登录以后第二天的日期，如果相等即为连续两天登录。  
    增加一个 `nextday` 字段，表示当前登录日期的第二天。  
    增加一个 `nextlogin` 字段，表示下一次的登录日期。
```sql
select userid,
       logintime,
       --本次登陆日期的第二天
       date_add(logintime, 1)                                              as nextday,
       --按照用户id分区，按照登陆日期排序，取下一次登陆时间，取不到就为0
       lead(logintime, 1, 0) over (partition by userid order by logintime) as nextlogin
from tb_login;
```
#### 连续两天表结构
|userid|logintime|nextday|nextlogin|
|-|-|-|-|
|A|2021-03-22|2021-03-23|2021-03-23|
|A|2021-03-23|2021-03-24|2021-03-24|
|A|2021-03-24|2021-03-25|0|
|B|2021-03-22|2021-03-23|2021-03-24|
|B|2021-03-24|2021-03-25|0|
|C|2021-03-22|2021-03-23|2021-03-23|
|C|2021-03-23|2021-03-24|0|
- 因此最终实现统计连续两天登录用户。
```sql
with t1
         as
         (select userid,
                 logintime,
                 --本次登陆日期的第二天
                 date_add(logintime, 1)                                              as nextday,
                 --按照用户id分区，按照登陆日期排序，取下一次登陆时间，取不到就为0
                 lead(logintime, 1, 0) over (partition by userid order by logintime) as nextlogin
          from tb_login)
select distinct userid
from t1
where nextday = nextlogin;
```
### 连续三天及 N 天
- 如果需要找出连续三天登录的用户，则将`nextday`表示为当前登录日期的第三天，并且使用窗口函数 lead 取两行，判断是否相等。
```sql
select userid,
       logintime,
       --本次登陆日期的第三天
       date_add(logintime, 2)                                              as nextday,
       --按照用户id分区，按照登陆日期排序，取下下一次登陆时间，取不到就为0
       lead(logintime, 2, 0) over (partition by userid order by logintime) as nextlogin
from tb_login;
```
|userid|logintime|nextday|nextlogin|
|-|-|-|-|
|A|2021-03-22|2021-03-24|2021-03-24|
|A|2021-03-23|2021-03-25|0|
|A|2021-03-24|2021-03-26|0|
|B|2021-03-22|2021-03-24|0|
|B|2021-03-24|2021-03-26|0|
|C|2021-03-22|2021-03-24|0|
|C|2021-03-23|2021-03-25|0|
- 上表中，如果 A 用户的 logintime 分别为 22，23，25，则 nextday 第一行为 24，而 nextlogin 为 25，不相等，表明 A 用户没有连续三天登录，与 22，23，25 的登录时间相符（前提是数据经过排序）。
- 因此计算连续 N 天登录（N >= 2）。
```sql
select userid,
       logintime,
       --本次登陆日期的第N天
       date_add(logintime, N - 1)                                              as nextday,
       --按照用户id分区，按照登陆日期排序，取下下一次登陆时间，取不到就为0
       lead(logintime, N - 1, 0) over (partition by userid order by logintime) as nextlogin
from tb_login;
```