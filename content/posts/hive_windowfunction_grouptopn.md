---
title: "Hive 窗口函数 分组 TopN"
description: 窗口函数应用
toc: true
authors: [hattrick]
tags: [Hive]
date: 2022-11-04T21:10:31+08:00
lastmod: 2022-11-04T21:28:52+08:00
draft: false
---

## 分组 TopN
### 表结构
|empno|ename|job|managerid|hiredate|salary|bonus|deptno|
|-|-|-|-|-|-|-|-|
|7369|SMITH|CLERK|7902|1980-12-17|800||20|
|7499|ALLEN|SALESMAN|7698|1981-2-20|1600|300|30|
|7521|WARD|SALESMAN|7698|1981-2-22|1250|500|30|
|7566|JONES|MANAGER|7839|1981-4-2|2975||20|
|7654|MARTIN|SALESMAN|7698|1981-9-28|1250|1400|30|
|7698|BLAKE|MANAGER|7839|1981-5-1|2850||30|
|7782|CLARK|MANAGER|7839|1981-6-9|2450||10|
|7788|SCOTT|ANALYST|7566|1987-4-19|3000||20|
|7839|KING|PRESIDENT|""|1981-11-17|5000||10|
|7844|TURNER|SALESMAN|7698|1981-9-8|1500|0|30|
|7876|ADAMS|CLERK|7788|1987-5-23|1100||20|
|7900|JAMES|CLERK|7698|1981-12-3|950||30|
|7902|FORD|ANALYST|7566|1981-12-3|3000||20|
|7934|MILLER|CLERK|7782|1982-1-23|1300||10|

- 如果是全局的 topN，则 order by 可以解决。如果是分组，则可以考虑 row_number。
- 基于 row_number 实现，按照部门分区，每个部门内部按照薪水降序排序
```sql
select empno,
       ename,
       salary,
       deptno,
       row_number() over (partition by deptno order by salary desc) as rn
from tb_emp;
```
|empno|ename|salary|deptno|rn|
|-|-|-|-|-|-|-|-|
|7839|KING|5000|10|1|
|7782|CLARK|2450|10|2|
|7934|MILLER|1300|10|3|
|7788|SCOTT|3000|20|1|
|7902|FORD|3000|20|2|
|7566|JONES|2975|20|3|
|7876|ADAMS|1100|20|4|
|7369|SMITH|800|20|5|
|7698|BLAKE|2850|30|1|
|7499|ALLEN|1600|30|2|
|7844|TURNER|1500|30|3|
|7521|WARD|1250|30|4|
|7654|MARTIN|1250|30|5|
|7900|JAMES|950|30|6|
- 过滤每个部门的薪资最高的前两名
- 为了找出每个部门薪资最高的前两名，只需要筛选 rn 列。
```sql
with t1 as (select empno,
                   ename,
                   salary,
                   deptno,
                   row_number() over (partition by deptno order by salary desc) as rn
            from tb_emp)
select *
from t1
where rn < 3;
```
|empno|ename|salary|deptno|rn|
|-|-|-|-|-|-|-|-|
|7839|KING|5000|10|1|
|7782|CLARK|2450|10|2|
|7788|SCOTT|3000|20|1|
|7902|FORD|3000|20|2|
|7698|BLAKE|2850|30|1|
|7499|ALLEN|1600|30|2|