---
title: DB-06-MySQL存储过程
date: 2018-08-27 19:08:33
tags: DB
---

## 存储过程简介
SQL语句需要先编译然后执行，而存储过程（Stored Procedure）是一组为了完成特定功能的SQL语句集，经编译后存储在数据库中，用户通过指定存储过程的名字并给定参数（如果该存储过程带有参数）来调用执行它。

MySQL 5.0以前并不支持存储过程，这使得MySQL在应用上大打折扣。好在MySQL 5.0开始支持存储过程，这样即可以大大提高数据库的处理速度，同时也可以提高数据库编程的灵活性。

## Example
```
delimiter $
create PROCEDURE phoneDeal()
BEGIN
    DECLARE  id varchar(64);   -- id
    DECLARE  phone1  varchar(16); -- phone
    DECLARE  password1  varchar(32); -- 密码
    DECLARE  name1 varchar(64);   -- id
    -- 遍历数据结束标志
    DECLARE done INT DEFAULT FALSE;
    -- 游标
    DECLARE cur_account CURSOR FOR select phone,password,name from account_temp;
    -- 将结束标志绑定到游标
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    -- 打开游标
    OPEN  cur_account;     
    -- 遍历
    read_loop: LOOP
            -- 取值 取多个字段
            FETCH  NEXT from cur_account INTO phone1,password1,name1;
            IF done THEN
                LEAVE read_loop;
             END IF;
 
        -- 你自己想做的操作
        insert into account(id,phone,password,name) value(UUID(),phone1,password1,CONCAT(name1,'的家长'));
    END LOOP;
 
 
    CLOSE cur_account;
END $
```

## 存储过程创建语法

```
CREATE PROCEDURE  过程名([  [IN|OUT|INOUT]  参数名 数据类型  [,  [IN|OUT|INOUT] 参数名 数据类型…]]) [特性 ...] 过程体
```
#### 参数类型
- IN：参数的值必须在调用存储过程时指定，在存储过程中修改该参数的值不能被返回，为默认值

```
-- 创建in_param存储过程
DELIMITER //  --声明分隔符
	CREATE PROCEDURE in_param(IN p_in int)
		BEGIN
    		SELECT p_in;
    		SET p_in=2;
    		SELECT p_in;
		END;
		// -- 表明存储过程结束
DELIMITER ; -- 还原;为分隔符

-- 调用
SET @p_in = 1;            --设置变量
CALL in_param(@p_in);   --调用存储过程
SELECT @p_in;           --查看变量值并未改变，仍为1

```


- OUT：该值可在存储过程内部被改变，并可返回


```
-- 创建out_param存储过程
delimiter $
	CREATE PROCEDURE out_param(OUT p_out int) 
		BEGIN
			SELECT p_out;
			SET p_out = 2;
			SELECT p_out;
		END;
		$
delimiter ;

-- 调用
SET @p_out = 1;         --设置变量
CALL out_param(@p_out); --调用
SELECT @p_out;          --值为2，经存储过程中改变
```


- INOUT：调用时指定，并且可被改变和返回

```
-- 创建inout_param存储过程
delimiter $
CREATE PROCEDURE inout_param(INOUT p_inout int) 
	BEGIN
		SELECT p_inout;
		SET p_inout = 2;
		SELECT p_inout;
	END;
	$
delimiter ;

-- 调用
SET @p_inout = 1;               --设置变量
CALL inout_param(@p_inout);     --调用
SELECT @p_inout;                --值为2，经存储过程中改变
```

#### 过程体
过程体的开始与结束使用BEGIN与END进行标识。

## 存储过程调用
用call和你过程名以及一个括号，括号里面根据需要，加入参数，参数包括输入参数、输出参数、输入输出参数。

## 存储过程查询

1. 如下都可查询test库中的所有存储过程
```

SELECT name FROM mysql.proc where db = 'test';

SELECT routine_name FROM information_schema.ROUTINES where ROUTINE_SCHEMA = 'test';

SHOW PROCEDURE STATUS WHERE db = 'test';
```
2. 查看存储过程详细信息

SHOW CREATE PROCEDURE 数据库.存储过程名; 如下查看tes库中的in_param

```
SHOW CREATE PROCEDURE test.in_param;
```

## 存储过程修改
ALTER PROCEDURE 更改用CREATE PROCEDURE 建立的预先指定的存储过程，其不会影响相关存储过程或存储功能。

#### 语法
```
ALTER {PROCEDURE | FUNCTION} sp_name [characteristic ...]

characteristic:
{ CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA } -- 要其中一个
| SQL SECURITY { DEFINER | INVOKER } -- definer和invoker要一个
| COMMENT 'string' -- 可选
```
- sp_name参数表示存储过程或函数的名称；
- characteristic参数指定存储函数的特性。
- 
- CONTAINS SQL表示子程序包含SQL语句，但不包含读或写数据的语句；
- NO SQL表示子程序中不包含SQL语句；
- READS SQL DATA表示子程序中包含读数据的语句；
- MODIFIES SQL DATA表示子程序中包含写数据的语句。
- 
- SQL SECURITY { DEFINER | INVOKER }指明谁有权限来执行，DEFINER表示只有定义者自己才能够执行；INVOKER表示调用者可以执行。
- COMMENT 'string'是注释信息。

#### example
1. 将读写权限改为MODIFIES SQL DATA，并指明调用者可以执行。

```
ALTER  PROCEDURE  num_from_employee
  MODIFIES SQL DATA
  SQL SECURITY INVOKER ;
```

2. 将读写权限改为READS SQL DATA，并加上注释信息'FIND NAME'。

```
ALTER  PROCEDURE  name_from_employee
  READS SQL DATA
  COMMENT 'FIND NAME' ;
```

## 存储过程删除

```
DROP PROCEDURE [过程1[,过程2…]]
```

从MySQL的表格中删除一个或多个存储过程。

## 存储过程控制语句
#### 条件语句
1. IF-THEN-ELSE语句



```
#条件语句IF-THEN-ELSE
DROP PROCEDURE IF EXISTS proc3;
DELIMITER //
CREATE PROCEDURE proc3(IN parameter int)
  BEGIN
    DECLARE var int;
    SET var=parameter+1;
    IF var=0 THEN
      INSERT INTO t VALUES (17);
    END IF ;
    IF parameter=0 THEN
      UPDATE t SET s1=s1+1;
    ELSE
      UPDATE t SET s1=s1+2;
    END IF ;
  END ;
  //
DELIMITER ;
```

2. CASE-WHEN-THEN-ELSE语句


```
#CASE-WHEN-THEN-ELSE语句
DELIMITER //
  CREATE PROCEDURE proc4 (IN parameter INT)
    BEGIN
      DECLARE var INT;
      SET var=parameter+1;
      CASE var
        WHEN 0 THEN
          INSERT INTO t VALUES (17);
        WHEN 1 THEN
          INSERT INTO t VALUES (18);
        ELSE
          INSERT INTO t VALUES (19);
      END CASE ;
    END ;
  //
DELIMITER ;
```

#### 循环语句
1. WHILE-DO…END-WHILE

```
DELIMITER //
  CREATE PROCEDURE proc5()
    BEGIN
      DECLARE var INT;
      SET var=0;
      WHILE var<6 DO
        INSERT INTO t VALUES (var);
        SET var=var+1;
      END WHILE ;
    END;
  //
DELIMITER ;
```

 
2. REPEAT...END REPEAT

此语句的特点是执行操作后检查结果

```
DELIMITER //
  CREATE PROCEDURE proc6 ()
    BEGIN
      DECLARE v INT;
      SET v=0;
      REPEAT
        INSERT INTO t VALUES(v);
        SET v=v+1;
        UNTIL v>=5
      END REPEAT;
    END;
  //
DELIMITER ;
```

 
3. LOOP...END LOOP

```
DELIMITER //
  CREATE PROCEDURE proc7 ()
    BEGIN
      DECLARE v INT;
      SET v=0;
      LOOP_LABLE:LOOP
        INSERT INTO t VALUES(v);
        SET v=v+1;
        IF v >=5 THEN
          LEAVE LOOP_LABLE;
        END IF;
      END LOOP;
    END;
  //
DELIMITER ;
```

* LABLES标号

标号可以用在begin repeat while 或者loop 语句前，语句标号只能在合法的语句前面使用。可以跳出循环，使运行指令达到复合语句的最后一步。

4. ITERATE迭代

通过引用复合语句的标号,来从新开始复合语句


```
#ITERATE
DELIMITER //
  CREATE PROCEDURE proc8()
  BEGIN
    DECLARE v INT;
    SET v=0;
    LOOP_LABLE:LOOP
      IF v=3 THEN
        SET v=v+1;
        ITERATE LOOP_LABLE;
      END IF;
      INSERT INTO t VALUES(v);
      SET v=v+1;
      IF v>=5 THEN
        LEAVE LOOP_LABLE;
      END IF;
    END LOOP;
  END;
  //
DELIMITER ;
```
