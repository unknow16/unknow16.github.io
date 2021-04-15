---
title: exist和in的使用
toc: true
date: 2021-04-11 11:53:35
tags:
categories:
---

如果查询的两个表大小相当，那么用in和exists差别不大。
如果两个表中一个较小，一个是大表，则子查询表大的用exists，子查询表小的用in。

例如：表A（小表），表B（大表）
```sql
-- 以A表为主表
-- 效率低，用到了A表上cc列的索引
select * from A where cc in (select cc from B) 
-- 效率高，用到了B表上cc列的索引
select * from A where exists(select cc from B where cc=A.cc) 

-- 以B表为主表
-- 效率高，用到了B表上cc列的索引
select * from B where cc in (select cc from A) 
-- 效率低，用到了A表上cc列的索引
select * from B where exists(select cc from A where cc=B.cc)        
```




## 参考资料
> - []()
> - []()
