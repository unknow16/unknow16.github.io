---
title: Mybatis返回生成主键
date: 2018-02-26 23:59:23
tags: Mybatis
---

### 非自增：用户自己生成主键，不需返回

### MySQL自增主键返回
* 可以在插入后，通过传入参数对象的id中获取
```
<!-- 
    useGeneratedKeys：默认值false, 让mybatis使用JDBC的getGeneratedKeys方法来取出数据内部生成的主键。
                    像MySQL和SQL Server这样DBMS的自增递增字段
    keyProperty：插入后，将自增主键值放入对应参数实体的该属性指定的字段中
-->
<insert id="insert" useGeneratedKeys="true" keyProperty="id" parameterType="User">
	insert into tb_user(name, age)
	value ( #{name}, #{age})
</insert>
```

### MySQL使用uuid函数生成

```
<insert id="insert" useGeneratedKeys="true" keyProperty="id" parameterType="User">

    <!-- 在执行插入之前，
        通过uuid函数生成主键，存入User参数的id属性中，
        再执行插入操作
    -->
    <selectKey  keyProperty="id" resultType="String" order="BEFORE">
      SELECT uuid() AS id
    </selectKey>

	insert into tb_user(id, name, age)
	value (#{id}, #{name}, #{age})
</insert>
```


### Oracle使用序列

```
<!-- 
    selectKey元素先运行，设置id，然后再执行插入sql语句
    keyProperty: selectKey语句结果应该被设置的目标属性。
    resultType：执行结果的类型。MyBatis通常可以算出来，允许简单类型，包括字符串
    order: 可以被设置成BEFORE、AFTER
        BEFORE: 先执行生成主键，设置keyProperty，然后执行插入语句
        AFTER: 先执行插入语句，再设置属性，如oracle,可在插入语句中嵌入序列调用
    statementType: 支持statement, prepared, callable分别代表PreparedStatement和CallableStatement类型

-->
<insert id="add" parameterType="vo.Category">
    
    <!-- 执行插入数据，先获取序列生成的下一个键，填入参数实体id属性中 -->
    <selectKey resultType="long" order="BEFORE" keyProperty="id">
        SELECT seqitemid.NEXTVAL FROM DUAL
    </selectKey>
    
    insert into category (name_zh, parent_id,
        show_order, delete_status, description
    )
    values (#{nameZh,jdbcType=VARCHAR},
            #{parentId,jdbcType=SMALLINT},
            #{showOrder,jdbcType=SMALLINT},
            #{deleteStatus,jdbcType=BIT},
            #{description,jdbcType=VARCHAR}
    )
</insert>
```

