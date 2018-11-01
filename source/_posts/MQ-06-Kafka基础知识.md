---
title: MQ-06-Kafka基础知识
date: 2018-10-10 17:40:39
tags: MQ
---

## 使用场景
- 分布式消息传递：解耦和生产者和消费者、缓存消息等
- 网站活跃数据跟踪/用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘
- 日志聚合：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。
- 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
- 流式数据处理：比如作为spark streaming和storm的数据源
- 数据存储
- 事件源

## 核心概念
#### Topic（主题）
一个Topic可以认为是一类消息，消费者可以只关注自己需要的Topic中的消息。在物理上不同Topic的消息分开存储,逻辑上一个Topic的消息对使用者透明。

#### Partition（分区）
每个Topics划分为一个或者多个Partition,每个partition在存储层面是一个append log文件。任何发布到此partition的消息都会被直接追加到log文件的尾部，每条消息在文件中的位置称为offset（偏移量），offset为一个long型数字，它是唯一标记一条消息。kafka并没有提供其他额外的索引机制来存储offset，因为在kafka中几乎不允许对消息进行“随机读写”。

一个Topic的多个partitions，被分布在kafka集群中的多个server上，每个server(kafka实例)负责partitions中消息的读写操作，此外kafka还可以配置partitions需要备份的个数(replicas)每个partition将会被备份到多台机器上，以提高可用性。

partitions的设计目的有多个，最根本原因是kafka基于文件存储。通过分区，可以将日志内容分散到多个server上，来避免文件尺寸达到单机磁盘的上限，每个partiton都会被至少一个server(kafka实例)保存，可以将一个topic切分多任意多个partitions，来提高消息保存/消费的效率。此外越多的partitions意味着可以容纳更多的consumer，有效提升并发消费的能力。

#### Partition Leader（分区Leader）
上面提到可以设置每个分区的副本数，用来进行容错，那么基于replicated方案，那么就意味着需要对多个备份进行调度，每个partition都有一个server为"leader"，leader负责所有的读写操作，如果leader失效，那么将会有其他follower来接管(成为新的leader)，follower只是单调的和leader跟进，同步消息即可。由此可见作为leader的server承载了全部的请求压力，因此从集群的整体考虑，有多少个partitions就意味着有多少个"leader"，kafka会将"leader"均衡的分散在每个实例上,来确保整体的性能稳定。

#### Segment（段文件）
如果一个topic的名称为"my_topic"，它有2个partitions，那么日志将会保存在my_topic_0和my_topic_1两个目录中，日志文件中保存了一序列"log entries"(日志条目)，每个log entry格式为"4个字节的数字N表示消息的长度" + "N个字节的消息内容"，每个日志都有一个offset来唯一的标记一条消息，offset的值为8个字节的数字，表示此消息在此partition中所处的起始位置。每个partition在物理存储层面，有多个log file组成(称为segment)。segmentfile的命名为"最小offset".kafka。例如"00000000000.kafka"，其中"最小offset"表示此segment中起始消息的offset。

![image](https://note.youdao.com/yws/api/personal/file/9C9CFDFCEB244597AE019F66F5CB85B6?method=download&shareKey=41ac6ab06df6fbf4cc746fe244f95c44)

其中每个partiton中所持有的segments列表信息会存储在zookeeper中。

当segment文件尺寸达到一定阀值时(可以通过配置文件设定，默认1G)，将会创建一个新的文件，当buffer中消息的条数达到阀值时将会触发日志信息flush到日志文件中，同时如果"距离最近一次flush的时间差"达到阀值时，也会触发flush到日志文件。如果broker失效，极有可能会丢失那些尚未flush到文件的消息。因为server意外实现，仍然会导致log文件格式的破坏(文件尾部)，那么就要求当server启动时需要检测最后一个segment的文件结构是否合法并进行必要的修复。

获取消息时，需要指定offset和最大chunk尺寸，offset用来表示消息的起始位置，chunk size用来表示最大获取消息的总长度(间接的表示消息的条数)。根据offset，可以找到此消息所在segment文件，然后根据segment的最小offset取差值，得到它在file中的相对位置，直接读取输出即可。

日志文件的删除策略非常简单:启动一个后台线程定期扫描log file列表，把保存时间超过阀值的文件直接删除(根据文件的创建时间)。为了避免删除文件时仍然有read操作(consumer消费)，采取copy-on-write方式。

#### Producers（生产者）
producer将会和Topic下所有partition leader保持socket连接，消息由producer直接通过socket发送到broker，中间不会经过任何"路由层"。事实上，消息被路由到哪个partition上，有producer客户端决定。比如可以采用"random"、"key-hash"、"轮询"等。如果一个topic中有多个partitions，那么在producer端实现"消息均衡分发"是必要的。

其中partition leader的位置(host:port)注册在zookeeper中，producer作为zookeeper client，已经注册了watch用来监听partition leader的变更事件。

异步发送：将多条消息暂且在客户端buffer起来，并将他们批量的发送到broker，小数据IO太多，会拖慢整体的网络延迟，批量延迟发送事实上提升了网络效率。不过这也有一定的隐患，比如说当producer失效时，那些尚未发送的消息将会丢失。

#### Consumers（消费者）
Consumer端向broker发送"fetch"请求，并告知其获取消息的offset，此后Consumer将会获得一定条数的消息，Consumer端也可以重置offset来重新消费消息。

在JMS实现中，Topic模型基于push方式，即broker将消息推送给Consumer端。不过在kafka中，采用了pull方式，即Consumer在和broker建立连接之后，主动去pull(或者说fetch)消息，这中模式有些优点，首先Consumer端可以根据自己的消费能力适时的去fetch消息并处理，且可以控制消息消费的进度(offset)，此外，消费者可以良好的控制消息消费的数量,batch fetch。

其他JMS实现，消息消费的位置是有prodiver保留，以便避免重复发送消息或者将没有消费成功的消息重发等，同时还要控制消息的状态。这就要求JMS broker需要太多额外的工作。在kafka中，partition中的消息只有一个consumer在消费，且不存在消息状态的控制，也没有复杂的消息确认机制，可见kafka broker端是相当轻量级的。当消息被Consumer接收之后，Consumer可以在本地保存最后消息的offset，并间歇性的向zookeeper注册offset。由此可见，consumer客户端也很轻量级。

本质上kafka只支持Topic。每个Consumer仅只属于一个Consumer Group，反过来说，每个Group中可以有多个Consumer。发送到Topic的消息，只会被订阅此Topic的每个Group中的一个Consumer消费。即如下：

1. 如果所有的Consumer都具有相同的Group，这种情况和Queue模式很像，消息将会在Consumers之间负载均衡，只有一个消费者能消费该消息。
1. 如果所有的Consumer都具有不同的Group，那这就是"发布-订阅"，消息将会广播给所有的消费者。

在kafka中，一个Partition中的消息只会被Group中的一个Consumer消费，每个Group中Consumer消息消费互相独立，我们可以认为一个Group是一个"订阅"者，一个Topic中的每个Partions，只会被一个"订阅者"中的一个Consumer消费，不过一个Consumer可以消费多个Partitions中的消息。kafka只能保证一个Partition中的消息被某个Consumer消费时，消息是顺序的。事实上，从Topic角度来说，消息仍不是有序的。

kafka的设计原理决定，对于一个Topic，同一个Group中不能有多于Partitions个数的Consumer同时消费，否则将意味着某些consumer将无法得到消息。

## Zookeeper的作用
无论是kafka集群，还是producer和consumer都依赖于zookeeper来保证系统可用性集群保存一些meta信息。kafka集群几乎不需要维护任何consumer和producer状态信息，这些信息有zookeeper保存，因此producer和consumer的客户端实现非常轻量级，它们可以随意离开，而不会对集群造成额外的影响。

1. Producer端使用zookeeper用来"发现"broker列表,以及和Topic下每个partition leader建立socket连接并发送消息.
1. Broker端使用zookeeper用来注册broker信息,已经监测partitionleader存活性.
1. Consumer端使用zookeeper用来注册consumer信息,其中包括consumer消费的partition列表等,同时也用来发现broker列表,并和partition leader建立socket连接,并获取消息.

![image](https://note.youdao.com/yws/api/personal/file/3B396CD3C7B1437E922686DAF17BBE0F?method=download&shareKey=4fb7ea64e68670f7b2a67e38f1f11cc8)

#### 1. Broker node registry:

当一个kafka broker启动后，首先会向zookeeper注册自己的节点信息(临时znode)，同时当broker和zookeeper断开连接时，此znode也会被删除。

格式: /broker/ids/[0...N]   -->host:port，其中[0..N]表示broker id，每个broker的配置文件中都需要指定一个数字类型的id(全局不可重复)，znode的值为此broker的host:port信息。

#### 2. Broker Topic Registry: 
当一个broker启动时,会向zookeeper注册自己持有的topic和partitions信息,仍然是一个临时znode.

格式: /broker/topics/[topic]/[0...N]  其中[0..N]表示partition索引号.

#### 3. Consumer and Consumer group: 

每个consumer客户端被创建时,会向zookeeper注册自己的信息;此作用主要是为了"负载均衡".

#### 4. Consumer id Registry:
每个consumer都有一个唯一的ID(host:uuid,可以通过配置文件指定,也可以由系统生成),此id用来标记消费者信息.

格式:/consumers/[group_id]/ids/[consumer_id] 

仍然是一个临时的znode,此节点的值为{"topic_name":#streams...},即表示此consumer目前所消费的topic + partitions列表.

#### 5. Consumer offset Tracking:
用来跟踪每个consumer目前所消费的partition中最大的offset.

格式:/consumers/[group_id]/offsets/[topic]/[broker_id-partition_id]-->offset_value

此znode为持久节点,可以看出offset跟group_id有关,以表明当group中一个消费者失效,其他consumer可以继续消费.

#### 6. Partition Owner registry:

用来标记partition被哪个consumer消费.临时znode

格式:/consumers/[group_id]/owners/[topic]/[broker_id-partition_id]-->consumer_node_id

#### 当consumer启动时,所触发的操作:
1. 首先进行"Consumer id Registry";
1. 然后在"Consumer id Registry"节点下注册一个watch用来监听当前group中其他consumer的"leave"和"join";只要此znode path下节点列表变更,都会触发此group下consumer的负载均衡.(比如一个consumer失效,那么其他consumer接管partitions).
1. 在"Broker id registry"节点下,注册一个watch用来监听broker的存活情况;如果broker列表变更,将会触发所有的groups下的consumer重新balance.

## Partition的复制备份
kafka将每个partition数据复制到多个server上,任何一个partition有一个leader和多个follower(可以没有);备份的个数可以通过broker配置文件来设定.leader处理所有的read-write请求,follower需要和leader保持同步.Follower和consumer一样,消费消息并保存在本地日志中;leader负责跟踪所有的follower状态,如果follower"落后"太多或者失效,leader将会把它从replicas同步列表中删除.当所有的follower都将一条消息保存成功,此消息才被认为是"committed",那么此时consumer才能消费它.即使只有一个replicas实例存活,仍然可以保证消息的正常发送和接收,只要zookeeper集群存活即可.(不同于其他分布式存储,比如hbase需要"多数派"存活才行)

当leader失效时,需在followers中选取出新的leader,可能此时follower落后于leader,因此需要选择一个"up-to-date"的follower.选择follower时需要兼顾一个问题,就是新leaderserver上所已经承载的partition leader的个数,如果一个server上有过多的partition leader,意味着此server将承受着更多的IO压力.在选举新leader,需要考虑到"负载均衡".

## 持久性
kafka使用文件存储消息,这就直接决定kafka在性能上严重依赖文件系统的本身特性.且无论任何OS下,对文件系统本身的优化几乎没有可能.文件缓存/直接内存映射等是常用的手段.因为kafka是对日志文件进行append操作,因此磁盘检索的开支是较小的;同时为了减少磁盘写入的次数,broker会将消息暂时buffer起来,当消息的个数(或尺寸)达到一定阀值时,再flush到磁盘,这样减少了磁盘IO调用的次数.

## 性能
需要考虑的影响性能点很多,除磁盘IO之外,我们还需要考虑网络IO,这直接关系到kafka的吞吐量问题.kafka并没有提供太多高超的技巧;对于producer端,可以将消息buffer起来,当消息的条数达到一定阀值时,批量发送给broker;对于consumer端也是一样,批量fetch多条消息.不过消息量的大小可以通过配置文件来指定.对于kafka broker端,似乎有个sendfile系统调用可以潜在的提升网络IO的性能:将文件的数据映射到系统内存中,socket直接读取相应的内存区域即可,而无需进程再次copy和交换. 其实对于producer/consumer/broker三者而言,CPU的开支应该都不大,因此启用消息压缩机制是一个良好的策略;压缩需要消耗少量的CPU资源,不过对于kafka而言,网络IO更应该需要考虑.可以将任何在网络上传输的消息都经过压缩.kafka支持gzip/snappy等多种压缩方式.

## 主要配置
#### broker配置

![image](https://note.youdao.com/yws/api/personal/file/2FA8F790C40243E4951986E44A080B2C?method=download&shareKey=62dbb9d7bf2c13bc20384d73d162293c)

#### producer配置

![image](https://note.youdao.com/yws/api/personal/file/86D5934A60E944E69F79BCF593782237?method=download&shareKey=88ef0209603254161c0ed48c3170d336)

#### consumer配置

![image](https://note.youdao.com/yws/api/personal/file/2461E03A1247401AA05A11F02DDCD462?method=download&shareKey=3419b43dda452d7760c0bf14da2129d0)


#### kafka 1.0.0 release in Nov 1,2017

https://www.cnblogs.com/likehua/p/3999538.html