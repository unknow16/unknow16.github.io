---
title: DB-02-Sharding-JDBC简介
date: 2018-07-18 18:13:37
tags: DB
---

### 简介
Sharding-JDBC是当当应用框架ddframe中，从关系型数据库模块dd-rdb中分离出来的数据库水平分片框架，实现透明化数据库分库分表访问。

Sharding-JDBC是继dubbox和elastic-job之后，ddfream系列开源的第3个项目。

Sharding-JDBC直接封装JDBC API，可以理解为增强版的JDBC驱动，旧代码迁移成本几乎为零：

- 可适用于任何基于Java的ORM框架，如JPA、Hibernate、Mybatis、Spring JDBC Template或直接使用JDBC。
- 可基于任何第三方的数据库连接池，如DBCP、C3P0、 BoneCP、Druid等。
- 理论上可支持任意实现JDBC规范的数据库。虽然目前仅支持MySQL，但已有支持Oracle、SQLServer等数据库的计划。

Sharding-JDBC定位为轻量Java框架，使用客户端直连数据库，以jar包形式提供服务，无proxy代理层，无需额外部署，无其他依赖，DBA也无需改变原有的运维方式。

Sharding-JDBC分片策略灵活，可支持等号、between、in等多维度分片，也可支持多分片键。

SQL解析功能完善，支持聚合、分组、排序、limit、or等查询，并支持Binding Table以及笛卡尔积表查询。

### 实现原理
首先Sharding-JDBC是实现了JDBC协议的jar文件。

基于JDBC协议的实现与基于MySQL等数据库协议实现的中间层略有差别。无论使用哪种架构，核心逻辑均极为相似，除了协议实现层不同（JDBC或数据库协议），都会分为分片规则配置、SQL解析、SQL改写、SQL路由、SQL执行以及结果归并等模块。

Sharding-JDBC的整体架构图如下：

![image](https://note.youdao.com/yws/api/personal/file/3D251DAE07784B9C8131F793F9396B39?method=download&shareKey=235e0040e75d10e7ed8ba746ed62254f)

### 分片规则配置
Sharding-JDBC的分片逻辑非常灵活，支持分片策略自定义、复数分片键、多运算符分片等功能。

如：1）根据用户ID分库，然后根据订单ID分表这种分库分表结合的分片策略；2）根据年分库和月份+用户区域ID分表这样的多片键分片。

Sharding-JDBC除了支持等号运算符进行分片，还支持in/between运算符分片，提供了更加强大的分片功能。

Sharding-JDBC提供了spring命名空间用于简化配置，以及规则引擎用于简化策略编写。由于目前刚开源分片核心逻辑，这两个模块暂未开源，待核心稳定后将会开源其他模块。

### JDBC规范重写
Sharding-JDBC对JDBC规范的重写思路是针对DataSource、Connection、Statement、PreparedStatement和ResultSet五个核心接口封装，将多个真实JDBC实现类集合（如：MySQL JDBC实现/DBCP JDBC实现等）纳入Sharding-JDBC实现类管理。

### SQL解析
SQL解析作为分库分表类产品的核心，性能和兼容性是最重要的衡量指标。目前常见的SQL解析器主要有fdb/jsqlparser和Druid。Sharding-JDBC使用Druid作为SQL解析器，经实际测试，Druid解析速度是另外两个解析器的几十倍。

### SQL改写
SQL改写分为两部分，一部分是将分表的逻辑表名称替换为真实表名称。另一部分是根据SQL解析结果替换一些在分片环境中不正确的功能。这里具两个例子：

第1个例子是avg计算。在分片的环境中，以avg1 +avg2+avg3/3计算平均值并不正确，需要改写为（sum1+sum2+sum3）/（count1+count2+ count3）。这就需要将包含avg的SQL改写为sum和count，然后再结果归并时重新计算平均值。

第2个例子是分页。假设每10条数据为一页，取第2页数据。在分片环境下获取limit 10, 10，归并之后再根据排序条件取出前10条数据是不正确的结果。正确的做法是将分条件改写为limit 0, 20，取出所有前2页数据，再结合排序条件算出正确的数据。可以看到越是靠后的Limit分页效率就会越低，也越浪费内存。有很多方法可避免使用limit进行分页，比如构建记录行记录数和行偏移量的二级索引，或使用上次分页数据结尾ID作为下次查询条件的分页方式。

### SQL路由
SQL路由是根据分片规则配置，将SQL定位至真正的数据源。主要分为单表路由、Binding表路由和笛卡尔积路由。

单表路由最为简单，但路由结果不一定落入唯一库（表），因为支持根据between和in这样的操作符进行分片，所以最终结果仍然可能落入多个库（表）。

Binding表可理解为分库分表规则完全一致的主从表。举例说明：订单表和订单详情表都根据订单ID作为分片键，任意时刻分片逻辑均相同。这样的关联查询和单表查询难度和性能相当。

笛卡尔积查询最为复杂，因为无法根据Binding关系定位分片规则的一致性，所以非Binding表的关联查询需要拆解为笛卡尔积组合执行。查询性能较低，而且数据库连接数较高，需谨慎使用。

### SQL执行
路由至真实数据源后，Sharding-JDBC将采用多线程并发执行SQL，并完成对addBatch等批量方法的处理。

### 结果归并
结果归并包括4类：普通遍历类、排序类、聚合类和分组类。每种类型都会先根据分页结果跳过不需要的数据。

普通遍历类最为简单，只需按顺序遍历ResultSet的集合即可。

排序类结果将结果先排序再输出，因为各分片结果均按照各自条件完成排序，所以采用归并排序算法整合最终结果。

聚合类分为3种类型，比较型、累加型和平均值型。比较型包括max和min，只返回最大（小）结果。累加型包括sum和count，需要将结果累加后返回。平均值则是通过SQL改写的sum和count计算，相关内容已在SQL改写涵盖，不再赘述。

分组类最为复杂，需要将所有的ResultSet结果放入内存，使用map-reduce算法分组，最后根据排序和聚合条件做相关处理。最消耗内存，最损失性能的部分即是此，可以考虑使用limit合理的限制分组数据大小。

结果归并部分目前并未采用管道解析的方式，之后会针对这里做更多改进。