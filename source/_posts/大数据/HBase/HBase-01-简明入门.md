---
title: HBase-01-简明入门
date: 2018-11-01 20:25:37
tags: HBase
---

这是HBase入门系列的第1篇文章，介绍HBase的数据模型、适用场景、集群关键角色、建表流程以及所涉及的HBase基础概念，本文内容基于HBase 2.0 beta2版本。本文既适用于HBase新手，也适用于已有一定经验的HBase开发人员。

#### 一些常见的HBase新手问题

1. 什么样的数据适合用HBase来存储？
1. 既然HBase也是一个数据库，能否用它将现有系统中昂贵的Oracle替换掉？
1. 存放于HBase中的数据记录，为何不直接存放于HDFS之上？
1. 能否直接使用HBase来存储文件数据？
1. Region(HBase中的数据分片)迁移后，数据是否也会被迁移？
2. 为何基于Spark/Hive分析HBase数据时性能较差？

## 开篇
用惯了Oracle/MySQL的同学们，心目中的数据表，应该是长成excel表格那样，这种表结构规整，每一行都有固定的列构成，因此，非常适合结构化数据的存储。但在NoSQL领域，数据表的模样却往往换成了另外一种”画风”：

![image](https://note.youdao.com/yws/api/personal/file/E8C27275CE094D9284B9DFD0E0402570?method=download&shareKey=156d4c7766cdcce63a506a1f7e55cc0a)

行由看似”杂乱无章”的列组成，行与行之间也无须遵循一致的定义，而这种定义恰好符合半结构化数据或非结构化数据的特点。本文所要讲述的HBase，就属于该派系的一个典型代表。这些”杂乱无章”的列所构成的多行数据，被称之为一个”稀疏矩阵”，而上图中的每一个”黑块块”，在HBase中称之为一个KeyValue。

HBase常被用来存放一些结构简单，但数据量非常大的数据(通常在TB/PB级别)，如历史订单记录，日志数据，监控Metris数据等等，HBase提供了简单的基于Key值的快速查询能力。

## 数据模型
#### RowKey
用来表示唯一一行记录的主键，HBase的数据是按照RowKey的字典顺序进行全局排序的，所有的查询都只能依赖于这一个排序维度。

#### 稀疏矩阵
参考了Bigtable，HBase中一个表的数据是按照稀疏矩阵的方式组织的，”开篇”部分给出了一张关于HBase数据表的抽象图，我们再结合下表来加深大家关于”稀疏矩阵”的印象：

RowKey | Columns
---|---
row 1 | {ID, Name, Phone}
row 2 | {ID, Adress, Title, Email}
row 3 | {ID, Adress, Email}

看的出来：每一行中，列的组成都是灵活的，行与行之间并不需要遵循相同的列定义， 也就是HBase数据表”schema-less“的特点。

#### Region


区别于Cassandra/DynamoDB的"Hash分区"设计，HBase中采用了"Range分区"，将Key的完整区间切割成一个个的"Key Range" ，每一个"Key Range"称之为一个Region。

也可以这么理解：将HBase中拥有数亿行的一个大表，横向切割成一个个"子表"，这一个个"子表"就是Region：

![image](https://note.youdao.com/yws/api/personal/file/71DFFB0974884B2E81815F5E35548BB7?method=download&shareKey=b991797ef7826b9b8f8875d49b9ea0e4)

Region是HBase中负载均衡的基本单元，当一个Region增长到一定大小以后，会自动分裂成两个。

#### Column Family


如果将Region看成是一个表的横向切割，那么，一个Region中的数据列的纵向切割，称之为一个Column Family。每一个列，都必须归属于一个Column Family，这个归属关系是在写数据时指定的，而不是建表时预先定义。

![image](https://note.youdao.com/yws/api/personal/file/4651FB0F2EFC449AB1E2D39BC04B4A10?method=download&shareKey=039e46ccbe497af7d383831c3aff5e66)

#### KeyValue

KeyValue的设计不是源自Bigtable，而是要追溯至论文"The log-structured merge-tree(LSM-Tree)"。每一行中的每一列数据，都被包装成独立的拥有特定结构的KeyValue，KeyValue中包含了丰富的自我描述信息:

Key | ColumnFamily | Qualifier | TimeStamp | Type | Value | Tags
---|---|---|---|---|---|---|

看的出来，KeyValue是支撑"稀疏矩阵"设计的一个关键点：一些Key相同的任意数量的独立KeyValue就可以构成一行数据。但这种设计带来的一个显而易见的缺点：每一个KeyValue所携带的自我描述信息，会带来显著的数据膨胀。

## 应用场景
在介绍完了HBase的数据模型以后，我们可以回答本文一开始的前两个问题：

1. 什么样的数据适合用HBase来存储？
1. 既然HBase也是一个数据库，能否用它将现有系统中昂贵的Oracle替换掉？

HBase的数据模型比较简单，数据按照RowKey排序存放，适合HBase存储的数据，可以简单总结如下：

1. 以实体为中心的数据，实体可以包括但不限于如下几种：描述这些实体的，可以有基础属性信息、实体关系(图数据)、所发生的事件(如交易记录、车辆轨迹点)等等。

- 自然人／账户／手机号／车辆相关数据
- 用户画像数据（含标签类数据）
- 图数据（关系类数据）



2. 以事件为中心的数据

- 监控数据
- 时序数据
- 实时位置类数据
- 消息/日志类数据

上面所描述的这些数据，有的是结构化数据，有的是半结构化或非结构化数据。HBase的“稀疏矩阵”设计，使其应对非结构化数据存储时能够得心应手，但在我们的实际用户场景中，结构化数据存储依然占据了比较重的比例。由于HBase仅提供了基于RowKey的单维度索引能力，在应对一些具体的场景时，依然还需要基于HBase之上构建一些专业的能力，如：



- OpenTSDB 时序数据存储，提供基于Metrics+时间+标签的一些组合维度查询与聚合能力
- GeoMesa 时空数据存储，提供基于时间+空间范围的索引能力
- JanusGraph 图数据存储，提供基于属性、关系的图索引能力



HBase擅长于存储结构简单的海量数据但索引能力有限，而Oracle等传统关系型数据库(RDBMS)能够提供丰富的查询能力，但却疲于应对TB级别的海量数据存储，HBase对传统的RDBMS并不是取代关系，而是一种补充。

## HBase与HDFS
我们都知道HBase的数据是存储于HDFS里面的，相信大家也都有这么的认知：HBase是一个分布式数据库，HDFS是一个分布式文件系统，理解了这一点，我们先来粗略回答本文已开始提出的其中两个问题：

- HBase中的数据为何不直接存放于HDFS之上？

HBase中存储的海量数据记录，通常在几百Bytes到KB级别，如果将这些数据直接存储于HDFS之上，会导致大量的小文件产生，为HDFS的元数据管理节点(NameNode)带来沉重的压力。

- 文件能否直接存储于HBase里面？

如果是几MB的文件，其实也可以直接存储于HBase里面，我们暂且将这类文件称之为小文件，HBase提供了一个名为MOB的特性来应对这类小文件的存储。但如果是更大的文件，强烈不建议用HBase来存储，关于这里更多的原因，希望你在详细读完本系列文章所有内容之后能够自己解答。

## 集群角色
关于集群环境，你可以使用国内外大数据厂商的平台，如Cloudera，Hontonworks以及国内的华为，都发行了自己的企业版大数据平台，另外，华为云、阿里云中也均推出了全托管式的HBase服务。



我们假设集群环境已经Ready了，先来看一下集群中的关键角色：

![image](https://note.youdao.com/yws/api/personal/file/A69A7B717934492AA2461E21B073B226?method=download&shareKey=bfd75444b2c1fabe6a472854055fdc53)

相信大部分人对这些角色都已经有了一定程度的了解，我们快速的介绍一下各个角色在集群中的主要职责：

#### ZooKeeper

在一个拥有多个节点的分布式系统中，假设，只能有一个节点是主节点，如何快速的选举出一个主节点而且让所有的节点都认可这个主节点？这就是HBase集群中存在的一个最基础命题。

利用ZooKeeper就可以非常简单的实现这类"仲裁"需求，ZooKeeper还提供了基础的事件通知机制，所有的数据都以 ZNode的形式存在，它也称得上是一个"微型数据库"。

#### NameNode

HDFS作为一个分布式文件系统，自然需要文件目录树的元数据信息，另外，在HDFS中每一个文件都是按照Block存储的，文件与Block的关联也通过元数据信息来描述。NameNode提供了这些元数据信息的存储。

#### DataNode

HDFS的数据存放节点。

#### RegionServer

HBase的数据服务节点。

#### Master

HBase的管理节点，通常在一个集群中设置一个主Master，一个备Master，主备角色的"仲裁"由ZooKeeper实现。 Master主要职责：

①负责管理所有的RegionServer。

②建表/修改表/删除表等DDL操作请求的服务端执行主体。

③管理所有的数据分片(Region)到RegionServer的分配。

④如果一个RegionServer宕机或进程故障，由Master负责将它原来所负责的Regions转移到其它的RegionServer上继续提供服务。

⑤Master自身也可以作为一个RegionServer提供服务，该能力是可配置的。

## 集群部署建议
如果基于物理机/虚拟机部署，通常建议：



1. RegionServer与DataNode联合部署，RegionServer与DataNode按1:1比例设置。



这种部署的优势在于，RegionServer中的数据文件可以存储一个副本于本机的DataNode节点中，从而在读取时可以利用HDFS中的"短路径读取(Short Circuit)"来绕过网络请求，降低读取时延

2. 管理节点独立于数据节点部署



如果是基于物理机部署，每一台物理机节点上可以设置几个RegionServers/DataNodes来提升资源使用率。



也可以选择基于容器来部署，如在HBaseCon Asia 2017大会知乎的演讲主题中，就提到了知乎基于Kubernetes部署HBase服务的实践。

对于公有云HBase服务而言，为了降低总体拥有成本(TCO)，通常选择"计算与存储物理分离"的方式，从架构上来说，可能导致平均时延略有下降，但可以借助于共享存储底层的IO优化来做一些"弥补"。



HBase集群中的RegionServers可以按逻辑划分为多个Groups，一个表可以与一个指定的Group绑定，可以将RegionServer Group理解成将一个大的集群划分成了多个逻辑子集群，借此可以实现多租户间的隔离，这就是HBase中的RegionServer Group特性。

## 示例数据
以我们日常生活都熟悉的手机通话记录的存储为例，先简单给出示例数据的字段定义：

字段中文名 | 字段定义
---|---
主叫号码 | MSISDN1
被叫号码 | MSISDN2
通话开始时间 | StartTime
通话时长(秒) | Duration

如上定义与电信领域的通话记录字段定义相去甚远，本文力求简洁，仅给出了最简单的示例。如下是"虚构"的样例数据：

MSISDN1 | MSISDN2 | StartTime  | Duration
---|---|---|---
13400006666 | 13500006666 | 20181101 | 666

## 写数据之前的准备工作
#### 1. 写数据之前：创建连接
- Login


在启用了安全特性的前提下，Login阶段是为了完成用户认证(确定用户的合法身份)，这是后续一切安全访问控制的基础。

当前Hadoop/HBase仅支持基于Kerberos的用户认证，ZooKeeper除了Kerberos认证，还能支持简单的用户名/密码认证，但都基于静态的配置，无法动态新增用户。如果要支持其它第三方认证，需要对现有的安全框架做出比较大的改动。



- 创建Connection


Connection可以理解为一个HBase集群连接的抽象，建议使用ConnectionFactory提供的工具方法来创建。因为HBase当前提供了两种连接模式：同步连接，异步连接，这两种连接模式下所创建的Connection也是不同的。我们给出ConnectionFactory中关于获取这两种连接的典型方法定义：

```
CompletableFuture<AsyncConnection> createAsyncConnection(Configuration conf,
                 User user);

Connection createConnection(Configuration conf, ExecutorService pool, User user)
      throws IOException;
```
Connection中主要维护着两类共享的资源：



- 线程池
- Socket连接



这些资源都是在真正使用的时候才会被创建，因此，此时的连接还只是一个"虚拟连接"。

#### 2. 写数据之前：创建数据表
- DDL操作的抽象接口 - Admin


Admin定义了常规的DDL接口，列举几个典型的接口：

```
void createNamespace(final NamespaceDescriptor descriptor) throws IOException;

void createTable(final HTableDescriptor desc, byte[][] splitKeys) throws IOException;

TableName[] listTableNames() throws IOException;
```
- 预设合理的数据分片 - Region



分片数量会给读写吞吐量带来直接的影响，因此，建表时通常建议由用户主动指定划分Region分割点，来设定Region的数量。

HBase中数据是按照RowKey的字典顺序排列的，为了能够划分出合理的Region分割点，需要依据如下几点信息：



1. Key的组成结构
1. Key的数据分布预估，如果不能基于Key的组成结构来预估数据分布的话，可能会导致数据在Region间的分布不均匀
1. 读写并发度需求，依据读写并发度需求，设置合理的Region数量

- 为表定义合理的Schema


既然HBase号称"schema-less"的数据存储系统，那何来的是"schema "？的确，在数据库范式的支持上，HBase非常弱，这里的"schema"，主要指如下一些信息的设置：



1. NameSpace设置
2. Column Family的数量
3. 每一个Column Family中所关联的一些关键配置：

① Compression

HBase当前可以支持Snappy，GZ，LZO，LZ4，Bzip2以及ZSTD压缩算法

② DataBlock Encoding

HBase针对自身的特殊数据模型所做的一种压缩编码

③ BloomFilter

可用来协助快速判断一条记录是否存在

④ TTL

指定数据的过期时间

⑤ StoragePolicy

指定Column Family的存储策略，可选项有："ALL_SSD"，"ONE_SSD"，"HOT"，"WARM"，"COLD"，"LAZY_PERSIST"



HBase中并不需要预先设置Column定义信息，这就是HBase schema less设计的核心。


#### 3. Client发送建表请求到Master
建表的请求是通过RPC的方式由Client发送到Master:



- RPC接口基于Protocol Buffer定义
- 建表相关的描述参数，也由Protocol Buffer进行定义及序列化



Client端侧调用了Master服务的什么接口，参数是什么，这些信息都被通过RPC通信传输到Master侧，Master再依据这些接口\参数描述信息决定要执行的操作。2.0版本中，HBase目前已经支持基于Netty的异步RPC框架。

Master侧接收到Client侧的建表请求以后，一些主要操作包括：



1. 生成每一个Region的描述信息对象HRegionInfo，这些描述信息包括：Region ID, Region名称，Key范围，表名称等信息。
2. 生成每一个Region在HDFS中的文件目录。
3. 将HRegionInfo信息写入到记录元数据的hbase:meta表中。

说明: meta表位于名为"hbase"的namespace中，因此，它的全称为"hbase:meta"。但在本系列文章范畴内，常将其缩写为"meta"。



整个过程中，新表的状态也是记录在hbase:meta表中的，而不用再存储在ZooKeeper中。



如果建表执行了一半，Master进程故障，如何处理？这里是由HBase自身提供的一个名为Procedure(V2)的框架来保障操作的事务性的，备Master接管服务以后，将会继续完成整个建表操作。



一个被创建成功的表，还可以被执行如下操作：



1. Disable  将所有的Region下线，该表暂停读写服务
1. Enable  将一个Disable过的表重新Enable，也就是上线所有的Region来正常提供读写服务
1. Alter  更改表或列族的描述信息

#### 4. Master分配Regions
新创建的所有的Regions，通过AssignmentManager将这些Region按照轮询(Round-Robin)的方式分配到每一个RegionServer中，具体的分配计划是由LoadBalancer来提供的。

AssignmentManager负责所有Regions的分配/迁移操作，Master中有一个定时运行的线程，来检查集群中的Regions在各个RegionServer之间的负载是否是均衡的，如果不均衡，则通过LoadBalancer生成相应的Region迁移计划，HBase中支持多种负载均衡算法，有最简单的仅考虑各RegionServer上的Regions数目的负载均衡算法，有基于迁移代价的负载均衡算法，也有数据本地化率优先的负载均衡算法，因为这一部分已经提供了插件化机制，用户也可以自定义负载均衡算法。

参考链接：http://www.nosqlnotes.com/technotes/hbase/hbase-overview-concepts/