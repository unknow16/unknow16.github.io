---
title: SQL优化
toc: true
date: 2021-03-18 14:37:25
tags:
categories:
---

## 查询SQL尽量不要使用select *，而是select具体字段
- 只取需要的字段，节省资源、减少网络开销。
- select * 进行查询时，很可能就不会使用到覆盖索引了，就会造成回表查询。

## 如果知道查询结果只有一条或者只要最大/最小一条记录，建议用limit 1

```
select id，name from employee where name = 'jay' limit 1;
```

- 加上limit 1后,只要找到了对应的一条记录,就不会继续向下扫描了,效率将会大大提高。
- 当然，如果name是唯一索引的话，是不必要加上limit 1了，因为limit的存在主要就是为了防止全表扫描，从而提高性能
- 如果一个语句本身可以预知不用全表扫描，有没有limit ，性能的差别并不大。
- 用limit分页时，如limit 10000，10，当偏移量最大的时候，查询效率就会越低，因为Mysql并非是跳过偏移量直接去取后面的数据，而是先取偏移量+要取的条数，然后再把前面偏移量这一段的数据抛弃掉再返回的。

## 应尽量避免在where子句中使用or来连接条件
一个user表，它有一个普通索引userId,假设现在需要查询userid为1或者年龄为18岁的用户，很容易有以下SQL

反例: select* from user where userid=1 or age=18
正例：
```
//使用union all
select* from user where userid=1
union all
select* from user where age = 18
 
//或者分开两条sql写：
select* from user where userid=1
select* from user where age = 18
```

理由：使用or可能会使索引失效，从而全表扫描
对于or+没有索引的age这种情况，假设它走了userId的索引，但是走到age查询条件时，它还得全表扫描，也就是需要三步过程：全表扫描+索引扫描+合并
如果它一开始就走全表扫描，直接一遍扫描就完事。mysql是有优化器的，处于效率与成本考虑，遇到or条件，索引可能失效，看起来也合情合理。

## 优化你的like语句
日常开发中，如果用到模糊关键字查询，很容易想到like，但是like很可能让你的索引失效。
遵循左前缀匹配原则，如下userId建有索引
```
-- 不走索引
select userId，name from user where userId like '%123';

-- 走索引
select userId，name from user where userId like '123%';
```

## 尽量避免在索引列上使用mysql的内置函数
业务需求：查询最近七天内登陆过的用户，假设loginTime加了索引
```
-- 不走索引，索引列上使用mysql的内置函数，索引失效
select userId,loginTime from loginuser where  Date_ADD(loginTime,Interval7 DAY) >=now();

-- 走索引
select userId,loginTime from loginuser where  loginTime >= Date_ADD(NOW(),INTERVAL - 7 DAY);
```

## 应尽量避免在where子句中对字段进行加减乘除或!=或<>或not in操作符
这将导致系统放弃使用索引而进行全表扫，假如age有索引
```
反例： select * from user where age-1 = 10；
正例： select * from user where age = 11；

反例：select age,name  from user where age <>18;
正例：可以考虑分开两条sql写
select age,name  from user where age <18;
select age,name  from user where age >18;
```

## where子句中考虑使用默认值代替null
```
反例：is not null 会走全表扫描
select * from user where age is not null;

正例：设置0为默认值
select * from user where age>0;
```
- 如果把null值，换成默认值，很多时候让走索引成为可能，同时，表达意思会相对清晰一点。
- 索引字段上使用is null， is not null，可能导致索引失效，并不是说使用了is null 或者 is not null 就会不走索引了，这个跟mysql版本以及查询成本都有关。
- 优化器有基于规则优化和基于成本优化两种，MySQL采用的是基于成本优化，如果MySQL优化器发现，走索引比不走索引成本还要高，肯定会放弃索引，这些条件 !=, >, is null, is not null经常被认为让索引失效，其实是因为一般情况下，查询的成本高，优化器自动放弃索引的。



## 小表驱动大表
都满足SQL需求的前提下，推荐优先使用Inner join（内连接），如果要使用left join，左边表数据结果尽量小，如果有条件的尽量放到左边处理。
```
反例: select* from tab1 t1 left join tab2 t2  on t1.size = t2.size where t1.id>2;
正例：select* from(select* from tab1 where id >2) t1 left join tab2 t2 on t1.size = t2.size;
```
使用了左连接，左边表数据结果尽量小，条件尽量放到左边处理，意味着返回的行数可能比较少。

## exist&in的合理利用
假设表A表示某企业的员工表，表B表示部门表，查询所有部门的所有员工

很容易有以下SQL: select * from A where deptId in (select deptId from B);
这样写等价于：
1. 先查询部门表B: select deptId from B
2. 再由部门deptId，查询A的员工: select * from A where A.deptId = B.deptId

显然，除了使用in，我们也可以用exists实现一样的查询功能: select * from A where exists (select 1 from B where A.deptId = B.deptId);
因为exists查询的理解就是，先执行主查询，获得数据后，再放到子查询中做条件验证，根据验证结果（true或者false），来决定主查询的数据结果是否得意保留。
那么，这样写就等价于：
1. 先从A表做循环: select * from A
2. 再从B表做循环: select * from B where A.deptId = B.deptId

- 因此，根据小表驱动大表，小的数据集驱动大的数据集原则，我们要选择最外层循环小的，也就是，如果B的数据量小于A，适合使用in，如果B的数据量大于A，即适合选择exist。

## 使用联合索引时，遵循最左匹配原则
- 当我们创建一个联合索引的时候，如(k1,k2,k3)，相当于创建了（k1）、(k1,k2)和(k1,k2,k3)三个索引，这就是最左匹配原则。
- 联合索引不满足最左原则，索引一般会失效，但是这个还跟Mysql优化器有关的。
- 涉及order by时，也循最左匹配原则，如：where k1 = x order by k2, 该条件会走(k1,k2) 索引

## 在适当的时候，使用覆盖索引
覆盖索引能够使得你的SQL语句不需要回表，仅仅访问索引就能够得到所有需要的数据，大大提高了查询效率。
```
反例：like模糊查询，不走索引了
select * from user where userid like '%123%'

正例：id为主键，name为普通索引，即用覆盖索引，虽然like后不符合左前缀原则，也会用
select id,name from user where userid like '%123%';
```

## 如果字段类型是字符串，where时一定用引号括起来，否则索引失效
- 这是因为不加单引号时，是字符串跟数字的比较，它们类型不匹配，MySQL会做隐式的类型转换，把它们转换为浮点数再做比较。
```
反例： select* from user where userid =123;
正例： select* from user where userid ='123';
```

## 索引不宜太多，一般5个以内
- 索引并不是越多越好，索引虽然提高了查询的效率，但是也降低了插入和更新的效率
- insert或update时要维护索引，所以建索引需要慎重考虑，视具体情况来定。
- 一个表的索引数最好不要超过5个，若太多需要考虑一些索引是否没有存在的必要。

## 删除冗余和重复索引
``` 
反例：   KEY `idx_userId`(`userId`)
         KEY `idx_userId_age`(`userId`,`age`)

正例:   删除userId索引，因为组合索引（A，B）相当于创建了（A）和（A，B）索引
        KEY `idx_userId_age`(`userId`,`age`)
```
重复的索引需要维护，并且优化器在优化查询的时候也需要逐个地进行考虑，这会影响性能的。

## 索引不适合建在有大量重复数据的字段上，如性别这类型数据库字段
- 如果索引列有大量重复数据，Mysql查询优化器推算发现不走索引的成本更低，很可能就放弃索引了。
- 所以建了索引不能用到，同时insert或update时还要维护索引，反而降低性能

## 如果插入数据过多，考虑批量插入
批量插入性能好，更加省时间
```
//一次500批量插入，分批进行
insert into user(name, age) values
<foreach collection="list" item="item" index="index" separator=",">
    (#{item.name},#{item.age})
</foreach>
```

## 慎用distinct关键字
- distinct 关键字一般用来过滤重复记录，以返回不重复的记录。在查询一个字段或者很少字段的情况下使用时，给查询带来优化效果。但是在字段很多的时候使用，却会大大降低查询效率。
- 带distinct的语句cpu时间和占用时间都高于不带distinct的语句。因为当查询很多字段时，如果使用distinct，数据库引擎就会对数据进行比较，过滤掉重复数据，然而这个比较、过滤的过程会占用系统资源，cpu时间。

## 如果检索结果中不会有重复的记录，尽量用union all替换 union
- 如果使用union，不管检索结果有没有重复，都会尝试进行合并，然后在输出最终结果前进行排序。
- 如果已知检索结果没有重复记录，使用union all 代替union，这样会提高效率。

## 尽量避免向客户端返回过多数据量。
假设业务需求是，用户请求查看自己最近一年观看过的直播数据。
一定要分页，如果是前端分页，可以先查询前两百条记录，因为一般用户应该也不会往下翻太多页，

## 尽可能使用varchar/nvarchar 代替 char/nchar
- 因为首先变长字段存储空间小，可以节省存储空间。
- 其次对于查询来说，在一个相对较小的字段内搜索，效率更高。

## 为了提高group by 语句的效率，可以在执行到该语句前，把不需要的记录过滤掉
```
反例： select job，avg(salary) from employee  group by job having job ='president' or job = 'managent'
正例： select job，avg（salary） from employee where job ='president' or job = 'managent' group by job；
```

## 参考资料
> - []()
> - []()
