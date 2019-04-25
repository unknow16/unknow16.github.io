---
title: HBase-03-Flush与Compaction
date: 2018-11-03 17:31:43
tags: HBase
---

Flush与Compaction其实属于Write流程的继续，所以本文应该称之为"Write后传"。在2.0版本中，最主要的变化就是新增了In-memory Flush/Compaction，而DateTieredCompaction并不算2.0新加入的特性，2.0版本在Compaction核心算法方面并没有什么新的突破。本文将带你探讨Compaction的一些本质问题。

如果看过前两篇文章，至此，数据已经被写入WAL与MemStore，可以说数据已经被成功写到HBase中了。事实上，上篇文章讲到的Flush流程是"简化"后的流程，在2.0版本中，这里已经变的更加复杂。


#### 本文思路

1. 回顾Flush&Compaction旧流程
2. 介绍2.0版本的In-memory Flush & Compaction特性
3. 介绍Compaction要解决的本质问题是什么
4. 在选择合理的Compaction策略之前，用户应该从哪些维度调研自己的业务模型
5. HBase现有的Compaction策略有哪些，各自的适用场景是什么
6. Compaction如何选择待合并的文件
7. 关于Compaction更多的一些思考

## 映射总结
- 一个数据表横向切割分为多个Region
- 一个RegionServer中可有多个Region
- 一个Region纵向切割分为多个Column Family
- 一个Column Family，在Region内部被抽象为了一个HStore对象，而每一个HStore拥有自身的MemStore
- 默认一个RegionServer仅有一个WAL文件，但可给每个Region单独配置一个WAL文件



## Flush&Compaction
在1.X系列版本以及更早版本中的Flush&Compaction行为：
MemStore中的数据，达到一定的阈值，被Flush成HDFS中的HFile文件。

但随着Flush次数的不断增多，HFile的文件数量也会不断增多，那这会带来什么影响？在HBaseCon 2013大会上，Hontonworks的名为《Compaction Improvements in Apache HBase》的演讲主题中，提到了他们测试过的随着HFile的数量的不断增多对读取时延带来的影响：

![image](https://note.youdao.com/yws/api/personal/file/821D90F8A19F40D2BEB43C1FE0C3D04A?method=download&shareKey=5d14b2e17b952ba4c0cbf57d6e599545)

尽管关于Read的流程在后面文章中才会讲到，但下图可以帮助我们简单的理解这其中的原因：

![image](https://note.youdao.com/yws/api/personal/file/0993005E2C7047B1AE78E50E01EF6250?method=download&shareKey=78716734e433458e71eb9b7533fe4db2)

图中说明了从一个文件中指定RowKey为“66660000431^201803011300”的记录，以及从两个文件中读取该行记录的区别，明显，从两个文件中读取，将导致更多的IOPS。这就是HBase Compaction存在的一大初衷，Compaction可以将一些HFile文件合并成较大的HFile文件，也可以把所有的HFile文件合并成一个大的HFile文件，这个过程可以理解为：将多个HFile的“交错无序状态”，变成单个HFile的“有序状态”，降低读取时延。小范围的HFile文件合并，称之为Minor Compaction，一个列族中将所有的HFile文件合并，称之为Major Compaction。除了文件合并范围的不同之外，Major Compaction还会清理一些TTL过期/版本过旧以及被标记删除的数据。下图直观描述了旧版本中的Flush与Compaction流程：

![image](https://note.youdao.com/yws/api/personal/file/E38A63DC9D0F473686DF3438EDD2E88E?method=download&shareKey=35eb66a771553e554479ed1d71012168)

## Flush
在2.0版本中，Flush的行为发生了变化，默认的Flush，仅仅是将正在写的MemStore中的数据归档成一个不可变的Segment，而这个Segment依然处于内存中，这就是2.0的新特性：In-memory Flush and Compaction ，而且该特性在2.0版本中已被默认启用(系统表除外)。该特性因为导致性能明显下降，在正式发布的2.0版本中又被默认禁用了，详情请参考：HBASE-20464。

上文中简单提到了MemStore的核心是一个CellSet，但到了这里，我们可以发现，MemStore的结构其实更加复杂：MemStore由一个可写的Segment，以及一个或多个不可写的Segments构成。

![image](https://note.youdao.com/yws/api/personal/file/18C407CFD04F4BA2A0DE4F18B7D7E324?method=download&shareKey=85948fe69defaf8ebd6f07a319590cb2)

MemStore中的数据先Flush成一个Immutable的Segment，多个Immutable Segments可以在内存中进行Compaction，当达到一定阈值以后才将内存中的数据持久化成HDFS中的HFile文件。

1. 这样做的好处是什么？为何不直接调大MemStore的Flush Size？

如果MemStore中的数据被直接Flush成HFile，而多个HFile又被Compaction合并成了一个大HFile，随着一次次Compaction发生以后，一条数据往往被重写了多次，这带来显著的IO放大问题，另外，频繁的Compaction对IO资源的抢占，其实也是导致HBase查询时延大毛刺的罪魁祸首之一。而In-memory Flush and Compaction特性可以有力改善这一问题。

那为何不干脆调大MemStore的大小？这里的本质原因在于，ConcurrentSkipListMap在存储的数据量达到一定大小以后，写入性能将会出现显著的恶化。

在融入了In-Memory Flush and Compaction特性之后，Flush与Compaction的整体流程演变为：

![image](https://note.youdao.com/yws/api/personal/file/60BAF425B41A4DCD8A96979EFC1D3F59?method=download&shareKey=a7f65551fb1aa017a5093ccd49cae702)

2. 关于Flush执行的策略

一个Region中是否执行Flush，原来的默认行为是通过计算Region中所有Column Family的整体大小，如果超过了一个阈值，则这个Region中所有的Column Family都会被执行Flush。

2.0版本中合入了0.89-fb版本中的一个特性：HBASE-10201/HBASE-3149：Make flush decisions per column family。

2.0版本中默认启用的Flush策略为FlushAllLargeStoresPolicy，也就是说，这个策略使得每一次只Flush超出阈值大小的Column Family，如果都未超出大小，则所有的Column Family都不会被Flush。

改动之前，一个Region中所有的Column Family的Flush都是同步的，虽然容易导致大量的小HFile，但也有好处，尤其是对于WAL文件的快速老化，避免导致过多的WAL文件。而如果这些Column Family的Flush不同步以后，可能会导致过多的WAL文件(过多的WAL文件会触发一些拥有老数据的Column Family执行Flush)，这里需要结合业务场景实测一下。如果每一行数据的写入，多个Column Family的数据都是同步写入的，那么，这个默认策略可能并不合适。

## Compaction
在这个章节，我们继续探讨HBase Compaction，主要是想理清这其中所涉及的一些"道"与"术"：

- "道"：HBase Compaction要解决的本质问题是什么？
- "术"：针对HBase Compaction的问题本质，HBase有哪些具体的改进/优秀实践？


1. Compaction会导致写入放大

我们先来看看Facebook在Fast 14提交的论文《Analysis of HDFS Under HBase: A Facebook Messages Case Study》所提供的一些测试结论（在我之前写的一篇文章《从HBase中移除WAL？3D XPoint技术带来的变革》已经提到过）：

在Facebook Messages业务系统中，业务读写比为99:1，而最终反映到磁盘中，读写比却变为了36:64。WAL，HDFS Replication，Compaction以及Caching，共同导致了磁盘写IO的显著放大。

虽然距离论文发表已经过去几年，但问题本质并未发生明显变化。尤其是在一个写多读少的应用中，因Compaction带来的写放大（Write Amplification）尤其明显，下图有助于你理解写放大的原因：

![image](https://note.youdao.com/yws/api/personal/file/EFC13CAB553A40B482DBBBA101E1DA4D?method=download&shareKey=3841d67ddf946ce2607eb9b753cd88a8)

随着不断的执行Minor Compaction以及Major Compaction，可以看到，这条数据被反复读取/写入了多次，这是导致写放大的一个关键原因，这里的写放大，涉及到网络IO与磁盘IO，因为数据在HDFS中默认有三个副本。

2. Compaction的本质

我们先来思考一下，在集群中执行Compaction，本质是为了什么？容易想到如下两点原因：

- 减少HFile文件数量，减少文件句柄数量，降低读取时延
- Major Compaction可以帮助清理集群中不再需要的数据（过期数据，被标记删除的数据，版本数溢出的数据）

很多HBase用户在集群中关闭了自动Major Compaction，为了降低Compaction对IO资源的抢占，但出于清理数据的需要，又不得不在一些非繁忙时段手动触发Major Compaction，这样既可以有效降低存储空间，也可以有效降低读取时延。

而关于如何合理的执行Compaction，我们需要结合业务数据特点，不断的权衡如下两点：

- 避免因文件数不断增多导致读取时延出现明显增大
- 合理控制写入放大



HFile文件数量一定会加大读取时延吗？也不一定，因为这里与RowKey的分布特点有关。我们通过列举几个典型场景来说明一下不同的RowKey分布，为了方便理解，我们先将一个Region的RowKey Range进一步划分成多个Sub-Range，来说明不同的RowKey分布是如何填充这些Sub-Ranges的：

![image](https://note.youdao.com/yws/api/personal/file/6A3B3B79159546CE9282EEB53AA19975?method=download&shareKey=4f1f62fd5b247b2c75de087ac841af97)

下面的不同行代表不同的时间点（我们使用不断递增的时间点T1,T2,T3,T4，来描述随着时间演变而产生的RowKey分布变化）：

分布A: ![image](https://note.youdao.com/yws/api/personal/file/75448A6D35B84635886316E5C71A6329?method=download&shareKey=d10325c53809995143fbe5984f60fc92)

在这个Region中，某一个时间段只填充与之有关的一个Sub-Range，RowKey随着时间的演进，整体呈现递增趋势。但在填充每一个Sub-Range的时候，又可能有如下两种情形（以Sub-Range1的填充为例，为了区别于T1~T4，我们使用了另外4个时间点Ta~Td）：

分布A-a: ![image](https://note.youdao.com/yws/api/personal/file/4CD3C659EA6641429C64231D02EB28F2?method=download&shareKey=09f330eae0725e32eba3d4fc5351e66d)

这种情形下，RowKey是严格递增的。

分布A-b: ![image](https://note.youdao.com/yws/api/personal/file/5B41A0AEC4834CE289E9F5DAA732752D?method=download&shareKey=b4d84b08339868e46d135e273e929054)

这种情形下，RowKey在Sub-Range1的范围内是完全随机的。

下面则是一种随机RowKey的场景，也就是说，每一个时间点产生的数据都是随机分布在所有的Sub-Range中的：

分布B: ![image](https://note.youdao.com/yws/api/personal/file/AD868301A389405F8F53390336630A54?method=download&shareKey=17ea296875378f826daa38028d1db001)

对于分布A-a来说，不同的HFile文件在RowKey Range（该HFile文件所涉及到的最小数据RowKey与最大数据RowKey构成的RowKey区间）上并不会产生重叠，如果要基于RowKey读取一行数据，只需要查看一个文件即可，而不需要查看所有的文件，这里完全可以通过优化读取逻辑来实现。即使不做Compaction，对于读取时延的影响并不明显（当然，从降低文件句柄数量，降低HDFS侧的小文件数量的维度来考虑，Compaction还是有意义的）。



对于分布B来说，如果有多个HFiles文件，如果想基于RowKey读取一行数据，则需要查看多个文件，因为不同的HFile文件的RowKey Range可能是重叠的，此时，Compaction对于降低读取时延是非常必要的。

## 调研自己的业务模型


在选择一个合理的Compaction策略之前，应该首先调研自己的业务模型，下面是一些参考维度：

1. 写入数据类型／单条记录大小: 是否是KB甚至小于KB级别的小记录？还是MB甚至更大的图片/小文件数据？
2. 业务读写比例
3. 随着时间的不断推移，RowKey的数据分布呈现什么特点？
4. 数据在读写上是否有冷热特点: 是否只读取/改写最近产生的数据？
5. 是否有频繁的更新与删除?
6. 数据是否有TTL限制?
7. 是否有较长时间段的业务高峰期和业务低谷期？

HBase中有几种典型的Compaction策略，来应对几类典型的业务场景：

## Stripe Compaction


它的设计初衷是，Major Compaction占用大量的IO资源，所以很多HBase用户关闭了自动触发的Major Compaction，改为手动触发，因为Major Compaction依然会被用来清理一些不再需要的数据。

随着时间的推移，Major Compaction要合并的文件总Size越来越大，但事实上，真的有必要每一次都将所有的文件合并成一个大的HFile文件吗？尤其是，不断的将一些较老的数据和最新的数据合并在一起，对于一些业务场景而言根本就是不必要的。

因此，它的设计思路为：

将一个Region划分为多个Stripes(可以理解为Sub-Regions），Compaction可以控制在Stripe(Sub-Region)层面发生，而不是整个Region级别，这样可以有效降低Compaction对IO资源的占用。

那为何不直接通过设置更多的Region数量来解决这个问题？更多的Region意味着会加大HBase集群的负担，尤其是加重Region Assignment流程的负担，另外，Region增多，MemStore占用的总体内存变大，而在实际内存无法变大的情况下，只会使得Flush更早被触发，Flush的质量变差。

新Flush产生的HFile文件，先放到一个称之为L0的区域，L0中Key Range是Region的完整Key Range，当对L0中的文件执行Compaction时，再将Compaction的结果输出到对应的Stripe中：

![image](https://note.youdao.com/yws/api/personal/file/2A4657967D2248889265C2D2C909FB41?method=download&shareKey=fcc186dcac00063b9f17d8cc2f179569)

HBase Document中这么描述Stripe Compaction的适用场景：

- Large regions. You can get the positive effects of smaller regions without additional overhead for MemStore and region management overhead.
- Non-uniform keys, such as time dimension in a key. Only the stripes receiving the new keys will need to compact. Old data will not compact as often, if at all

在我们上面列举的几种RowKey分布的场景中，分布A(含分布A-a，分布A-b)就是特别适合Stripe Compaction的场景，因为仅仅新写入数据的Sub-Range合并即可，而对于老的Sub-Range中所关联的数据文件，根本没有必要再执行Compaction。

Stripe Compaction关于Sub-Region的划分，其实可以加速Region Split操作，因为有的情形下，直接将所有的Stripes分成两部分即可。

## Date Tiered Compaction


我们假设有这样一种场景：

新数据的产生与时间有关，而且无更新、删除场景

读取时通常会指定时间范围，而且通常读取最近的数据



在这种情形下，如果将老数据与新数据合并在一起，那么，指定时间范围读取时，就需要扫描一些不必要的老数据：因为合并后，数据按RowKey排序，RowKey排序未必与按照数据产生的时间排序一致，这使得新老数据交叉存放，而扫描时老数据也会被读到。

这是Date Tiered Compaction的设计初衷，Date Tiered Compaction在选择文件执行合并的时候，会感知Date信息，使得Compaction时，不需要将新老数据合并在一起。这对于基于Time Range的Scan操作是非常有利的，因为与本次Scan不相关的文件可以直接忽略。

容易想到，时序数据就是最典型的一个适用场景。但需要注意的是，如下场景并不适合使用Date Tiered Compaction:

- 读取时通常不指定时间范围
- 涉及频繁的更新与删除
- 写入时主动指定时间戳，而且可能会指定一个未来的时间戳
- 基于bulk load加载数据，而且加载的数据可能会在时间范围上重叠


1. 什么是Time Range Based Scan?

HBase的Scan查询，通常都是指定RowKey的Range的，但HBase也支持这样一类查询：通过指定一个起始的Timestamp，扫描出所有的落在此Timestamp Range中的所有数据，这就是Time Range Based Scan。

可以参考接口：Scan#setTimeRange(long minStamp, long maxStamp)

Time Range Based Scan可以用来实现针对HBase数据表的增量数据导出/备份能力。

## MOB Compaction


能否使用HBase来存储MB级别的Blob(如图片之类的小文件)数据？

这是很多应用面临的一个基础问题，因为这些数据相比于普通的存储于HBase中的结构化/半结构化数据显得过大了，而如果将这些数据直接存储成HDFS中的独立文件，会加重HDFS的NameNode的负担，再者，如何索引这些小文件也是一个极大的痛点。



当然，也有人采用了这样的方式：将多个小文件合并成HDFS上的一个大文件，这样子可以减轻HDFS的NameNode的负担，但需要维护每一个小文件的索引信息（文件名以及每一个小文件的偏移信息）。



如果存这些这些小文件时，像普通的结构化数据/半结构化数据一样，直接写到HBase中，会有什么问题？这样子多条数据可以被合并在较大的HFile文件中，减轻了NameNode的负担，同时解决了快速索引的问题。但基于前面的内容，我们已经清楚知道了Compaction带来的写放大问题。试想一下，数MB级别的Blob数据，被反复多次合并以后，会带来什么样的影响？这对IO资源的抢占将会更加严重。



因此，HBase的MOB特性的设计思想为：将Blob数据与描述Blob的元数据分离存储，Blob元数据采用正常的HBase的数据存储方式，而Blob数据存储在额外的MOB文件中，但在Blob元数据行中，存储了这个MOB文件的路径信息。MOB文件本质还是一个HFile文件，但这种HFile文件不参与HBase正常的Compaction流程。仅仅合并Blob元数据信息，写IO放大的问题就得到了有效的缓解。



MOB Compaction也主要是针对MOB特性而存在的，这里涉及到数据在MOB文件与普通的HFile文件之间的一些流动，尤其是MOB的阈值大小发生变更的时候(即当一个列超过预设的配置值时，才被认定为MOB)，本文暂不展开过多的细节。



在HBase社区中，MOB特性(HBASE-11339)一直在一个独立的特性分支中开发的，直到2.0版本才最终合入进来（华为的FusionInsight的HBase版本中，以及华为云的CloudTable的HBase版本中，都包含了完整的MOB特性）。

## Default Compaction


就是默认的Compaction行为，2.0版本和旧版本中的行为没有什么明显变化。

所谓的Default Compaction，具有更广泛的适用场景，它在选择待合并的文件时是在整个Region级别进行选择的，所以往往意味着更高的写IO放大。



在实际应用中，应该结合自己的应用场景选择合适的Compaction策略，如果前几种策略能够匹配自己的应用场景，那么应该是优选的（这几个策略的质量状态如何尚不好判断，建议结合业务场景进行实测观察），否则应该选择Default Compaction。

如果上述几种Compaction策略都无法很好的满足业务需求的话，用户还可以自定义Compaction策略，因为HBase已经具备良好的Compaction插件化机制。

## 如何选择待合并的文件
无论哪种Compaction策略，都涉及一个至关重要的问题：“如何选择待合并的文件列表”

Major Compaction是为了合并所有的文件，所以，不存在如何选择文件的问题。

选择文件时，应该考虑如下几个原则：

1. 选择合理的文件数量

如果一次选择了过多的文件： 对于读取时延的影响时间范围可能比较长，但Compaction产生的写IO总量较低。

如果一次选择了较少的文件： 可能导致过于频繁的合并，导致写IO被严重放大。

2. 选择的文件在时间产生顺序上应该是连续的，即应该遵循HBase的Sequence ID的顺序

RatioBasedCompactionPolicy曾一度作为主力文件选择算法沿用了较长的时间，后来，出现了一种ExploringCompactionPolicy，它的设计初衷为：

RatioBasedCompactionPolicy虽然选择出来了一种文件组合，但其实这个文件组合并不是最优的，因此它期望在所有的候选组合中，选择一组性价比更高的组合，性价比更高的定义为：文件数相对较多，而整体大小却较小。这样，即可以有效降低HFiles数量，又可能有效控制Compaction所占用的IO总量。

也可以这么理解它们之间的一些不同，RatioBasedCompactionPolicy仅仅是在自定义的规则之下找到第一个"可行解"即可，而ExploringCompactionPolicy却在尽可能的去寻求一种自定义评价标准中的"最优解"。

另外，需要说明的一点：ExploringCompactionPolicy在选择候选组合时，正是采用了RatioBasedCompactionPolicy中的文件选择算法。

## 更多的一些思考


Compaction会导致写入放大，前面的内容中已经反复提及了很多次。在实际应用中，你是否关注过Compaction对于查询毛刺的影响（查询时延总是会出现偶发性的陡增）？



关于Compaction的参数调优，我们可能看到过这样的一些建议：尽可能的减少每一次Compaction的文件数量，目的是为了减短每一次Compaction的执行时间。这就好比，采用Java GC算法中的CMS算法时，每一次回收少量的无引用对象，尽管GC被触发的频次会增大，但可以有效降低Full GC的发生次数和发生时间。



但在实践中，这可能并不是一个合理的建议，例如，HBase默认的触发Minor Compaction的最小文件数量为3，但事实上，对于大多数场景而言，这可能是一个非常不合理的默认值，在我们的测试中，将最小文件数加大到10个，我们发现对于整体的吞吐量以及查询毛刺，都有极大的改进，所以，这里的建议为：Minor Compaction的文件数量应该要结合实际业务场景设置合理的值。另外，在实践中，合理的限制Compaction资源的占用也是非常关键的，如Compaction的并发执行度，以及Compaction的吞吐量以及网络带宽占用等等。



另外，需要关注到的一点：Compaction会影响Block Cache，因为HFile文件发生合并以后，旧HFile文件所关联的被Cache的Block将会失效。这也会影响到读取时延。HBase社区有一个问题单(HBASE-20045)，试图在Compaction时缓存一些最近的Blocks。

在Facebook的那篇论文中，还有一个比较有意思的实践：

![image](https://note.youdao.com/yws/api/personal/file/7039B195BB7F4D4FB62437E896E62111?method=download&shareKey=4a58720bafd5ccd6ffdccce5875360b0)

他们将Compaction下推到存储层(HDFS)执行，这样，每一个DateNode在本地合并自己的文件，这样可以降低一半以上的网络IO请求，但本地磁盘IO请求会增大，这事实上是用磁盘IO资源来换取昂贵的网络IO资源。在我们自己的测试中也发现，将Compaction下推到HDFS侧执行，能够明显的优化读写时延毛刺问题。

## 总结
本文基于2.0版本阐述了Flush与Compaction流程，讲述了Compaction所面临的本质问题，介绍了HBase现有的几种Compaction策略以及各自的适用场景，更多是从原理层面展开的，并没有过多讲述如何调整参数的实际建议，唯一的建议为：请一定要结合实际的业务场景，选择合理的Compaction策略，通过不断的测试和观察，选择合理的配置，何谓合理？可以观察如下几点：

1. 写入吞吐量能否满足要求。随着时间的推移，写入吞吐量是否会不断降低？
1. 读取时延能否满足要求。随着时间的推移，读取时延是否出现明显的增大？
1. 观察过程中，建议不断的统计分析Compaction产生的IO总量，以及随着时间的变化趋势。2.0版本中尽管增加了一些与Compaction相关的Metrics信息，但关于Compaction IO总量的统计依然是非常不充分的，这一点可以自己定制实现，如果你有兴趣，也完全可以贡献给社区。 