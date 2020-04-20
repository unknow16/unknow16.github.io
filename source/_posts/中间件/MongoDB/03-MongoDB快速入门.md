---
title: 03-MongoDB快速入门
toc: true
date: 2019-08-26 10:30:59
tags:
categories:
---

> 请使用mongodb 3.2或以上版本进行学习，或者直接从3.4开始。另外，百度出来的中文资料，请查看15年及以后的信息，可以少走很多弯路。另外，建议使用linux系统进行学习，方便排错。

MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。

MongoDB 将数据存储为一个文档，文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

## 逻辑结构

MongoDB 的逻辑结构是一种层次结构。

主要由： 文档(document)、集合(collection)、数据库(database)这三部分组成的。

1. MongoDB 的文档（document），相当于关系数据库中的一行记录。 

2. 多个文档组成一个集合（collection），相当于关系数据库的表。 

3. 多个集合（collection），逻辑上组织在一起，就是数据库（database）。 

4. 一个 MongoDB 实例支持多个数据库（database）。 



集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。比如，我们可以将以下不同数据结构的文档插入到集合中：

```
{"site":"www.baidu.com"}
{"site":"www.google.com","name":"Google"}
{"site":"www.yfming.com","name":"yfming","num":5}
```

## 常用命令

mongo 是MongoDB的客户端命令行工具，它其实是一个JavaScritp Shell

```
show dbs    显示所有数据库的列表
db			显示当前数据库对象或集合
use local   连接到一个指定的数据库，如果数据库不存在则自动创建
db.getCollectionNames()  获取当前库中的所有集合名，返回一个数组
```



## 插入与查询文档

以下语句创建spitdb数据库 ，吐槽数据库

```
use spitdb 
```

插入文档的语法格式： 

```
db.集合名称.insert(数据);
```

我们这里可以插入以下测试数据： 

```
db.spit.insert({content:"听说你博客很给力呀",userid:"1011",nickname:"小 雅",visits:NumberInt(902)})
```

查询集合的语法格式： 

```
db.集合名称.find()
```

如果我们要查询spit集合的所有文档，我们输入以下命令:

```
db.spit.find()
```

这里你会发现每条文档会有一个叫_id的字段，这个相当于我们原来关系数据库中表的主 

键，当你在插入文档记录时没有指定该字段，MongoDB会自动创建，其类型是ObjectID 

类型。如果我们在插入文档记录时指定该字段也可以，其类型可以是ObjectID类型，也 

可以是MongoDB支持的任意类型。 

再加些测试数据: 

```
db.spit.insert({_id:"1",content:"我还是没有想明白到底为啥出错",userid:"1012",nickname:"小明",visits:NumberInt(2020)}); 

db.spit.insert({_id:"2",content:"加班到半夜",userid:"1013",nickname:"凯 撒",visits:NumberInt(1023)}); 

db.spit.insert({_id:"3",content:"手机流量超了咋 办？",userid:"1013",nickname:"凯撒",visits:NumberInt(111)}); 

db.spit.insert({_id:"4",content:"坚持就是胜利",userid:"1014",nickname:"诺 诺",visits:NumberInt(1223)});
```

如果我想按一定条件来查询，比如我想查询userid为1013的记录，怎么办？很简单！只要在find()中添加参数即可，参数也是json格式，如下：

```
db.spit.find({userid:'1013'})
```

如果你只需要返回符合条件的第一条数据，我们可以使用findOne命令来实现

```
db.spit.findOne({userid:'1013'})
```

如果你想返回指定条数的记录，可以在find方法后调用limit来返回结果，例如： 

```
db.spit.find().limit(3)
```

## 修改与删除文档 

修改文档的语法结构： 

```
db.集合名称.update(条件,修改后的数据)
```

如果我们想修改_id为1的记录，浏览量为1000，输入以下语句： 

```
db.spit.update({_id:"1"},{visits:NumberInt(1000)})
```

执行后，我们会发现，这条文档除了visits字段其它字段都不见了，为了解决这个问题， 

我们需要使用修改器$set来实现，命令如下： 

```
db.spit.update({_id:"2"},{$set:{visits:NumberInt(2000)}})
```

这样就OK啦。 



删除文档的语法结构： 

```
db.集合名称.remove(条件)
```

以下语句可以将数据全部删除，请慎用 

```
db.spit.remove({})
```

如果删除visits=1000的记录，输入以下语句

```
db.spit.remove({visits:1000})
```

## 统计条数 

统计记录条件使用count()方法。以下语句统计spit集合的记录数

```
db.spit.count()
```



如果按条件统计 ，例如：统计userid为1013的记录条数 

```
db.spit.count({userid:"1013"})
```

## 模糊查询 

MongoDB的模糊查询是通过正则表达式的方式实现的。格式为： 

```
/模糊查询字符串/
```



例如，我要查询吐槽内容包含“流量”的所有文档，代码如下： 

```
db.spit.find({content:/流量/})
```



如果要查询吐槽内容中以“加班”开头的，代码如下：

```
db.spit.find({content:/^加班/})
```

## 大于 小于 不等于

```
db.集合名称.find({ "field" : { $gt: value }}) // 大于: field > value 
db.集合名称.find({ "field" : { $lt: value }}) // 小于: field < value 
db.集合名称.find({ "field" : { $gte: value }}) // 大于等于: field >= value 
db.集合名称.find({ "field" : { $lte: value }}) // 小于等于: field <= value 
db.集合名称.find({ "field" : { $ne: value }}) // 不等于: field != value
```

示例：查询吐槽浏览量大于1000的记录

```
db.spit.find({visits:{$gt:1000}})
```

## 包含与不包含 

包含使用$in操作符。 

示例：查询吐槽集合中userid字段包含1013和1014的文档

```
db.spit.find({userid:{$in:["1013","1014"]}})
```



不包含使用$nin操作符。 

示例：查询吐槽集合中userid字段不包含1013和1014的文档

```
db.spit.find({userid:{$nin:["1013","1014"]}})
```

## 条件连接 

我们如果需要查询同时满足两个以上条件，需要使用$and操作符将条件进行关联。（相 

当于SQL的and） 

格式为： 

```
$and:[ { },{ },{ } ] 
```

示例：查询吐槽集合中visits大于等于1000 并且小于2000的文档 

```
db.spit.find({$and:[ {visits:{$gte:1000}} ,{visits:{$lt:2000} }]})
```

如果两个以上条件之间是或者的关系，我们使用 操作符进行关联，与前面and的使用 

方式相同 

格式为： 

```
$or:[ { },{ },{ } ] 
```

示例：查询吐槽集合中userid为1013，或者浏览量小于2000的文档记录

```
db.spit.find({$or:[ {userid:"1013"} ,{visits:{$lt:2000} }]})
```

## 列值增长 

如果我们想实现对某列值在原有值的基础上进行增加或减少，可以使用$inc运算符来实现

```
db.spit.update({_id:"2"},{$inc:{visits:NumberInt(1)}} )
```



## 参考资料

> - []()
> - []()
