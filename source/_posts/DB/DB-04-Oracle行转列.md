---
title: DB-04-Oracle行转列
date: 2018-07-18 18:13:59
tags: DB
---

### 1. 表结构

```
CREATE TABLE course_score (
	NAME NVARCHAR2 (2),
	course NVARCHAR2 (2),
	score INT
);
```

### 2. 插入测试数据

```
INSERT into course_score
select N'张三',N'语文',78 from dual union all
select N'张三',N'数学',87 from dual union all
select N'张三',N'英语',82 from dual union all
select N'张三',N'物理',90 from dual union all
select N'李四',N'语文',65 from dual union all
select N'李四',N'数学',77 from dual union all
select N'李四',N'英语',65 from dual union all
select N'李四',N'物理',85 from dual ;
commit;
```

### 3. 行转列前

查询语句：
```
select * from course_score;
```

查询结果：
```
张三	语文	78
张三	数学	87
张三	英语	82
张三	物理	90
李四	语文	65
李四	数学	77
李四	英语	65
李四	物理	85
```

### 4. wm_concat()函数-行转列后
查询语句：
```
select name, wm_concat(score), sum(score) from course_score GROUP BY name;
```

查询结果：
```
张三	78,90,82,87	    337
李四	65,85,65,77	    292
```

### 5. Oracle 11g pivot()函数
查询语句：

```
select * from course_score 
pivot ( MAX(SCORE) FOR COURSE IN ('语文' AS A , '数学' AS B, '英语' AS C,'物理' AS D)  )
```
查询结果：

```
李四	65	77	65	85
张三	78	87	82	90
```

包含总分的查询语句：

```
select t.*, t.a+t.b+t.c+t.d as total 
from (
	select * from course_score 
	pivot ( MAX(SCORE) FOR COURSE IN ('语文' AS A , '数学' AS B, '英语' AS C,'物理' AS D)  )
) t;
```


查询结果：

```
李四	65	77	65	85	292
张三	78	87	82	90	337
```

### 6. 使用decode()函数
查询语句：

```
select name,
	MAX(decode(COURSE, '语文', SCORE)) A,
	MAX(decode(COURSE, '数学', SCORE)) B,
	MAX(decode(COURSE, '英语', SCORE)) C,
	MAX(decode(COURSE, '物理', SCORE)) D,
	SUM(SCORE) TOTAL
from course_score
group BY name;
```
查询结果：

```
李四	65	77	65	85	292
张三	78	87	82	90	337
```



