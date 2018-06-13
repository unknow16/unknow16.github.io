---
title: Distributed-05-2PC and 3PC
date: 2018-06-13 15:02:53
tags: Distributed
---

## 前言
在分布式系统中，为了保证数据的高可用，通常，我们会将数据保留多个副本(replica)，这些副本会放置在不同的物理的机器上。为了对用户提供正确的增\删\改\差等语义，我们需要保证这些放置在不同物理机器上的副本是一致的。

为了解决这种分布式一致性问题，前人在性能和数据一致性的反反复复权衡过程中总结了许多典型的协议和算法。其中比较著名的有二阶提交协议（Two Phase Commitment Protocol）、三阶提交协议（Three Phase Commitment Protocol）和Paxos算法。

## 何为一致性问题？
简单而言，一致性问题就是相互独立的节点之间如何达成一项决议的问题。分布式系统中，进行数据库事务提交(commit transaction)、Leader选举、序列号生成等都会遇到一致性问题。

这个问题在我们的日常生活中也很常见，比如牌友怎么商定几点在哪打几圈麻将：
```
我: 老王，今晚7点老地方，搓够48圈不见不散！
……
（第二天凌晨3点） 隔壁老王: 没问题！       // 消息延迟
我: ……
----------------------------------------------
我: 小张，今晚7点老地方，搓够48圈不见不散！
小张: No ……                           
（两小时后……）
小张: No problem！                     // 宕机节点恢复
我: ……
-----------------------------------------------
我: 老李头，今晚7点老地方，搓够48圈不见不散！
老李: 必须的，大保健走起！               // 拜占庭将军
（这是要打麻将呢？还是要大保健？还是一边打麻将一边大保健……）
```
#### 满足一致性条件
假设一个具有N个节点的分布式系统，当其满足以下条件时，我们说这个系统满足一致性：

1. 全认同(agreement): 所有N个节点都认同一个结果
1. 值合法(validity): 该结果必须由N个节点中的节点提出
1. 可结束(termination): 决议过程在一定时间内结束，不会无休止地进行下去

#### 面临问题
但就这样看似简单的事情，分布式系统实现起来并不轻松，因为它面临着这些问题：

- 消息传递异步无序(asynchronous): 现实网络不是一个可靠的信道，存在消息延时、丢失，节点间消息传递做不到同步有序(synchronous)
- 节点宕机(fail-stop): 节点持续宕机，不会恢复
- 节点宕机恢复(fail-recover): 节点宕机一段时间后恢复，在分布式系统中最常见
- 网络分化(network partition): 网络链路出现问题，将N个节点隔离成多个部分
- 拜占庭将军问题(byzantine failure): 节点或宕机或逻辑失败，甚至不按套路出牌抛出干扰决议的信息

我们把以上所列的问题称为系统模型(system model)，讨论分布式系统理论和工程实践的时候，必先划定模型。例如有以下两种模型：

1. 异步环境(asynchronous)下，节点宕机(fail-stop)
1. 异步环境(asynchronous)下，节点宕机恢复(fail-recover)、网络分化(network partition)

2比1多了节点恢复、网络分化的考量，因而对这两种模型的理论研究和工程解决方案必定是不同的，在还没有明晰所要解决的问题前谈解决方案都是一本正经地耍流氓。

## 二阶段提交模型(2PC)
2PC(tow phase commit)两阶段提交，顾名思义它分成两个阶段，先由一方进行提议(propose)并收集其他节点的反馈(vote)，再根据反馈决定提交(commit)或中止(abort)事务。我们将提议的节点称为协调者(coordinator)，其他参与决议节点称为参与者(participants, 或cohorts)。

在阶段1中，coordinator发起一个提议，分别问询各participant是否接受，如下图

![image](https://note.youdao.com/yws/api/personal/file/978FBF4E3B5D408B936D09800C1FA8B6?method=download&shareKey=dc75331d9237af740eed839dab70f88d)

在阶段2中，coordinator根据participant的反馈，提交或中止事务，如果participant全部同意则提交，只要有一个participant不同意就中止。如下图，另外参与者执行完事务提交或回滚后，会向协调者反馈，图中没有画出。

![image](https://note.youdao.com/yws/api/personal/file/38678F1DAD384845AE338D0B871D0488?method=download&shareKey=76e9b6a3616093f856194cdce9743eef)

#### 映射到数据库事务
阶段1：协调者让参与者执行一个事务包含的SQL,但不提交，同时写redo和undo日志，反馈执行结果，成功或失败。

阶段2：协调者根据所有参与者反馈的执行结果，如果所有参与者反馈执行成功，则通知所有参与者提交事务，否则通知所有参与者回滚事务。

#### 协调者单点故障
在异步环境(asynchronous)并且没有节点宕机(fail-stop)的模型下，2PC可以满足全认同、值合法、可结束，是解决一致性问题的一种协议。但如果再加上节点宕机(fail-recover)的考虑，2PC是否还能解决一致性问题呢？

coordinator如果在发起提议后宕机，那么participant将进入阻塞(block)状态、一直等待coordinator回应以完成该次决议。这时需要另一角色把系统从不可结束的状态中带出来，我们把新增的这一角色叫协调者备份(coordinator watchdog)。coordinator宕机一定时间后，watchdog接替原coordinator工作，通过问询(query) 各participant的状态，决定阶段2是提交还是中止。这也要求 coordinator/participant 记录(logging)历史状态，以备coordinator宕机后watchdog对participant查询、coordinator宕机恢复后重新找回状态。

#### 同步阻塞问题
执行过程中，所有参与节点都是事务阻塞型的。
当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。

#### 数据不一致
在二阶段提交的阶段二中，当协调者向参与者发送commit请求之后，发生了局部网络异常或者在发送commit请求过程中协调者发生了故障，这回导致只有一部分参与者接受到了commit请求。

而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。

于是整个分布式系统便出现了数据部一致性的现象。

## 三阶段提交模型(3PC)
相比2PC，3PC增加了一个准备提交(prepare to commit)阶段来解决以上问题，并在协调者和参与者中都引入超时机制，主要分为：询问，然后再锁资源，最后真正提交。

![image](https://note.youdao.com/yws/api/personal/file/11DCBE566D4C4AAA82AE587971620278?method=download&shareKey=288902bda3c75926a52ea0d39e09edb3)

coordinator接收完participant的反馈(vote)之后，进入阶段2，给各个participant发送准备提交(prepare to commit)指令。participant接到准备提交指令后可以锁资源，但要求相关操作必须可回滚。coordinator接收完确认(ACK)后进入阶段3、进行commit/abort，3PC的阶段3与2PC的阶段2无异。协调者备份(coordinator watchdog)、状态记录(logging)同样应用在3PC。

participant如果在不同阶段宕机，我们来看看3PC如何应对：
- 阶段1: coordinator或watchdog未收到宕机participant的vote，直接中止事务；宕机的participant恢复后，读取logging发现未发出赞成vote，自行中止该次事务
- 阶段2: coordinator未收到宕机participant的precommit ACK，但因为之前已经收到了宕机participant的赞成反馈(不然也不会进入到阶段2)，coordinator进行commit；watchdog可以通过问询其他participant获得这些信息，过程同理；宕机的participant恢复后发现收到precommit或已经发出赞成vote，则自行commit该次事务
- 阶段3: 即便coordinator或watchdog未收到宕机participant的commit ACK，也结束该次事务；宕机的participant恢复后发现收到commit或者precommit，也将自行commit该次事务

#### 3PC默认提交缺点
如果进入PreCommit后，Coordinator发出的是abort请求，假设只有一个Cohort收到并进行了abort操作，
而其他对于系统状态未知的Cohort会根据3PC选择继续Commit，此时系统状态发生不一致性。