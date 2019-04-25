---
title: Zookeeper-07-事务日志和快照文件
date: 2018-06-13 15:24:39
tags: Zookeeper
---

## 前言
在zookeeper的zoo.cfg配置文件中，通常有一个必须配置项dataDir。该选项一般用来存储事务日志文件和快照文件。另外zk也提供了一个单独的选项dataLogDir来配置事务日志存储目录。这样的话，dataDir就是指快照文件数据的存放目录了。我的zoo.cfg配置如下：

```
dataDir = zk安装目录/data/
dataLogDir = zk安装目录/data/log/
```
dataDir是必须配置项，dataLogDir是非必须配置，如果dataLogDir没有配置则使用dataDir配置的目录存放事务日志。

## 事务日志文件
在Zab协议中我们知道每次接收到客户端的请求后，Leader和Follwer都会创建一个事务，关联一个事务id即zxid，并存进事务日志文件中。

写事务日志文件和快照文件操作源码在zookeeper-3.4.6.jar的org.apache.zookeeper.server.persistence下，从其中我们能得到下列信息：

1. 会在dataDir和dataLogDir下创建version-2目录存放具体文件
2. 文件名为log，后缀为十六进制的zxid,该zxid为log文件中第一条事务的zxid。
3. zxid规则为前32字节为Leader周期，后32字节为事务请求序列，所以通过事务日志就可以轻松的知道当前的Leader周期与每个文件所属的Leader周期.

#### 事务日志二进制格式

在org.apache.zookeeper.server.persistence.FileTxnLog类注释上描述了事务日志二进制格式，如下分为3大部分：

1. 16bytes的FileHeader
2. TxnList : 事务日志列表
3. ZeroPad ：0分隔符
```
 * LogFile:
 *     FileHeader TxnList ZeroPad
 * 
 * FileHeader: {
 *     magic 4bytes (ZKLG)
 *     version 4bytes
 *     dbid 8bytes
 *   }
 * 
 * TxnList:
 *     Txn || Txn TxnList
 *     
 * Txn:
 *     checksum Txnlen TxnHeader Record 0x42
 * 
 * checksum: 8bytes Adler32 is currently used
 *   calculated across payload -- Txnlen, TxnHeader, Record and 0x42
 * 
 * Txnlen:
 *     len 4bytes
 * 
 * TxnHeader: {
 *     sessionid 8bytes
 *     cxid 4bytes
 *     zxid 8bytes
 *     time 8bytes
 *     type 4bytes
 *   }
 *     
 * Record:
 *     See Jute definition file for details on the various record types
 *      
 * ZeroPad:
 *     0 padded to EOF (filled during preallocation stage)

```


#### 事务日志可视化
事务日志文件都是二进制文件，不能直接查看，zk提供了一个工具类，用来查看这些事务日志文件：org.apache.zookeeper.server.LogFormatter

```
java -classpath .;./zookeeper-3.4.6.jar;./lib/slf4j-api-1.6.1.jar org.apache.zookeeper.server.LogFormatter ./data/log/version-2/log.1

// -classpath 可简写为 -cp
```

#### 事务日志实例解析

![image](https://note.youdao.com/yws/api/personal/file/49975F5B492848F6A058E0EECDE933EF?method=download&shareKey=4835af2bd2c8e00cbc6502904a3e0f3b)

* 第一行：ZooKeeper Transactional Log File with dbid 0 txnlog format version 2

上面的代码分析中有说到每个日志文件都有一个这就是那里所说的日志头，这里magic没有输出，只输出了dbid还有version；

* 第二行：18-6-11 下午03时59分53秒 session 0x14f20ea71c10000 cxid 0x0 zxid 0x1 createSession 4000

这也就是具体的事务日志内容了，这里是说xxx时间有一个sessionid为0x14f20ea71c10000、cxid为0x0、zxid为0x1、类型为createSession、超时时间为4000毫秒　　

* 第三行：同样创建一个会话
* 第四行：18-6-11 下午03时59分54秒 session 0x14f20ea71c10000 cxid 0x1 zxid 0x2 create ‘/solinx0000000000,#736f6c696e78,v{s{31,s{‘world,’anyone}}},F,1

sessionID为0x14f20ea71c10000，cxid：0x01、zxid：0x02、创建了一个节点路径为：/solinx0000000000、节点内容为：#736f6c696e78(经过ASCII，实际内容为solinx)、acl为world:anyone任何人都可以管理该节点、节点不是ephemeral节点的、父节点子版本：1

* 第五行：18-6-11 下午04时15分56秒 session 0x14f20ea71c10000 cxid 0x0 zxid 0x3 closeSession null

这里是说xxx时间有一个sessionid为0x14f20ea71c10000、cxid为0x0、zxid为0x3、类型为closeSession

## 快照文件
快照文件的处理在FileSnap类中，与事务日志文件一样快照文件也一样有SNAP_MAGIC、VERSION、dbId这些，这作用也只是用来标识这是一个快照文件。

Zookeeper的数据在内存中是以DataTree为数据结构存储的，而快照就是每间隔一段时间Zookeeper就会把整个DataTree的数据序列化然后把它存储在磁盘中，这就是Zookeeper的快照文件，快照文件是指定时间间隔对数据的备份，所以快照文件中数据通常都不是最新的，多久抓一个快照这也是可以配置的，snapCount配置项用于配置处理几个事务请求后生成一个快照文件。

与事务日志文件一样快照文件也是使用zxid作为快照文件的后缀，在FileTxnSnapLog类中的save方法中生成文件并调用FileSnap类序列化DataTree数据并且写入快照文件中。

#### 快照文件可视化
与日志文件一样Zookeeper也为快照文件提供了可视化的工具org.apache.zookeeper.server包中的SnapshotFormatter类，接下来就使用该工具输出该事务日志文件，并解释该数据

```
java -cp ../../zookeeper-3.4.6.jar;../../lib/slf4j-api-1.6.1.jar org.apache.zookeeper.server.SnapshotFormatter ./data/version-2/snapshot.17
```

#### 快照分析
快照文件就很容易看得懂了，这就是Zookeeper整个节点数据的输出。

一般如下：

```
ZNode Details (count=11):

/cZxid = 0x00000000000000
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x00000000000000
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x00000000000016
cversion = 7
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x00000000000000
dataLength = 0

...

Session Details (sid, timeout, ephemeralCount): 
0x14f211584840000, 4000, 0 　　
0x14f211399480001, 4000, 0
```

* 第一行：ZNode节点数总共有11个
* 第二行：节点详情列表，显示每个节点下列情况

```
cZxid：创建节点时的ZXID
ctime：创建节点的时间
mZxid：节点最新一次更新发生时的zxid
mtime：最近一次节点更新的时间
pZxid：父节点的zxid
cversion：子节点更新次数
dataVersion：节点数据更新次数
aclVersion：节点acl更新次数
ephemeralOwner：如果节点为ephemeral节点则该值为sessionid，否则为0
dataLength：该节点数据的长度
```
* 最后一行：

这里是说当前抓取快照文件的时间Zookeeper中Session的详情，有两个session超时时间都是4000毫秒ephemeral节点为0。