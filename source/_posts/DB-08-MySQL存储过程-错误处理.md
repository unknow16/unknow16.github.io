---
title: DB-08-MySQL存储过程-错误处理
date: 2018-08-27 19:09:53
tags: DB
---

默认情况下，当存储过程运行出错时，过程会立即终止，并打印系统错误消息。

## MySQL错误码
#### 关于错误编号和SQLSTATE码：
每个MySQL错误都有一个唯一的数字错误编号(mysql_error_code)，每个错误又对应一个5字符的SQLSTATE码(ANSI SQL 采用)。

#### SQLSTATE码对应的处理程序
1. SQLWARNING处理程序：以‘01’开头的所有sqlstate码与之对应；
2. NOT FOUND处理程序：以‘02’开头的所有sqlstate码与之对应；
3. SQLEXCEPTION处理程序：不以‘01’或‘02’开头的所有sqlstate码，也就是所有未被SQLWARNING或NOTFOUND捕获的SQLSTATE(常遇到的MySQL错误就是非‘01’\‘02’开头的)

> 注意：‘01’、‘02’开头和‘1’、‘2’开头是有区别的，是不一样的错误sqlsate码。

#### 违反唯一约束

违反唯一约束会报如下错误，下文提到的23000即SQLSTATE码

```
ERROR 1062 (23000): Duplicate entry '2' for key 'PRIMARY'
```
## 定义存储过程的异常处理
也就是定义一个异常处理程序，指定当过程某条语句出错时，相应的采取什么操作
    
```
DECLARE handler_action HANDLER
    FOR condition_value [, condition_value] ...
    statement

handler_action:
    CONTINUE
    | EXIT

condition_value:
    mysql_error_code
    | SQLSTATE [VALUE] sqlstate_value
    | condition_name
    | SQLWARNING
    | NOT FOUND
    | SQLEXCEPTION
```
> declare……handler语句必须出现在变量或条件声明的后面。

当某个错误（condition_value）发生时--->执行指定的语句（statement--记录错误信息），执行完之后再决定如何操作（handler_action）。

1. handler_action

- continue：继续执行当前的程序(接着执行出错的SQL的下一条语句)；
- exit：当前程序终止(退出当前declare所在的begin end)；
- 目前还不支持undo功能。

2. statement

可以是单条语句或复合语句。

3. condition_value

指明handler被何种条件触发；如果条件被触发，却没有handler被声明用于处理该条件，程序的进行将取决于条件类型。

## 单个异常处理


#### 1. 遇到异常继续执行
```
DELIMITER $$
CREATE  PROCEDURE small_mistake1( OUT error VARCHAR(5) )  

BEGIN
    DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' 
    SET error = '23000';    -- 用来记录错误发生时的一些信息

    select error;
    SET error = '00000';
    select error;
    
    INSERT INTO TEAMS VALUES(2,27,'third');  -- 会出错的语句
    SET error = '23001';     

END$$

```
begin end块里，定义declare……handler语句用来捕获错误(待命ing)


select、set、select顺序执行，insert语句出错，SQLSTATE码23000，捕获，进行异常处理(赋值记录)，结束后会继续执行出错的insert语句的下一条语句。


#### 2. 遇到错误即停止存储过程的执行
```
DELIMITER $$
CREATE  PROCEDURE small_mistake2( OUT error VARCHAR(5) )  
BEGIN
    DECLARE EXIT HANDLER FOR SQLSTATE '23000' 
    SET error = '23000';

    select error;
    SET error = '00000';
    select error;
    
    INSERT INTO TEAMS VALUES(2,27,'third');
    SET error = '23001';     
END$$
```
与例1唯一不同的是：handler_action选择的是exit，表明在异常处理结束后不会继续执行错误语句后面的语句，直接退出begin end语句块。

## 多个异常处理程序
可以在一个过程中定义多个异常处理程序，针对不同的错误进行不同的处理。
```
DELIMITER $$
CREATE  PROCEDURE small_mistake3( OUT error VARCHAR(5) )  
BEGIN
    DECLARE CONTINUE HANDLER FOR SQLSTATE '23000'  
    SET error = '23000'; 
    
    DECLARE CONTINUE HANDLER FOR SQLSTATE '21S01' 
    SET error = '21S01';
    
    INSERT INTO TEAMS VALUES(2,27,'third',5);  -- 错误语句   
END$$
```
- 如果SQLSTATE码是23000，异常处理执行SET error = '23000'；
- 如果SQLSTATE码是21S01，异常处理执行SET error = '21S01'；

当然，异常处理，可以是自定义，一般都是上述方式的错误信息记录。

## 异常处理程序也可以使用错误编号
```
DECLARE CONTINUE HANDLER FOR 1062 
        SET error = '23000';     

DECLARE CONTINUE HANDLER FOR 1136 
        SET error = '21S01';
```

## 使用默认异常处理程序
当不想为每个错误都定义一个处理程序时，可以使用3个处理程序

```
DELIMITER $$
CREATE  PROCEDURE small_mistake4( 
OUT error VARCHAR(5))  
BEGIN
    DECLARE CONTINUE HANDLER FOR SQLWARNING,NOT FOUND,SQLEXCEPTION  
    SET error = 'xxxxx';  
    
    INSERT INTO teams VALUES(2,27,'third');   
END$$
```

## 忽略某一条件
如果是在定义处理程序时，想忽略某一个条件


```
e.g：DECLARE CONTINUE HANDLER FOR SQLWARNING BEGIN END;
```

也就是说，当遇到SQLWARNING的问题时，进行的异常处理是begin end块，因为里面什么都没有，就类同于直接忽略。

## 异常处理的命名
为了提高可读性，可以给某个sqlstate代码或mysql错误代码一个名字，并且在后面的异常处理程序中使用这个名字。



```
DECLARE condition_name CONDITION FOR condition_value

condition_value:
    mysql_error_code
   |SQLSTATE [VALUE] sqlstate_value
```


#### 1. 未命名的基本格式：


```
BEGIN
　　DECLARE CONTINUE HANDLER FOR 1051
    -- body of handler
END;
```

 

#### 2. 有命名的基本格式：


```
BEGIN
　　DECLARE no_such_table CONDITION FOR 1051;
　　DECLARE CONTINUE HANDLER FOR no_such_table
    -- body of handler
END;
```

#### 3. 命名示例

```
mysql> DELIMITER $$
mysql> CREATE  PROCEDURE small_mistake5( 
    -> 　　OUT error VARCHAR(5))  
    -> BEGIN
    -> 　　DECLARE non_unique CONDITION FOR SQLSTATE '23000';  
    -> 　　DECLARE CONTINUE HANDLER FOR non_unique
    -> 　　begin 
    -> 　　　　SET error = '23000';
    -> 　　　　select error;
    -> 　　end;
    ->
    -> 　　INSERT INTO TEAMS VALUES(2,27,'third');  #会出错语句  
    -> END$$
mysql> DELIMITER ;
mysql> call small_mistake5(@error);
+-------+
| error |
+-------+
| 23000 |
+-------+
```

## 异常传播
在嵌套块的情况下，内部块中发生异常了，首先由本块的异常处理程序来处理，如果本块没有处理，则由外部块的异常处理程序来处理。
```
mysql> DELIMITER $$
mysql> CREATE  PROCEDURE small_mistake6()  
    -> BEGIN    
    -> 　　DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' 
    -> 　　SET @processed = 100;  
    ->
    -> 　　BEGIN      
    -> 　　　　DECLARE CONTINUE HANDLER FOR SQLSTATE '21000' 
    -> 　　　　SET @processed = 200;    
    -> 　　　　INSERT INTO TEAMS VALUES(2,27,'third');   
    -> 　　　　set @test=123321;  
    ->    END;
    -> END$$
mysql> DELIMITER ;
mysql> call small_mistake6;
mysql> select @processed,@test;
+------------+--------+
| @processed | @test |
+------------+--------+
| 300 | 123321 |
+------------+--------+
```

解析：@processed=100说明内部块里的异常传播到了外部块，交由外部块的异常处理程序进行的处理。

建议：当有多层begin end的时候，每层都应该有自己完善的异常处理，做到：自己的异常，自己这层去处理。


参考： https://www.cnblogs.com/geaozhang/p/6814567.html#mingming